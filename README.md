# Async flow controll in modern JavaScript

#### A collection of async flow control patterns 
---

### Wait for a list of promises
Say, you have an array of promises and you want to make sure the order of them is kept as is, regardless of their resolve order.

```
async function fireInParallelWaitForAll(promises, callback) {
  const results = await Promise.all(promises)
  results.forEach(value => callback(value))
}
```

---

### Run promises in serial
In this scenario you have a bunch of promises(async tasks) and you want them to behave like a chain, meaning: a promise is
taken care of when the previous promise is resolved

```
const chainPromiseFlow = (promisesArray, callback) => 
   promisesArray.reduce((chain, promise) => chain.then(() => promise).then(callback), Promise.resolve())
```

starting with a resolved promise, we compose each promise in the array with the previous ones to make sure they
behaive in a serial way.

This way of thinking although sometimes usefull, but in general contradicts the purpose of async programming.

---

### Run promises in parallel but dont wait for all!
This is a very common use case. you have a bunch of promises you want to run them concurrently but you dont need the result of them
all at the same time. you want the values to arrive over time, but you also want them in order.

```
async function fileInParallelIterateASAP(promises, callback) {
  for await (const value of promises) {
    callback(value)
  }
}
```

With the help of async functions and async iterator loop, we can acheive this easily

---

### Listening to events and keeping the order!
In a similar scenario you might need to map an event to an async task, for example a ajax call or such, in this case we can use a little 
utility to make sure responses to that async tasks come back in the same order that they were fired.

```
let chainRoot = Promise.resolve()

const myAsyncTask = (value, cb) => {
  const newChainRoot = chain.then(() => promiseReturningFunction(value)).then(cb)
  chainRoot = newChainRoot
}
```

So, each time the function is called, the newly created promise will be chained to the previous one, but because it returns
a new promise altogether, we need to make sure the chain root is updated, otherwise we are basically forking off the initial Promise.resolve()
which means our tasks we run in parallel with no order.

---
