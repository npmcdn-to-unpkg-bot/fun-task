### Project status: code isn't ready and outdated, don't look at it :-)


# Task

An abstraction for managing asynchronous code in JS.

Task is an abstraction similar to Promises. The key difference from Promises is that a Task represents a computation while a Promise represents only the result of a computation. Therefore if we have a Task we can: start the computation, terminate it before it finished, or wait until it finishes and get the result. While with a Promise we can only get the result. This difference don't make Tasks **better**, they are just different from Promises and we can find legitimate use cases for both abstractions. Let's review it again:

If we have a Task:

 - We can start the computation that it represents (e.g. a network request)
 - Although we can choose not to start the computation and just throw task away
 - Also we can start it more than once
 - While computation is running we can notify it that we don't interested in the result any more, and as a response computation can choose to terminate itself
 - When computation finishes we get the result

If we have a Promise:

 - Computation is already running (or finished) and we don't have any control of it
 - We can get the result whenever it's ready
 - If two or more consumers have a same Promise they all will get the same result

The last item is important. This is the key advantage of Promises over Tasks. Tasks don't have this feature. If two consumers have a same Task, each of them have to spawn their own versions of the computation in order to get the result, and they may even get different results.


## What is a computation?

```js
function computation(onSuccess, onFailure) {
  // ...
  return () => {
    // ... cancelation logic
  }
}
```

Computation from Task API perspective is just a function that accepts two callbacks.
It should call one of those callbacks after completion with the final result.
Also a computation may return a function with cancelation logic or it can return `undefined`
if particular computation has no cancelation logic.

Creating a Task from a computation is easy, we just call `task = Task.create(computation)`.
This is very similar to `new Promise(computation)`, but Task won't call `computation`
immediately, the computation starts only when `task.run()` is called
(unlike with Promises where computation is started immediately).


## API comparison with Promises

| Task | Promise | Comments |
| ---- | ------- | -------- |
| `Task.create(computation)` | `new Promise(computation)` |  |
| `Task.of(x)` | `Promise.resolve(x)` | With Promises behaviour is different if `x` is a Promise (this makes writing generic code more difficult with Promises) |
| `task.map(fn)` | `promise.then(fn)` | With Promises behaviour is different if `fn` retruns a Promise |
| `task.chain(fn)` | `promise.then(fn)` |  |
| `task.mapRejected(fn)` | `promise.then(undefined, fn)` | With Promises behaviour is different if `fn` retruns a Promise |
| `task.orElse(fn)` | `promise.then(undefined, fn)` |  |
| `task.ap(otherTask)` | `Promise.all(promise, otherPromise).then(([fn, x]) => fn(x))` | This method exists mainly for compliance with [Fantasy Land Specification](https://github.com/fantasyland/fantasy-land) |
| `Task.empty()` | new Promise(() => {}) |  |
| `task.concat(otherTask)` | Promise.race([promose, otherPromise]) | Aslo mainly for Fantasy Land, makes Task a [Monoid](https://github.com/fantasyland/fantasy-land#monoid) |
| `Task.all(tasks)` | `Promise.all(promises)` |  |
| `Task.race(tasks)` | `Promise.race(promises)` |  |
| `Task.run(onSuccess, onFailure)` | `Promise.then(onSuccess, onFailure)` |  |
| `Task.runAndCatch(onSuccess, onFailure, onException)` | `Promise.then(onSuccess, onFailure)` | By default tasks don't catch exceptions thrown from `map`, `chain` etc. But we can choose to catch them by using `runAndCatch` instead of `run`. Also notice that exceptions go into their own callback. |



## Development

```
npm run lobot -- --help
```

Run [lobot](https://github.com/rpominov/lobot) commands as `npm run lobot -- args...`