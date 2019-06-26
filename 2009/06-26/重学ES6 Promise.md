## **Promise含义**

Promise是异步编程的一种解决方案,比传统的回调更加合理,更加强大.

Promise简单来说是一个窗口,里面存放着某个未来才会结束的结果.Promise是一个对象,用来获取异步操作的消息.

Promise有以下两个特点:

1. 对象不爱外界影响  
有三种状态:Pending(进行中),Fulfilled(已成功),Rjected(已失败).只有异步操作的结果可以决定当前是哪一种状态,任何操作都无法改变这个状态.

2. 一旦改变,不会再变,任何时候都可以得到这个结果.  
Promise对象的状态改变只有两种可能:从Pending到Fulfilled或者从Pending到Rejected.只要这两种情况发生,状态就会凝固,不会再变,一直保持这个结果.这时就称为Resolved(已定型).就算改变已发生,再对Promise添加回调函数,也会得到这个结果.这与事件(event)完全不同,事件的特点就是:如果错过了,再去监听是得不到结果的.

Promise缺点:

1. 无法取消,一旦建立就会立即执行,无法中途取消.
2. 如果不设置回调函数,Promise内部错误不会抛出到外面.
3. 当处于Pending状态,无法得知目前进展到哪一阶段(是刚刚开始,还是即将完成)

---

## **基本用法**

```
var promise = new Promise((resolve,reject) => {
  // ...

  if(/*...*/){
    resolve(value)
  }else{
    reject)error
  }
})
```

resolve作用是将Promise状态由未完成编程成功,在异步操作成功时调用,并将异步结果作为参数传播出去.

reject函数作用是将Promise状态从未完成变为失败,在异步操作失败时调用,并将异步操作失败报出错误传递出去.

Promise实例生成后,可以调用then,指定resolved和rejected状态的回调函数.

```
promise.then(val =>{
  //success
},error => {
  //failure
})
```

then方法接受两个回调函数作为参数.第一个是Promise对象的状态变为Resolved时调用,第二个回调是Promise变为Rejected时调用.第二个函数是可选的.

```
function timeout(ms){
  return new Promise((resolve,reject) => {
    setTimeout(resolve,ms,'done')
  })
}

timeout(100).then(value => {
  console.log(value)
})
```

Promise新建好之后就会立即执行.

```
let promise = new Promise((resolve,reject) => {
  console.log('Promise')
  resolve()
})

promise.then(() => {
  console.log('resolved')
})

console.log('hi')

// Promise
// hi
// resolved
```

异步加载图片

```
function loadImageAsync(url){
  return new Promise((resolve,reject) => {
    var img = new Image()
    img.onload = () => {
      resolve(img)
    }
    img.onerror = () => {
      reject(new Error('could not load img at ' + url))
    }
    img.src = url
  })
}
```

Promise实现ajax

```
var getJSON = function (url) {
  var promise = new Promise((resolve,reject) => {
    var client = new XMLHttpRequest()
    client.open('GET',url)
    client.onreadystatechange = handle
    client.responseType = 'json'
    client.setRequestHeader('Accept','application/json')
    client.send()

    function handler(){
      if(this.readyState !== 4){
        return
      }
      if(this.status === 200){
        reolve(this.response)
      }else{
        reject(new Error(this.statusText))
      }
    }

  })
  return promise
}
getJSON('/post.json').then(json => {
  console.log('Contents ' + json)
},error => {
  console.log(error)
})
```

```
var p1 = new Promise((resolve,reject) => {
  setTimeout(() => {
    reject(new Error('fail'),300)
  })
})

var p2 = new Promise((resolve,reject) => {
  setTimeout(() => {
    resolve(p1)
  },1000)
})

p2.then(result => console.log(result))
.catch(error => console.log(error))
// Error:fail
```

上船来说,调用resolve或者reject以后,Promise的使命就已经完成了,后面的操作应该放到then,而不应该直接写到resolve或者reject后面,最好在它们之前添加return语句,这样就不会出现意外.

```
new Promise((resolve,reject) => {
  return resolve(1)
  // 后面不会执行
  console.log(2)
})
```

---

## **Promise.prototype.then()**

Promise实例具有then方法,作用是为Promise实例添加状态改变时的回调函数.

then方法返回一个新的Promise实例(不是原来的那个Promise),因此可以使用链式写法,then后面再添加另外一个then.

```
getJSON('/posts.json').then(json => {
  return json.post
}).then(post => {
  // ...
})
```

链式调用的then可以指定一组按照次序调用的回调函数.前一个回调函数可能返回的还是一个Promise,后一个回调函数就会等待该Promise对象的状态发生变化,在被调用.

```
getJSON('/post/1.json').then(post => {
  return getJSON(post.commentUrl)
}).then(function funcA(comments){
  console.log('resolved')
},function funcB(error){
  console.log(error)
})
```

---

## **Promise.prototype.catch()**

这个方法是`.then(null,rejection)的别名,指定发生错误的回调.

```
getJSON('/posts.json').then(posts => {
  // ...
}).catch(error => {
  console.log('error ' + error)
})
```

Promise在resolve语句之后抛出错误,并不会被捕获,等于没有抛出.因为Promise状态一旦发生改变,就会永久保存这个状态,不会改变了.

Promise对象错误具有冒泡性质,会一起向后传递,走到捕获为止,错误总会被下一个catch捕获.

一般不要在then方法中定义Rejected状态的回调函数,而应总是使用catch方法

```
var someAsyncThing = () => {
  return new Promise((resolve,reject) => {
    // 报错 x 未声明
    resolve(x + 2)
  })
}

someAsyncThing().then(() => {
  return someotherAsyncThing()
}).catch(error => {
  console.log('oh no ',error)
  // 报错 y 没有声明
  y + 2
}).then(() => {
  console.log('carry on')
})
// oh no [ReferenceError: x is not defined]
```

catch方法抛出一个错误,后面如果没有catch,导致这个错误不会被捕获,也不地传递到外层.

可以用第二个catch方法捕获前一个catch方法抛出的错误.

```
someAsyncThing().then(() => {
  return someotherAsyncThing()
}).catch(error => {
  console.log('oh no',error)
  // 报错 y 没有声明
  y + 2
}).catch(error => {
  console.log('carry on ',error)
})
```

---

## **Promise.all()**

将多个Promise实例,包装成一个新的Promise实例

```
var p = Promise.all([p1,p2,p3])
```

p状态由p1,p2,p3决定,分两种情况

1. 只有p1,p2,p3状态都变成fulfilled,p的状态才会变成fulfilled,此时,p1,p2,p3返回值组成一个数组,传递给p的回调.

2. 只要p1,p2,p3有一个被Rejected,p状态也会变成Rejected,此时第一个被Rejected的实例返回值会传递给p的回调函数.

```
var promises = [2,3,5,7,11,13].map(id => {
  return getJSON('/post' + id + '.json')
})

promise.all(promises).then(posts => {
  // ...
}).catch(reason => {
  // ...
})
```

另一个例子

```
const databasePromise = contentDatabase()
const bookPromise = databasePromise.then(findAllBooks)
const userPromise = databasePromise.then(getCurrentUser)

Promise.all([bookPromise,userPromise]).then(([books,user]) => {
  // ...
})
```

如果作为参数的Prmoise自身定义了catch方法,那么被rejected时并不会触发Promise.all的catch方法.

---

## **Promise.race()**

```
var p = new Promise.race([p1,p2,p3])
```

只要p1,p2,p3中有一个实例率先改变状态,p的状态就跟着改变,那个率先改变的Promise实例,返回值就是传递给p的回调函数.

```
const p = Promise.race([
  fetch('/xxx'),
  new Promise((resolve,reject) => {
    setTimeout(() => {
      reject(new Error('time out'))
    },5000)
  })
])
p.then(res => {
  console.log)res
}).catch(error => {
  console.log(error)
})
```

---

## **附加方法**

**done**

无论Promise对象的回调链以then方法还是catch结尾,最后一个方法抛出的错误都可能无法捕获到(Promise内部的错误不会冒泡的全局),因此提供一个done方法,总是牌回调链尾端,保证抛出任何可能的错误.

```
asyncFunc()
.then()
.catch()
.then()
.done()
```

**finally()**

finnaly方法用于指定不管Promise对象最后状态如何都会执行的操作.与done最大的区别在于他接受一个普通的回调函数作为参数,该函数不管怎么样都必须执行.