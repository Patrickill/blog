---
title: 从防抖和节流函数中理解this和闭包
date: 2024-04-19 14:17:12
tags: ['front']
summary: 今天学到了防抖和节流函数的写法，总结一下我的思考
---

## 先来看代码

```javascript
// 防抖函数
function debounce(fn, wait) {
  let timer;
  return function () {
    let _this = this;
    let args = arguments;
    if (timer) {
      clearTimeout(timer);
    }
    timer = setTimeout(function () {
      fn.apply(_this, args);
    }, wait);
  };
}
// 使用
window.onresize = debounce(function () {
  console.log("resize");
}, 500);
//用法是用debounce传递给事件一个函数的引用，使得每次触发事件都执行一次业务函数
//debounce参数分别为（业务函数，防抖时长） 需要注意的是函数不会直接执行，即使是触发了一次也是会在wait的时间过后执行的

// 节流函数
function thorttle2(fn, wait) {
  let timer;
  return function () {
    let _this = this;
    let args = arguments;
 
    if (!timer) {
      timer = setTimeout(function () {
        timer = null;
        fn.apply(_this, args);
      }, wait);
    }
  };
}
//节流和防抖的差别主要在于对timer的处理 节流不会重置timer而是直接return
```

如果你是第一次读防抖的代码实现你可能会有以下疑惑：

- 为什么要返回一个匿名函数
- 为什么要let _this = this 然后再apply显示绑定this

别急，下面我会分别介绍为什么要这么写以及这么写的目的是什么：）

## 为什么要返回一个匿名函数

为了了解这个问题我们需要先明确闭包的知识

### 什么是闭包？

闭包是JavaScript中一个重要的概念。简单地说，闭包是一个函数和其周围的状态（即词法环境）的组合。在JavaScript中，如果一个函数使用了其外部函数的变量，即使外部函数已经执行完毕，这些变量仍然可以被内部函数访问。这就是闭包的核心特性，也就是**内部函数对外部函数变量的不释放**。**无论何种手段将内部函数传递到所在词法作用域之外，他都会持有对原始定义作用于的引用，无论在何处执行到这个函数都会使用闭包**

### 为什么需要返回一个匿名函数？

在 `debounce` 函数的案例中，我们定义了一个函数，这个函数接收另一个函数 `fn` 和一个延迟时间 `wait` 作为参数，并返回一个新的匿名函数。这个返回的函数是实际上被事件处理器调用的函数。让我们分析为什么这样做：

1. **持久状态**：`debounce` 函数的目的是控制另一个函数 `fn` 的执行频率，使其不会因为事件频繁触发而过于频繁地执行。为了实现这一点，我们需要跟踪上一次尝试执行 `fn` 的时间。这通过使用 `timer` 变量来实现，它保存了一个计时器ID。这个 `timer` 必须在事件多次触发时持续存在，因此需要一个闭包来记住这个状态。
2. **控制执行逻辑**：通过返回一个匿名函数，我们可以确保每次事件触发时，这个函数都被调用，而且每次调用都能访问到相同的 `timer` 变量。这是因为这个匿名函数通过闭包形成了对 `debounce` 函数内部变量（如 `timer`）的引用。如果没有闭包，变量 `timer` 将在 `debounce` 函数执行完后丢失，我们无法实现防抖功能。

### 事件触发时发生了什么？

当我们执行以下代码时

```javascript
window.onresize = debounce(function () {
  console.log("resize");
}, 500)
```

我们做的实际上是把debounc中return的匿名函数的引用传给了onresize这一事件使得每次触发事件时都执行那个匿名函数，也就是在这里产生了闭包。

以下是被调用时发生的步骤：

1. **检查和清理**：函数首先检查是否有已经设置的计时器（通过 `timer` 变量）。如果有，它使用 `clearTimeout` 清除该计时器，这阻止了之前准备执行的 `fn` 函数的执行。
2. **重新设置计时器**：然后，函数设置一个新的计时器，延迟 `wait` 毫秒执行 `fn` 函数。这确保了只有在最后一次事件触发后的 `wait` 毫秒内没有新的事件触发，`fn` 函数才会执行。
3. **使用闭包的优势**：由于匿名函数是在 `debounce` 函数的词法作用域内定义的，它通过闭包记住了 `timer` 变量。这意味着不论事件触发多少次，所有的这些触发都是通过操作同一个 `timer` 变量来决定是否执行 `fn`。

## 为什么要let _this = this 然后再apply显示绑定this

### 为什么要获取 `let _this = this`？

在JavaScript中，`this` 关键字的值指向调用函数的上下文。在全局函数中调用时，`this` 通常指向全局对象（在浏览器中是 `window`），而在对象的方法中调用时，`this` 指向该方法所属的对象。然而，当你使用像 `setTimeout` 这样的函数时，由于 `setTimeout` 是由全局对象调用的，其回调函数中的 `this` 通常会指向全局对象，而不是你期望的上下文（比如某个特定对象或类的实例）。

### 为什么要在 `setTimeout` 中显式绑定 `this`？

由于 `setTimeout` 回调函数中的 `this` 指向全局对象（或者在严格模式下是 `undefined`），如果你希望回调函数中的 `this` 指向正确的上下文，你需要手动保存和传递这个上下文。这就是为什么我们常见 `let _this = this;` 这行代码，它将外部函数的 `this` 值保存在 `_this` 变量中，然后在 `setTimeout` 的回调函数中通过闭包访问 `_this` 来确保正确的上下文。

### 显式绑定的替代方法

除了使用 `let _this = this;`，还有其他几种方法可以确保 `this` 指向正确：

1. **使用箭头函数**：箭头函数不会创建自己的 `this` 上下文，而是继承它们被定义时的 `this` 值（我认为这是词法作用域的向上寻找的特性导致的结果）。这意味着你可以在 `setTimeout` 中直接使用箭头函数，而不必担心 `this` 会被错误地绑定。

   ```
   javascriptCopy codesetTimeout(() => {
       this.doSomething(); // 这里的 this 继承自外部作用域
   }, 1000);
   ```

2. **使用 `Function.prototype.bind`**：`bind` 方法创建一个新的函数，这个函数在被调用时将其 `this` 关键字设置为提供的值。

   ```
   javascriptCopy code
   setTimeout(this.doSomething.bind(this), 1000);
   ```

### 如果不使用 `setTimeout` 还需不需要绑定 `this`？

是否需要显式绑定 `this` 取决于函数被如何调用。在事件处理器、回调函数、或者任何可能导致 `this` 上下文变化的场景下，如果你需要保证 `this` 指向特定的上下文，那么显式绑定 `this` 是必要的。在同步代码中也可能需要绑定，特别是当你将函数作为参数传递给期望以不同上下文调用这些函数的代码时。

## 具体业务场景

- 防抖

1. 登录、发短信等按钮避免用户点击太快，以致于发送了多次请求，需要防抖
2. 调整浏览器窗口大小时，resize 次数过于频繁，造成计算过多，此时需要一次到位，就用到了防抖
3. 文本编辑器实时保存，当无任何更改操作一秒后进行保存



以文本编辑器保存为例子：

```javascript
function saveContent() {
  console.log("Saving content:", document.getElementById("editor").value);
  // 这里可以添加将数据保存到服务器的API调用
}

// 应用防抖
const debouncedSave = debounce(saveContent, 1000);

// 添加事件监听
window.onload = function() {
  document.getElementById("editor").addEventListener('input', debouncedSave);
}
```



- 节流

1. `scroll` 事件，每隔一秒计算一次位置信息等
2. 浏览器播放事件，每个一秒计算一次进度信息等
3. input 框实时搜索并发送请求展示下拉列表，每隔一秒发送一次请求 (也可做防抖)

下面是一个用Date实现的节流函数

```javascript
function throttle(func, limit) {
  let lastFunc;
  let lastRan;
  return function() {
    const context = this;
    const args = arguments;
    if (!lastRan) {
      func.apply(context, args);
      lastRan = Date.now();
    } else {
      clearTimeout(lastFunc);
      lastFunc = setTimeout(function() {
        if ((Date.now() - lastRan) >= limit) {
          func.apply(context, args);
          lastRan = Date.now();
        }
      }, limit - (Date.now() - lastRan));
    }
  }
}
document.addEventListener('DOMContentLoaded', function() {
  const video = document.getElementById('videoPlayer');
  video.addEventListener('timeupdate', throttle(function() {
    console.log(`Current Time: ${video.currentTime} seconds`);
  }, 1000));
});
```

