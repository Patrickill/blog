---
title: JavaScript基础#1——事件循环与Promise
date: 2024-4-17 18:54:35
tags: ['front']
summary: 一篇文章带你了解JavaScript的事件机制与Promise
---

# 事件循环

## 基本介绍

JavaScript 是 **`单线程`** 语言，意味着只有单独的一个调用栈，同一时间只能处理一个任务或一段代码。队列、堆、栈、事件循环构成了 js 的并发模型，**`事件循环`** 是 JavaScript 的执行机制。

为什么js是一门单线程语言呢？最初设计JS是用来在浏览器验证表单以及操控DOM元素，为了避免同一时间对同一个DOM元素进行操作从而导致不可预知的问题，JavaScript从一诞生就是单线程。

既然是单线程也就意味着不存在异步，只能自上而下执行，如果代码阻塞只能一直等下去，这样导致很差的用户体验，所以事件循环的出现让 js 拥有异步的能力。
  
## 整体流程

在JavaScript中，可执行代码的执行环境主要涉及三个关键部分：执行栈、宏任务队列和微任务队列。这些部分共同构成了JavaScript的事件循环机制，确保了代码的异步执行和非阻塞性质，同时也保持了执行的有序性。

首先，执行栈，也被称为调用栈，是用于存储函数调用的数据结构。当执行一段JavaScript代码时，引擎首先处理全局代码，创建全局执行上下文并将其压入执行栈。当遇到函数调用时，函数的执行上下文被创建并压入执行栈的顶部，函数执行结束后，其上下文被弹出。

在全局代码和函数执行过程中，如果遇到异步操作，如`setTimeout`或`Promise`，则其回调函数或后续操作会被分别放入宏任务队列或微任务队列。宏任务队列包括来自`setTimeout`、`setInterval`、I/O等的任务，而微任务队列主要来源于`Promise`回调和`MutationObserver`回调。

执行栈清空后，即所有同步任务执行完毕，JavaScript引擎会首先检查微任务队列。如果微任务队列非空，引擎会连续执行所有微任务，直到该队列清空。这一过程确保了诸如`Promise`链这样的微任务能够快速且连续地被处理，而不受宏任务的干扰。

只有当微任务队列为空时，引擎才会从宏任务队列中取出一个任务执行。执行完一个宏任务后，引擎再次回到微任务队列检查，如果有新的微任务被添加，那么这些微任务会在下一个宏任务执行之前全部完成。这样，事件循环保证了即使在宏任务和微任务混合的情况下，所有的任务都能按照预定的顺序执行，同时也确保了更紧急的微任务能够优先得到处理。

通过这种机制，JavaScript能够在单线程环境中有效地执行异步代码，同时保持对用户界面的响应性和处理复杂逻辑的能力

```
let a = 1;

  

setTimeout(() => {

    console.log(1);

}, 100);

  

console.log(2);

  

setTimeout(() => {

    console.log(3);

}, 0);

  

Promise.resolve().then(() => {

    console.log(4);

});

  

console.log(5);
```

```
const promise = new Promise((resolve, reject) => {
  console.log("Promise 执行函数");
  resolve();
}).then((result) => {
  console.log("Promise 回调（.then）");
});

setTimeout(() => {
  console.log("新一轮事件循环：Promise（已完成）", promise);
}, 0);

console.log("Promise（队列中）", promise);

```

## 为什么promise要对setTimeout进行封装

# Promise

## 是什么

[`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) 是一个对象，它代表了一个异步操作的最终完成或者失败。以及操作的成功或者失败之后执行什么回调函数

## promise简介

1. Promise的概念

    是一个对象，它代表了一个异步操作的最终完成或者失败。以及操作的成功或者失败之后执行什么回调函数

    从语法上说,Promise是一个构造函数,用来生成Promise实例。Promise实例代表一个异步操作,有三种状态:pending(进行中)、fulfilled(已成功)和rejected(已失败)。

2. Promise的三种状态

    - pending(进行中):这是Promise的初始状态。在Promise被创建时,它处于pending状态,表示异步操作尚未完成。

    - fulfilled(已成功):当异步操作成功完成时,Promise从pending状态转换为fulfilled状态。此时,Promise的then方法绑定的回调函数将被调用。

    - rejected(已失败):当异步操作遇到错误或无法完成时,Promise从pending状态转换为rejected状态。此时,Promise的catch方法绑定的回调函数将被调用。

    需要注意的是,Promise的状态一旦改变,就会凝固,不会再发生变化。这意味着一个Promise要么成功,要么失败,不存在中间状态。

3. Promise解决的问题

    在传统的异步编程中,我们通常使用回调函数来处理异步操作的结果。然而,当异步操作变得复杂,出现多层嵌套的回调函数时,代码会变得难以理解和维护,这就是所谓的"回调地狱"。

```
doSomething(function (result) {
  doSomethingElse(result, function (newResult) {
    doThirdThing(newResult, function (finalResult) {
      console.log(`得到最终结果：${finalResult}`);
    }, failureCallback);
  }, failureCallback);
}, failureCallback);
```

```
doSomething()
  .then(function (result) {
    return doSomethingElse(result);
  })
  .then(function (newResult) {
    return doThirdThing(newResult);
  })
  .then(function (finalResult) {
    console.log(`得到最终结果：${finalResult}`);
  })
  .catch(failureCallback);

```

## 如何使用

### 使用构造函数创建Promise

Promise构造函数接受一个函数作为参数,这个函数被称为执行器函数(executor function)。执行器函数接受两个参数:resolve和reject,它们都是函数。

```javascript
const promise = new Promise((resolve, reject) => {
  // 异步操作
  if (/* 异步操作成功 */) {
    resolve(value); // 调用resolve,将Promise状态变为fulfilled
  } else {
    reject(error); // 调用reject,将Promise状态变为rejected
  }
});
```

*注意这里resolve是在Promise内部实现的不由我们定义*

```javascript
function Promise(executor) {
  // JavaScript引擎内部创建resolve和reject函数
  function resolve(value) {
    // 将Promise状态变为fulfilled
    // ...
  }
  
  function reject(reason) {
    // 将Promise状态变为rejected
    // ...
  }
  
  // 调用执行器函数,并将resolve和reject函数作为参数传递
  executor(resolve, reject);
}
```

### 基础使用

```javascript
promise
  .then(value => {
    return new Promise((resolve, reject) => {
      // 嵌套的异步操作
      if (/* 异步操作成功 */) {
        resolve(newValue);
      } else {
        reject(error);
      }
    });
  })
  .then(newValue => {
    // 处理嵌套异步操作的结果
  })
  .catch(error => {
    // 处理嵌套异步操作的错误
  });
  .finally(() => {
 //最后执行的操作
  })
```

#### then做了什么工作

##### `.then()` 方法的工作流程

1. **处理结果**：当 Promise 成功解决（resolved）时，会调用 `.then()` 中的第一个参数（成功的回调函数），并将解决的结果作为参数传递给这个回调函数。
2. **错误处理**：如果 Promise 被拒绝（rejected），并且 `.then()` 提供了第二个参数（失败的回调函数），则会调用此回调函数，将拒绝的原因作为参数传递给它。
3. **链式调用**：`.then()` 方法会返回一个新的 Promise（可以在这里使用Promise.resolve或者reject显示调用方法，但是可以在函数中返回值会自动地resolve那个返回的值，传递给下一个函数）。根据你在 `.then()` 回调函数中返回的值，这个新的 Promise 可能会被解决（resolve）或拒绝（reject）。这使得 Promise 可以形成链式调用。

#### 在最后统一处理error情况

在实践中，使用`.catch()`方法来捕获并处理错误更为常见。这种模式有几个好处：
 1. **更清晰的错误处理逻辑**
使用`.catch()`可以让成功的处理逻辑和错误处理逻辑分开，使得代码更易读和维护。在链式调用中，你可以在链的末尾使用一个`.catch()`，来捕获链中任何一步发生的错误。
2. **避免遗漏错误**
如果你在每个`.then()`后面都使用第二个参数来处理错误，很容易遗漏某个环节的错误处理。使用`.catch()`可以确保所有未被捕获的错误都能在链的末尾被处理。
3. **统一错误处理**

在复杂的应用中，可能需要根据不同类型的错误做不同的处理。使用`.catch()`允许你在一个地方集中处理错误，甚至根据错误的类型做出不同的响应，这比在每个`.then()`方法中分别处理要简洁得多。

## 一些原理

每个 `.then()` 或 `.catch()` 返回一个新的 Promise，这个新的 Promise 的结果值会成为下一个 `.then()` 的参数，而如果这个新的 Promise 被拒绝，则会跳过后续的 `.then()` 方法，直到遇到一个 `.catch()` 方法

```javascript
function validateCredentials(username, password) {
    return new Promise((resolve, reject) => {
        // 模拟异步验证
        setTimeout(() => {
            if (username === 'johndoe' && password === 'secret') {
                resolve({ userId: 123 });
            } else {
                reject(new Error('Invalid credentials'));
            }
        }, 1000);
    });
}

function getUserInfo(userId) {
    return new Promise((resolve, reject) => {
        // 模拟异步获取用户信息
        setTimeout(() => {
            if (userId === 123) {
                resolve({ userId: 123, name: 'John Doe', email: 'johndoe@example.com' });
            } else {
                reject(new Error('User not found'));
            }
        }, 1000);
    });
}

function getUserRoles(user) {
    return new Promise((resolve, reject) => {
        // 模拟异步获取用户角色
        setTimeout(() => {
            if (user.userId === 123) {
                resolve({ ...user, roles: ['admin', 'user'] });
            } else {
                reject(new Error('Failed to fetch user roles'));
            }
        }, 1000);
    });
}

// 使用Promise链
validateCredentials('johndoe', 'secret')
    .then(result => {
        console.log('Authentication successful:', result);
        return getUserInfo(result.userId);
    })
    .then(user => {
        console.log('User info retrieved:', user);
        return getUserRoles(user);
    })
    .then(userWithRoles => {
        console.log('User roles retrieved:', userWithRoles);
    })
    .catch(error => {
        console.error('Error:', error);
    });
```

当你在 `.then()` 中返回一个值时,这个值会被自动封装成一个已解决的 Promise,并被传递给下一个 `.then()`。但如果你返回一个 Promise,那么这个 Promise 的状态将决定下一步会发生什么:

- 如果返回的 Promise 被解决(resolved),它的结果值会被传递给下一个 `.then()`。
- 如果返回的 Promise 被拒绝(rejected),链式调用会跳到下一个 `.catch()` 方法。

这种行为允许你在 Promise 链中处理异步操作,每个 `.then()` 可以返回一个 Promise,表示需要完成的异步操作,然后下一个 `.then()` 会等待这个 Promise 解决后再执行
