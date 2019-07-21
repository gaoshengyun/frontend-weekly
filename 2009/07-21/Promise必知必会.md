## **Promise构造函数**

```
const promise = new Promise((resolve,reject) => {
  console.log(1)
  resolve()
  console.log(2)
})
promise.then(() => {
  console.log(3)
})
console.log(4)
```

运行结果

```
1
2
4
3
```

解释: Promise构造函数是同步执行的,promise.then中的函数是异步执行的

---

## **Promise状态机制**

```
const promise1 = new Promise((resolve,reject) => {
  setTimeout(() => {
    resolve('success')
  },1000)
})
const promise2 = promise1.then(() => {
  throw new Error('error!!!')
})

console.log('promise1',promise1)
console.log('promise2',promise2)

setTimeout(() => {
  console.log('promise1',promise1)
  console.log('promise2',promise2)
},2000)
```

运行结果

```
promise1 Promise { <pending> }
promise2 Promise { <pending> }
(node:50928) UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 1): Error: error!!!
(node:50928) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
promise1 Promise { 'success' }
promise2 Promise {
  <rejected> Error: error!!!
    at promise.then (...)
    at <anonymous> }
```

解释: promise有3种状态:pending,fulfilled或reject.状态改变只能是pending>fulfilled或者pending->reject,状态一旦改变则不能再变.上面 promise2 并不是promise1,而是返回一个新的Promise实例.

---

## **Promise状态的不可改变**

```
const promise = new Promise((resolve,reject) => {
  resolve('success')
  reject('error')
  resolve('success2')
})

promise.then(res => {
  console.log('then: ',res)
}).catch(err => {
  console.log('catch: ',err)
})
```

运行结果

```
then: success1
```

解释: 构造函数中的resolve或reject只有一次执行时有效,多次调用没有任何作用,promise的状态一旦改变则不无再变

再看两个例子

```
const promise = new Promise((resolve,reject) => {
  console.log(1)
  return Promise.reject(new Error('haha'))
})
promise.then(res => {
  console.log(2,res)
}).catch(err => {
  console.error(3.err)
})
console.log(4)
console.log(promise)
```

运行结果

```
1
4
Promise { <pending> }
(node:22493) UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 1): Error: haha
(node:22493) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.

```

```
const promise = new Promise((resolve,reject) => {
  console.log(1)
  throw new Error('haha')
})
promise.then(res => {
  console.log(2,res)
}).catch(err => {
  console.error(3,err)
})
console.log(3)
console.log(promise)
```

运行结果

```
1
4
Promise {
  <rejected> Error: haha
    at Promise (/Users/nswbmw/Desktop/test/app.js:6:9)
    ...
3 Error: haha
    at Promise (/Users/nswbmw/Desktop/test/app.js:6:9)
    ...
```

解释: 构造函数内只能通过调用resolve(pending->fulfiled)或者reject(pending->reject)或者throw一个error(pending->reject)改变状态.所以第一个例子的promise是pending,也就不会调用.then/.catch

---

## **Promise链式调用**
```
Promise.resolve(1).then(res => {
  console.log(res)
  return 2
}).catch(err => {
  return 3
}).then(res => {
  console.log(res)
})
```

运行结果

```
1
2
```

解释: promise可以链式调用.提起链式调用我们通常会想到return this实现,不过Promise并不是这样实现的.promise在每次调用.then或者.cathc时都会返回一个新的promise,从而可以实现链式调用.

Axios就是通过链式调用来实现拦截器的.原理如下

```
/** 简单解释链式调用，定义一个promiseRequest */
let requestPromise = function () {
  let chain = [{
    resolved: () => { console.log('xhr'); return 'xhr'},
    rejected: () => 'rejected'
  }]

  let requestInters = ['request1', 'request2', 'request3']
  requestInters.forEach(request => {
    chain.unshift({
      resolved: () => { console.log(request); return request},
      rejected: () => `${err}${request}`
    })
  })

  let responseInters = ['response1', 'response2', 'response3']
  responseInters.forEach(response => {
    chain.push({
      resolved: () => { console.log(response); return response},
      rejected: () => `${err}${response}`
    })
  })

  let promise = Promise.resolve('config')

  while (chain.length) {
    let { resolved, rejected } = chain.shift()
    // 精华都是在这句

    promise = promise.then(resolved, rejected)
  }
  // return promise调用链
  // promise.then(resolved1, rejected1).then(resolved2, rejected2).then(resolved2, rejected3).then() ....
  return promise
}

requestPromise().then(res => {

  console.log(res)
  // request3
  // request2
  // request1
  // xhr
  // response1
  // response2
  // response3
  // response3
})
```

---

## **Promise内部状态**

```
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    console.log('once')
    resolve('success')
  }, 1000)
})

const start = Date.now()
promise.then((res) => {
  console.log(res, Date.now() - start)
})
promise.then((res) => {
  console.log(res, Date.now() - start)
})
```

运行结果

```
once
success 1002
success 1002
```

解释: promise的.then或者.catch可以被调用多次,但是里Promise构造函数只执行一次.或者说,promise内部状态一经改变,并且有了一个值,则后续在每次调用.then或者.catch时都会直直接拿到该值

这在我们平时的promise缓存中是很有用的,比如app启动时可以暂存promise,在没有返回时,将promise直接给其他调用方,而不是每次调用都去new一个新的promise

---

## **Promise错误的正常抛出**

```
Promise.resolve().then(() => {
  return new Error('error!!!')
}).then(res => {
  console.log('then: ',res)
}).catch(err => {
  console.log('catch: ',err)
})
```

运行结果

```
then: Error: error!!!
  at Promise.resolve.then (...)
  at ...  
```

解释: .then或者.catch中return一个error对象并不会招聘错误,所以不会被后续的.catch捕获,需要改成如下一种

1. `return Promise.reject(new Error('error!!!'))`
2. `throw new Error('error')`

因为返回任意一个非promise的值都会包裹成promise对象,即 `return new Error('error!!!')` 等价于 `return Promise.resolve(new Error('error!!!'))`

---

## **小心Promise死循环**

```
const promise = Promise.resolve().then(() => {
  return promise
})
promise.catch(console.error)
```

运行结果

```
TypeError: Chaining cycle detected for promise #<Promise>
    at <anonymous>
    at process._tickCallback (internal/process/next_tick.js:188:7)
    at Function.Module.runMain (module.js:667:11)
    at startup (bootstrap_node.js:187:16)
    at bootstrap_node.js:607:3
```

解释: .then或.catch返回的值不能是promise本身,否则会造成死循环.类似于:

```
process.nextTick(function tick(){
  console.log('tick')
  process.nextTick(tick)
})
```

---

## **Promise值穿透**

```
Promise.resolve(1).then(2)
.then(Promise.resolve(3))
.then(console.log)
```

运行结果

```
1
```

解释: .then或者.catch的参数期望是函数,传入非函数则会发生值穿透

---

## **Promise捕获错误**

```
Promise.resolve()
.then(function success(res){
  throw new Error('error')
},function fail(e){
  console.error('fail',e)
})
.catch(function fail2(e){
  console.error('fail2',e)
})
```

运行结果

```
fail2: Error: error
  at success (...)
  at ...
```

解释: .then可以接收两个参数,第1个是处理成功的函数第2个是处理错误的函数. .catch是.then第2个参数的简便写法,但是在用法上有一点需要注意: /then的第2个处理错误的函数(fail1)捕获不了第1个处理成功的函数(success)抛出的错误,而后续的.catch方法(fail2)可以行政区之前的错误.当然,以下代码也可以:

```
Promise.resolve()
.then(function success1(res){
  throw new Error('error')
},function fail1(e){
  console.error('fail: ',e)
})
.then(function success2(res){

},function fail2(e){
  console.error('fail2: ',e)
})
```

---

## **Promise与事件循环**

```
Promise.resolve()
.then(() => {
  console.log('then')
})
process.nextTick(() => {
  console.log('nextTick')
})
setTimeout(() => {
  console.log('setImmediate')
})
console.log('end')
```

运行结果

```
end
nextTick
then
setImmediate
```

解释: process.nextTick和promise.then都属于microtask(但process.nextTick的优先级大于promise.then),而setImmediate属于macrotask,在事件循环的check阶段执行.带伤循环的每个阶段(macrotask)之间都会执行microtask,以上代码本身(macrotask)在执行完后会执行一次microtask