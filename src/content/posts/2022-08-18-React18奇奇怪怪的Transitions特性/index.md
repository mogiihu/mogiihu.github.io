---
title: React 18 奇奇怪怪的 Transitions 特性
published: 2022-08-18
description: ''
tags: [React]
category: 'React'
draft: false 
lang: ''
---
React18 新增了 **并发（Transitions）** 特性，`startTransition` 、 `useTransition` 、`useDeferredValue` 是并发模式中的关键部分，旨在提升用户体验，通过区分**紧急更新** 和**非紧急更新** 任务来优化应用的响应性。

乍一眼看上去让人匪夷所思，仔细研究起来发现很有意思，下面是关于相关特性说明及 api 解读，以及我对其在项目中应用场景的思考。

# 紧急更新和非紧急更新

例如，我们在长列表搜索时，会先在输入框打字，这是我们期望输入框能立即更新，这就是**紧急更新**。

如果没有优化，每次输入都会触发列表过滤和重新渲染，导致输入感觉卡顿。所以可以在输入结束后一定时间内呈现，这时及减少了页面更新渲染，又可以给用户较好的体验。这里列表呈现属于**非紧急更新**。

**使用场景：**

- 紧急更新：点击按钮、搜索框打字等需要立即响应的交互
- 非紧急更新：UI 视图过渡，可以接受一定延迟

# startTransition

这是一个函数，用于将状态更新标记为非紧急。

```javascript
import { startTransition } from 'react';

// 紧急更新
setInputValue(e.target.value);

// 非紧急更新
startTransition(() => {
  setSearchQuery(input);
});
```



# useTransition

`useTransition` **Hook** 提供 Transition 状态跟踪，返回一个包含 `isPending` 状态和 `startTransition` 函数的数组。

`startTransition`  和上面介绍的作用相同，`isPending`  表示是否有过渡任务正在进行中。

```javascript
function TransitionTest() {
  const [isPending, startTransition] = useTransition();
  const [count, setCount] = useState(0);

  function handleClick() {
    startTransition(() => {
      setCount((c) => c + 1);
    });
  }

  return (
    <div>
      {isPending && <div>加载中...</div>}
      <button onClick={handleClick}>{count}</button>
    </div>
  );
}
```



# useDeferredValue

`useDeferredValue` 用于延迟更新某些不太重要的部分，返回一个延迟版本的值。

```javascript
function Typeahead() {
  const query = useSearchQuery('');
  const deferredQuery = useDeferredValue(query);

  const suggestions = useMemo(() =>
    <SearchSuggestions query={deferredQuery} />,
    [deferredQuery]
  );

  return (
    <>
      <SearchInput query={query} />
      <Suspense fallback="加载结果中...">
        {suggestions}
      </Suspense>
    </>
  );
}
```

是不是感觉和  `useTransition`  有点类似，`useDeferredValue` 和 `useTransition` 都是 React 18 中用于实现并发渲染、优化用户体验的 Hook，但它们从不同的角度和层面解决问题。



# 核心区别总结

是不是感觉这三个有点类似，它们都可以用于实现并发渲染、优化用户体验，但它们从不同的角度和层面解决问题。

- **`startTransition`**：这是一个函数，用于将状态更新标记为非紧急。它可以通过 `useTransition` Hook 中解构出来，也可以直接从 `react` 包中导入。
- **`useTransition`**：它是一个 **`hook`** ，用于包装**更新状态的代码**（如 `setSearchQuery`），让我们能控制某个状态更新的优先级。
  - 需要加载状态 `isPending` 用 `useTransition`，不需要加载状态用 `startTransition`。
- **`useDeferredValue`**：用于包装**一个状态值本身**，我们使用该值时，使用的是该值的延迟版本。

它们通常可以达到相似的优化效果，但根据我们的代码结构和逻辑，选择其中一个会更方便。



## 应用区别总结

- **`useTransition`**：
  - 需要显示加载状态（如旋转器、骨架屏）
  - 可以控制状态更新函数
- **`startTransition`**：
  - 只需要标记非紧急更新
  - 不需要显示加载状态
  - 可以控制状态更新函数

- **`useDeferredValue`**：

  - 接收一个值作为 prop，**无法控制更新它的函数**

  - 只是想延迟显示某个值，而不需要显示加载状态

    

## 组合使用示例

```jsx
function ParentComponent() {
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();
  
  const handleChange = (e) => {
    const value = e.target.value;
    setQuery(value);
    startTransition(() => {
      setQuery(value);
    });
  };

  return <ChildComponent query={query} isPending={isPending} />;
}

function ChildComponent({ query, isPending }) {
  const deferredQuery = useDeferredValue(query);
  
  return (
    <div>
      {isPending && <Spinner />}
      <Results query={deferredQuery} />
    </div>
  );
}
```



# 实际应用场景

以下是一些典型且实用的应用场景：

- **搜索框和实时过滤**：当用户在输入框中快速输入时，我们希望输入框本身能立即响应（显示用户输入的字符），但实际显示搜索结果或过滤列表的更新可以稍有延迟。
- **数据可视化和大图表渲染**：当用户与控件（如下拉框、滑块）交互来更改图表的数据或参数时，图表的重新渲染可能非常消耗性能。
- **标签页切换**：当用户在不同标签页之间切换时，期望标签页按钮的激活状态能立即响应点击，而新标签页的内容可以稍后渲染。
- **文本编辑器或富文本预览**：在一个 Markdown 编辑器中，一边是源代码输入，另一边预览区域使用延迟的输入文本，确保编辑器的输入始终流畅。



# 并发特性与防抖节流的区别

看到 React 18 并发特性的代码示例和应用场景有种似曾相识的感觉？是的，它和防抖节流在目的上相似，但实现原理和效果上有本质区别。让我们先回忆一下防抖节流，再来看看有哪些区别

### 防抖 (Debounce)

防抖确保函数在最后一次调用后经过一定延迟才执行。

```javascript
// 传统防抖实现
function debounce(func, delay) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => func.apply(this, args), delay);
  };
}

// 使用防抖
const debouncedSearch = debounce((query) => {
  setSearchResults(fetchResults(query));
}, 300);

const handleChange = (e) => {
  setInputValue(e.target.value);
  debouncedSearch(e.target.value);
};
```

### 节流 (Throttle)

节流确保函数在指定时间间隔内最多执行一次。

```javascript
// 传统节流实现
function throttle(func, delay) {
  let lastCall = 0;
  return function(...args) {
    const now = new Date().getTime();
    if (now - lastCall < delay) return;
    lastCall = now;
    return func.apply(this, args);
  };
}

// 使用节流
const throttledSearch = throttle((query) => {
  setSearchResults(fetchResults(query));
}, 300);

const handleChange = (e) => {
  setInputValue(e.target.value);
  throttledSearch(e.target.value);
};
```



## 关键区别详解

- 工作原理不同

  - **防抖/节流**：基于时间控制函数执行频率

  - **并发特性**：基于优先级控制渲染时机，允许React中断低优先级渲染


- 延迟机制不同

  - **防抖/节流**：固定时间延迟（如300ms），无论设备性能如何

  - **并发特性**：自适应延迟，在性能好的设备上延迟更短，性能差的设备上延迟更长

- 用户体验不同

  - **防抖**：用户必须停止输入一段时间才能看到结果

  - **节流**：结果定期更新，可能错过中间状态

  - **并发特性**：输入始终保持流畅，结果在后台逐渐更新

- 渲染行为不同

  - **防抖/节流**：减少渲染次数

  - **并发特性**：不减少渲染次数，只是调整渲染优先级


在我们想要减少 API 调用次数，节省服务器资源时，可以使用防抖/节流，因为React并发特性只对渲染进行优化，不决定直接的API

调用。



## 组合使用示例

```javascript
function SearchComponent() {
  const [inputValue, setInputValue] = useState('');
  const [searchQuery, setSearchQuery] = useState('');
  const [isPending, startTransition] = useTransition();
  
  // 对API调用使用防抖，对渲染使用并发特性
  const debouncedAPICall = debounce((query) => {
    startTransition(() => {
      setSearchQuery(query); // 非紧急更新
    });
  }, 150);
  
  const handleChange = (e) => {
    const value = e.target.value;
    setInputValue(value); // 紧急更新
    debouncedAPICall(value); // 防抖+非紧急更新
  };
  
  return (
    <div>
      <input value={inputValue} onChange={handleChange} />
      {isPending && <Spinner />}
      <Results query={searchQuery} />
    </div>
  );
}
```

# 结语
React 18的并发特性为我们前端优化开启了新思路，这些特性让我们能更细致的控制渲染优先级，在保持应用响应性的同时处理复杂的渲染任务。

在实际项目中，建议大家可以先从一两个关键场景开始尝试这些特性，比如搜索框、标签切换或数据列表渲染。逐步体验它们带来的用户体验提升，相信你会感受到并发模式的魅力。

