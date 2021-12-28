---
title: React 状态更新
date: 2021-12-23 22:35:09
tags:
---

## React 的架构大概分为：

- Scheduler（调度器）—— 调度任务的优先级，高优任务优先进入Reconciler
- Reconciler（协调器）—— 负责找出变化的组件
- Renderer（渲染器）—— 负责将变化的组件渲染到页面上

Reconciler 工作的阶段被称为 render 阶段。因为在该阶段会调用组件的 render 方法。
Renderer 工作的阶段被称为 commit 阶段。就像你完成一个需求的编码后执行 git commit 提交代码。commit 阶段会把 render 阶段提交的信息渲染在页面上。
render 与 commit 阶段统称为 work，即 React 在工作中。相对应的，如果任务正在 Scheduler 内调度，就不属于 work。

## Render 阶段
Render 阶段开始于 performSyncWorkOnRoot 或 performConcurrentWorkOnRoot 方法的调用。这取决于本次更新是同步更新还是异步更新。

```js
// performSyncWorkOnRoot会调用该方法
function workLoopSync() {
  while (workInProgress !== null) {
    performUnitOfWork(workInProgress);
  }
}

// performConcurrentWorkOnRoot会调用该方法
function workLoopConcurrent() {
  while (workInProgress !== null && !shouldYield()) {
    performUnitOfWork(workInProgress);
  }
}
```

可以看到，他们唯一的区别是是否调用 shouldYield。如果当前浏览器帧没有剩余时间，shouldYield 会中止循环，直到浏览器有空闲时间后再继续遍历。

workInProgress 代表当前已创建的 workInProgress fiber。

performUnitOfWork 方法会创建下一个 Fiber 节点并赋值给 workInProgress，并将 workInProgress 与已创建的 Fiber 节点连接起来构成Fiber 树。

我们知道 Fiber Reconciler 是从 Stack Reconciler 重构而来，通过遍历的方式实现可中断的递归，所以 performUnitOfWork 的工作可以分为两部分：“递”和“归”

### “递”阶段：

首先从 rootFiber 开始向下深度优先遍历。为遍历到的每个 Fiber 节点调用 beginWork 方法。
该方法会根据传入的 Fiber 节点创建子 Fiber 节点，并将这两个 Fiber 节点连接起来。
当遍历到叶子节点（即没有子组件的组件）时就会进入“归”阶段。

### “归”阶段：

在“归”阶段会调用 completeWork 处理 Fiber 节点。
当某个 Fiber 节点执行完 completeWork，
如果其存在兄弟 Fiber 节点（即 fiber.sibling !== null），会进入其兄弟 Fiber 的“递”阶段。
如果不存在兄弟 Fiber，会进入父级 Fiber 的“归”阶段。

“递”和“归”阶段会交错执行直到“归”到 rootFiber。至此，render 阶段的工作就结束了。

render 阶段全部工作完成。在 performSyncWorkOnRoot 函数中 fiberRootNode 被传递给 commitRoot 方法，开启 commit 阶段工作流程。

## commit 阶段

commitRoot 方法是 commit 阶段工作的起点。fiberRootNode 会作为传参。

```js
commitRoot(root);
```

在 rootFiber.firstEffect 上保存了一条需要执行副作用的 Fiber 节点的单向链表 effectList，这些 Fiber 节点的 updateQueue 中保存了变化的 props。

这些副作用对应的 DOM 操作在 commit 阶段执行。

除此之外，一些生命周期钩子（比如 componentDidXXX）、hook（比如 useEffect）需要在 commit 阶段执行。

commit 阶段的主要工作（即 Renderer 的工作流程）分为三部分：
- before mutation 阶段（执行 DOM 操作前）
- mutation 阶段（执行 DOM 操作）
- layout 阶段（执行 DOM 操作后）

在 before mutation 阶段之前和 layout 阶段之后还有一些额外工作，涉及到比如 useEffect 的触发、优先级相关的重置、ref 的绑定/解绑。

### before mutation 之前

commitRootImpl 方法中直到第一句 `if (firstEffect !== null)` 之前属于 before mutation 之前。

before mutation 之前主要做一些变量赋值，状态重置的工作。

### layout 之后

主要包括三点内容：
- useEffect相关的处理。
- 性能追踪相关。
  源码里有很多和 interaction 相关的变量。他们都和追踪 React 渲染时间、性能相关，在 Profiler API 和 DevTools 中使用。
- 在 commit 阶段会触发一些生命周期钩子（如 componentDidXXX）和 hook（如 useLayoutEffect、useEffect）。

在这些回调方法中可能触发新的更新，新的更新会开启新的 render-commit 流程。

### before mutation

