JavaScript是单线程语言, 以前异步编程有四种方式

- 回调函数
- 事件监听
- 发布/订阅
- Promise 对象

现在据说异步编程的终极解决方案是 async/await

---

### **回调函数**

所谓回调函数 (callback), 就是把任务分成两步完成,第二步单独写在一个函数里,等到重新执行这个任务时,就直接调用这个函数.

例如,在node中读取文件

```
fs.readFile('a.txt', (err, data) => {
  if(err) throw err
  console.log(data)
})
```

上面代码中 readFile 的第二个参数就是回调函数, 等到读取完 a.txt 文件时, 这个函数才会执行.

---

### **Prmise**

使用回调函数本身没有问题,但有回调地狱的问题

假定我们有一个需求, 读完 A 文件后读取 B 文件, 再读取 C 文件, 代码如下

```
fs.readFile(fileA, (err, data) => {
  fs.readFile(fileB, (err, data) => {
    fs.readFile(fileC, (err, data) => {
      // do something
    })
  })
})
```

可网民, 三个回调函数代码看来已经够呛了, 有时候实际业务中还不止嵌套这几个, 难以管理.

这时候 Promise 出现了! 它不是新的功能,而是一种新的写法, 用来解决回调地狱的问题.

我们再来假定一个业务, 分多个步骤完成, 每个步骤篮子是异步的, 而且依赖于上一个步骤结果, 用 setTimeout() 来模拟异步操作

```
/**
* 传入参数 n,表示这个函数执行的时间(毫秒)
* 执行的结果是 n + 200, 这个值将用于下一步骤
*/
function A(n) {
  return new Promise(resolve => {
    setTimeout(() => resolve(n + 200), n)
  })
}

function step1(n) {
  console.log(`step1 with ${n}`)
  return A(n)
}

function step2(n) {
  console.log(`step2 with ${n}`)
  return A(n)
}

function step3(n) {
  console.log(`step3 with ${n}`)
  return A(n)
}
```

上面代码中有4个函数, A() 返回一个 Promise 对象, 接收参数 n, n 秒后执行 resolve(n + 200). step1, step2, step3对应三个步骤.

现在用 Promise 实现三个步骤:

```
function doIt() {
  console.log('do it now')
  const time1 = 300
  step1(time1)
    .then(time2 => step2(time2))
    .then(time3 => step3(time3))
    .then(result => {
      console.log(`result is ${result}`)
    })
}
doIt()
```

可见, Promise 的写法只是回调函数的改进, 用 then() 方法免去了嵌套, 更为直观.
但这样写绝对不是最好的,代码变得十分冗余,一堆then.

---

### **Generator 函数**

Generator 是协程在 ES6 的实现, 最大的特点就是可以交出函数的执行权, 懂得退让.

```
function* gen(x) {
  var y = yield x +2
  return y
}

var g = gen(1)
console.log(g.next())
console.log(g.next())
```

上面代码中, 函数多了 * 号, 用来表示这是一个 Generator 函数, 和普通函数不一样, 不同之处在于执行它不返回结果.

返回的是指针对象 g, 这个指针 g 有一个 next 方法, 调用它会执行异步任务的第一步.

对象中有两个值, value 和 node, value 属性是 yield 语句后面表达式的值, 表示当前阶段的值, done 表示是否 Generator 函数是否执行完毕.

下面看看 Generator 函数如何执行一个真实的异步任务

```
var fetch = require('node-fetch')

function* gen() {
  var url = 'https://api.github.com/users/github'
  var result = yield fetch(url)
  console.log(result.bio)
}

var g = gen()
var result = g.next()
result.value.then(data => return data.json)
  .then(data => g.next(data))
```

上面代码中, 首先执行 Generator 函数, 得到对象 g, 调用 next 方法, 此时  
`result = {value: Promise {<pending>}, done: false}`  
因为 fetch 返回的是一个 Promise 对象, (即 value 是一个 Promise 对象) 所以要用 then 才能调用下一个 next 方法.

虽然 Generator 将异步操作表示很简洁, 但是管理麻烦, 何时执行第一阶段, 何时执行第二阶段?

---

### **Async/await**

从回调函数, 到 Promise 对象, 再到 Generator 函数, JavaScript 异步编程解决方案历程可谓辛酸.终于到了 Async/await, 很多人认为它是异步操作的最终解决方案.

其实 async 函数就是 Generator 函数的语法糖, 例如下面两段代码

```
// generator 函数
var gen = function* () {
  var f1 = yield readFile('a.txt')
  var f2 = yield readFile('b.txt')
  console.log(f1.toString())
  console.log(f2.toString())
}

// async 函数
var asyncReadFile = async function() {
  var f1 = await readFile('a.txt')
  var f2 = await readFile('b.txt')
  connsole.log(f1.toString())
  console.log(f2.toString())
}
```

比较可发现, 两个函数其实是一样的, async 不过是把 Generator 函数的 * 号换成 async, yield 换成await

**1. async 函数的用法**

async 返回的是一个 Promise 对象, 如果直接 return 一个直接量, async 会把这个直接量通过 Promisee.resolve() 封装成 Promise 对象.

> 注意: 一般来说,都认为 await 是等待一个 async 函数完成, 确切地说是一个表达式, 这个表达式的计算结果是 Promise 对象或者是其他值(没有限定是什么), 即 await 后面不仅可以接 Promise, 还可以接普通函数或直接量.

同时,我们可以把 async 理解为一个运算符, 用于组成表达式, 表达式的结果取决于它等到的东西

- 等到非 Promise 对象, 表达式结果为它等到的东西
- 等到 Promise 对象, await 就会阻塞后面的代码, 等待 Promise 对象或 resolve, 取得 resolve 的值, 作为表达式的结果

还是那个业务, 分多个步骤完成, 每个步骤依赖于上一个步骤的结果, 用setTimeout模拟异步操作.

```
async function doIt() {
  console.time('doIt')
  const time1 = 300
  const time2 = await step1(time1)
  const time3 = await step2(time2)
  const result = await step3(time3)
  console.log(`result is ${result}`)
  console.timeEnd('doIt')
}
doIt()
```

输出结果和 Promise 实现是一样的,但这个代码结构看起来清晰得多,几乎跟同步写法一样

**2. async 函数的优点**

- 内置执行器  
Generator 函数的执行必须靠执行器, 所以才有了 co 函数库, 而 async 函数自带执行器. 也就是说, async 函数的执行与普通函数一模一样, 只要一行.

- 语义化更好  
async 和 await, 比起星号和 yield, 语义更清晰了. async 是异步的简写, 而 await 可以认为是 async await 的简写. 所以应该很好理解 async 用于申明一个 function 是异步的, 而 await 用于等待一个异步方法执行完成.

- 更广的适用性  
yield 命令后面只能是 Thunk 函数或 Promise 对象, 而 async 函数的 await 命令后面, 可以跟 Promise 和原始类型的值 (数值, 字符串, 布尔值, 但这时赞同于同步操作)