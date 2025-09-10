---
title: Next.js 使用 aws-sdk + uppy 文件上传到EOS云存储
published: 2025-01-10
description: ''
tags: [Next]
category: 'Next记录'
draft: false 
lang: ''
---
# 使用到的库及文档

前端库

- [@uppy](https://uppy.io/docs)

后端库：

- [@aws-sdk/client-s3](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/client/s3/)
- [@aws-sdk/s3-request-presigner](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/Package/-aws-sdk-s3-request-presigner/)

- [zod](https://zod.dev/)



# 接口实现流程

```typescript
import { z } from "zod";
import { router, checkedSessionProcedure } from "@/utils/trpc";
import dayjs from "dayjs";
import { S3Client, PutObjectCommand } from "@aws-sdk/client-s3";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";

// COS 配置
const REGION = "xx"; // COS 存储桶地域
const ENDPOINT = "xx"; // COS 服务端点
const COS_SECRET_ID = "xx"; // 访问密钥 ID
const COS_SECRET_KEY = "xx"; // 访问密钥 Secret
const BUCKET = "xx"; // 存储桶名称

/**
 * 文件上传路由
 * 提供文件上传相关的 API 接口
 */
export const fileRouter = router({
  /**
   * 创建预签名 URL
   * 用于客户端直传文件到云存储，避免通过服务器中转
   *
   * 工作流程：
   * 1. 客户端请求预签名 URL（包含文件名、类型、大小）
   * 2. 服务端生成带签名的临时上传 URL
   * 3. 客户端使用该 URL 直接上传文件到云存储
   *
   * @param filename - 原始文件名
   * @param contentType - 文件 MIME 类型
   * @param size - 文件大小（字节）
   * @returns 预签名 URL 和 HTTP 方法
   */
  createPresignedUrl: procedure
    .input(
      // 通过 zod 控制接口输入类型
      z.object({
        filename: z.string(),
        contentType: z.string(),
        size: z.number(),
      })
    )
    .mutation(async ({ input }) => {
      // 按日期创建文件夹结构，便于管理
      const dateString = dayjs().format("YYYY-MM-DD");

      // 创建 S3 客户端实例
      const s3Client = new S3Client({
        region: REGION,
        endpoint: ENDPOINT,
        credentials: {
          accessKeyId: COS_SECRET_ID,
          secretAccessKey: COS_SECRET_KEY,
        },
      });

      // 创建上传命令
      const command = new PutObjectCommand({
        Bucket: BUCKET, // 目标存储桶
        Key: `${dateString}/${input.filename}`, // 文件在云存储中的完整路径
        ContentType: input.contentType, // 设置正确的 MIME 类型
        ContentLength: input.size, // 设置文件大小
        // 可选：添加更多元数据
        // Metadata: {
        //   'original-name': input.filename,
        //   'upload-date': dateString,
        // }
      });

      // 生成预签名 URL
      const url = await getSignedUrl(s3Client, command, {
        expiresIn: 60, // URL 有效期（秒）
      });

      // 返回预签名 URL 给客户端
      return {
        url, // 客户端用于上传的完整 URL
        method: "PUT" as const, // HTTP 方法必须为 PUT
      };
    }),
});

```

## getSignedUrl 作用

### 核心作用

`getSignedUrl`的作用是**为一个原本需要权限才能访问的私有资源（如云存储中的文件），生成一个有时效性的、带签名的临时访问链接**。

任何拥有这个链接的人，在链接有效期内，都可以**直接访问该资源**，而无需拥有云服务账户的正式身份认证凭证（如 Access Key、用户名密码等），以及**无需经过的服务器中转** 。

---

### 为什么需要它？

假如我们开发了一个社交App，用户上传的图片都私密地存放在腾讯云OSS或AWS S3的Bucket里。

- **问题1（安全性）**：我们不能直接把文件在存储桶里的永久URL（例如 `https://my-bucket.s3.amazonaws.com/user-123/avatar.jpg`）给客户端。因为这个URL指向的是私有文件，如果没有正确的身份认证信息，访问它会直接被拒绝（返回403错误）。

- **问题2（成本与性能）**：我们也不能让客户端每次都通过应用服务器来下载文件。这会占用服务器的带宽和线程（服务器需要先从云存储下载文件，再提供到客户端），导致速度慢且成本高。

**`getSignedUrl` 解决了这个问题**

*   **安全**：文件本身是私有的，普通用户无法访问其他人的文件URL。
*   **高效**：生成的签名URL是临时的（例如，有效期为5分钟、1小时等）。一旦生成，客户端可以**直接使用这个URL从云存储下载或上传文件**，getSignedUrl，减轻了服务器负担，速度也更快。
*   **灵活控制**：我们可以精确控制这个临时链接的**权限（读或写）和有效期**。

简单来说， `getSignedUrl` 就是 给客户端一个临时"通行证" ，让它能安全地直接把文件上传到云存储。 几乎所有主流的云存储服务（AWS S3，Google Cloud Storage，Azure Blob Storage，阿里云 OSS，腾讯云 COS 等）都提供了这个核心功能。

# 前端上传流程

```jsx
"use client";
import Uppy from "@uppy/core";
import AwsS3 from "@uppy/aws-s3";
import { useState } from "react";
import { Input } from "@/components/ui/input";
import { useUppyState } from "@uppy/react";
import { trpcClient } from "@/utils/api";
import { Button } from "@/components/ui/button";

export default function Home() {
  // 创建 Uppy 实例配置 
  const [uppy] = useState(() => {
    const uppy = new Uppy({
      // 调试模式：开发时可开启查看更多日志
      debug: true,
      // 限制配置（可选）
      restrictions: {
        maxFileSize: 10 * 1024 * 1024, // 10MB 限制
        allowedFileTypes: ['image/*', 'video/*', 'application/pdf'], // 允许的文件类型
      },
    });

    /**
     * 配置 AWS S3 插件
     * 用于直传文件到腾讯云 COS
     */
    uppy.use(AwsS3, {
       // 禁用分片上传，适合小文件
      shouldUseMultipart: false, 
      // 获取上传参数 - 从服务端获取预签名 URL
      getUploadParameters(file) {
        if (file.data instanceof File) {
          // 调用 tRPC 服务端获取预签名 URL
          return trpcClient.fileRouter.createPresignedUrl.mutate({
            filename: file.name || "unkonw", // 文件名（带默认值）
            contentType: file.data.type || "", // 文件 MIME 类型
            size: file.data.size || 0, // 文件大小（字节）
          });
        } else {
          // 处理无效文件类型的情况
          return {
            url: "",
          };
        }
      },
    });
    return uppy;
  });

  // 获取当前选择的文件列表
  const files = useUppyState(uppy, (state) => Object.values(state.files));
  
  // 获取总上传进度
  const progress = useUppyState(uppy, (state) => state.totalProgress);

  return (
    <div className="h-screen flex items-center justify-center">
      {/* 文件选择表单 */}
      <form className="w-full max-w-md flex flex-col" action="">
        <Input
          type="file"
          multiple // 允许多文件选择
          onChange={(e) => {
            if (e.target.files) {
              // 遍历所有选择的文件并添加到 Uppy state
              Array.from(e.target.files).forEach((file) => {
                uppy.addFile({
                  name: file.name, // 文件名
                  data: file, // 文件数据（这里添加注释）
                });
              });
            }
          }}
        />
      </form>
      
      {/* 文件预览区域 */}
      <div className="w-full max-w-md flex flex-col">
        {files?.map((file) => {
          // 创建本地预览 URL
          const url = URL.createObjectURL(file.data);
          return <img key={file.id} src={url} alt={file.name} />;
        })}
      </div>
      
      {/* 手动上传按钮 */}
      <Button
        onClick={() => {
          // 手动触发上传
          uppy.upload();
        }}
      >
        upload
      </Button>
      
      {/* 进度显示 */}
      <div>{progress}</div>
    </div>
  );
}
```





