## 什么函数式编程

当下,函数式编程已经成为JavaScript界非常热门的一个话题.

函数式编程是通过组合纯函数来构建软件的过程,避免共享状态,数据可变和副作用.函数式编程是声明式而非命令式的,应用程序的状态通过纯函数流动.与面向对象编程不同,它的程序状态通常与方法共享和协作.

函数式编程是一种编程范式,这意味着它是基于一些特定和原则(上面列出的)去考虑软件架构的方法论.其他的编程范式还包括面向对象和面向过程编程.

相比于命令式或面向对象的代码,函数式代码更加简洁,可预测和易测试,但如果你不熟悉函数式编程以及与之相关的常见模式,那么函数式代码看起来会更加密集,并且与之相关的文章对于新手来说也很难理解.

如果你开始在何上搜索函数式编程的术语,你很快就会遇到一堵学术术语的砖墙,这对于初学者来说是非常可怕的.但如果你已经用JavaScript编程有一段时间,那么很可能已经在实际的软件开发开发中使用了很多函数式编程的思想和实用函数.

> 不要让所有的生词把你吓跑,这远比它听起来要简单得多

在开始理解函数式编程的含义之前,你需要理解上面那个看似简单的定义中的几个概念

- 纯函数
- 函数组合
- 避免共享状态
- 避免改变状态
- 避免副作用

换句话说,如果你想知道函数式编程在实践上意味着什么,那你就必须先理解这些核心概念.

纯函数有两个特点:

- 给定相同的输入,总会返回相同的输出
- 无副作用

在函数式编程中纯函数还有很多重要特性,包括引用透明(指的是函数运行不依赖于外部变量或状态).

函数组合是将两个或两个以上函数组合形成一个新函数的过程.例如,复函数`f(g(x))`.

---

### **共享状态**

共享状态是指任何变量,对象或内存空间存在与共享作用域下,或者对象的属性在作用域之间传递.共享作用域可以包括全局作用域或闭包作用域,通常,在面向对象编程中,对象通过向其他对象添加属性的方式在作用域之间共享.

比如计算机游戏可能有一个主游戏对象,其中的角色和道具作为该对象的属性存储.函数式编程避免了共享状态,依赖不可变的数据结构和纯计算从现有数据派生新数据.

共享状态的问题在于,为了理解函数的效果,你必须了解函数中使用影响的每一个共享变量的完整历史.

追假设你有不念旧恶需要保存的`user`对象,使用`saveUser()`函数向服务器上的一个API发起请求,在这个过程中,用户使用`updateAvatar()`更改头像,并触发了另一个`saveUser()`请求.在保存时,服务器返回一个远东的`user`对象,该对象应该替换内在中的任何内容,以便与服务器上发生的更改同步,或者响应其他API的调用.

不幸的是,第二个`saveUser`响应在第一个`saveUser`响应之前被接收,所以当第一个(现在已经过时了)响应返回时,新头像将在内在中删除,并被替换为旧的.这是竟态条件中,一个与共享状态相关的非常常见的不bug.

与共享状态相关的另一个问题是,更改函数的调用顺序可能会导致级联故障,因为作用于共享状态里面的函数依赖于时序.

```
// 对于共享状态,函数的调用会影响结果
const x - {
  val:2
}
const x1 = () => x.val += 1
const x2 = () => x.val *= 2
x1()
x2()
console.log(x.val)  //6

// 这个例子于上面的相同
const y = {
  val : 2
}
const y1 = () => y.val += 1
const y2 = () => y.val *= 2
// 倒置调用顺序
y2()
y1()
console.log(y.val)  //5
```

当避免了共享状态时,函数调用的时序不会改变结果.对于纯函数,给定相同的输入,你总是得到相同的输出,这使得函数调用完全独立于其他函数的调用,从而在根本上简化修改和重构.

```
const x = {
    val: 2
};
const x1 = x => Object.assign({}, x, { val: x.val + 1});
const x2 = x => Object.assign({}, x, { val: x.val * 2});
console.log(x1(x2(x)).val); // 5

const y = {
    val: 2
};

x2(y);
x1(y);

console.log(x1(x2(y)).val); // 5
```

在上面的例子中,我们使用`Object.assign()`并且传递一个空对象作为第一个参数去复制对象x中的属性,在这种情况下,它相当于从零创建一个对象.

如果你仔细看了本例中的`console.log`语句,你会注意到一些我们前面已经提到过的内容:函数组合.前面我们的函数组合是这样的:`f(g(x))`,但是在这个例子中,我们分别用`x1()`和`x2()`替换了原来的`f()`和`g()`以获得新组合.

当然,如果更改组合函数中的顺序,输出的结果也会相应改变.操作的顺序还是很重要的.`f(g(x))`并不总等价于`g(f(x))`,但是函数外部的变量会发生什么已经变得不再重要.对于非纯函数,除非知道函数使用或影响的每个变量完成的奋然,否则不可能完全理解这个函数的作用.

消除函数调用的时序依赖性,也就消除了一整类潜在的bug.

### **不可变性**

不可变性的对象在其创建后不可再更改,相反地,可变对象是任何创建之后任然可以被修改.不变性是函数式编程中的核心概念.因它没有它,程序中的数据流就会丢失.状态的历史丢失,奇怪的bug就会蔓延在你的软件中.

在JavaScript中.不能混淆const和不娈性.const创建了一个变量名绑定,该绑定在创建之后不可再赋值.const并没有创建一个不可变的对象,我们仍然可以更改对象的属性,这意味着const创建的绑定对象是可变的,而非不可变.

通过深度冻结对象,可以全对象真正不可变.JavaScript中有一个方法可以将对象的第一层进行冻结.

```
const a = Object.freeze({
  foo:'Hello',
  bar:'world',
  baz:'!'
})
a.foo = 'Goodbye'
// Error:Cannot assign to read only porperty 'foo' of object Object
```

> 在非`strict`模式下,浏览器或者屡有环境对`a.foo`赋值不会引起摄氏,但也无法修改值

但冻结的对象只是第一层不可变.例如,下面的对象仍然可变:

```
const a = Object.freeze({
  foo:{greeting:'Hello'},
  bar:'world',
  baz:'!'
})

a.foo.greeting = 'Goodbye'
console.log(`${a.foo.greeding}, ${a.bar}${a.baz}`)
// Goodbye world!
```

正如你所见,冻结对象中顶层的原始属性不可变,但对于同时也是对象的任何属性(包括Array等)仍然可变,因此即便冻结对象也是可变的,除非遍历整个对象树并冻结每一个对象属性.

在许多函数式编程语言中,有一种特殊的不可变数据结构称为tire数据结构(又称字典树,发音tree),它实际上是深度冻结的,这意味着不论对象的巍峨层级有多深,属性都不可更改.

tires(字典树),通过结构共享的方式共享对象中所有节点的内在引用地址,这些内在地址在对象被操作符复制之后保持不变,这将消耗更少的内在,并为某些类型的操作带来显著的性能提升.

例如,你可以在对象的要位置通过标识进行比较,如果标识相同,则不必遍历整个树去检查差异.

---

### **副作用**

副作用是指在被调用函数之外可被察觉在除了返回值之外的任何应用程序状态的更改,包括:

- 修改任何外部变量或对象属性(例如全局变量和父函数作用域链中的变量)
- 打印日志到控制台
- 在屏幕上输出
- 定稿文件
- 发起网络请求
- 触发外部程序
- 调用其他有副作用的函数

函数式编程通常可以避免副作用,这使得程序的效果更易理解和测试

Haskell和其他函数语言经常使用Mondas从纯函数中分享和封装副作用.

你现在需要明白的是,副作用的操作需要与软件的其他部分隔离.如果将副作用与程序逻辑部分分开,你的软件奖变得更易扩展,重构,调试,测试和维护.

这也是大多数前端框架鼓励用户在独立的,低耦合在模块中和组件渲染的原因.

---

### **通过高阶函数实现利用**

函数式编程倾向于重复利用一组公共函数库来处理数据,面向对象编程则倾向于将对象和数据放在一起,这些方法对操作的数据有类型的限制,并且通常只能对特定类实例中的数据进行操作.

在函数式编程中,所有的数据类型都是对等的.同一个`map()`工具函数可以映射对象,字符串,数字或其他数据类型,因为它接收一个函数作为参数,该参数会适当地处理给定的数据类型.

JavaScript中函数是一等公民,允许我们把函数当作数据看待,把它们赋值给变量,作为参数传入函数或者从函数中返回等等.

高阶函数是以函数为参数,返回函数或者两者兼有的函数.高阶函数通常用于:

- 通过回调,Promise,Mondas抽离或隔离操作,效果和异步流控制.
- 创建可以处理多种数据类型的工具函数
- 将函数作为参数或者创建一个柯里化的函数以用于重用或函数组合
- 获取函数列表并返回这些输入函数的一些组合

---

### **容器,函子,列表和流**

函子是可映射的.换句话说,它是一个容器,包含了一个接口,可以将函数应用于其中的值,当你看到函子这个词,你就应该想到可映射的.

前面我们了解了一个`map()`可以作用于各种数据类型.它通过函子API提升映射操作来实现这一点.`map()`中很重要的是流控制操作使用了该接口.对于`Array.prototype.map()`.容器是一个数组,当然,其他数据结构也可以是函子,只要它们提供映射API.

让我们看看`Array.prototype.map()`是如何允许你从映射实用程序中抽象数据类型,从而便`map()`可以用于任何数据类型的.下面我们将创建一个`double`映射,它会将每上个传入的值乘以2:

```
const double = n => n * 2
const doubleMap = numbers => numbers.map(double)
console.log(doubleMap[2,3,4])
// [4,6,8]
```

如果我们想给下面对象中的点数翻倍呢?我们所要做的仅是对传递给`map()`和`double()`函数做一些细微的修改,一切仍然正常:

```
const double = n => n.points * 2
const doubleMap = numbers => map(double)

console.log(doubleMap([
  {name:'ball',points:2},
  {name:'coin',points:2},
  {name:'candy',points:2}
]))
// [4,6,8]
```

在函数式编程中,使用诸如函子和高阶函数这样的抽象并利用工具函数操作任意数据类型的思想非常重要.

> 流即是随着时间推移而变化的列表

现在你需要清楚的是,数组和函子不是应用容器和其中的概念的唯一方式.例如,数组是一个列表,随着时间推移所表达的列表是一个流,因此你可以应用相同类型的工具函数来处理传入带伤流.当你真正开始使用FP构建软件时,你会经常看到这种情况.

---

### **声明式 VS 命令式**

函数式编程是一种声明范式,这意味着程序逻辑的表达不需要显示地描述控制流.

命令式程序花费数行代码去描述用于实现预期结果所需的特定步骤,流程控制.

声明式程序抽象了流程控制过程,并且通过数行代码描述数据流.

例如,这个命令式的映射接受一个数字数组,并返回每个元素乘以2后的新数组:

```
const doubleMap = numbers => {
  const doubled = []
  for(let i=0;i<numbers.length;i++){
    doubled.push(numbers[i]*2)
  }
  return doubled
}
console.log(doubleMap([2,3,4]))
// [4,6,8]
```

这个声明式映射做了同样的事情,通过`Array.protitype.map()`函数抽象了控制流,使得我们可以更清晰地表达数据流.

```
const doubleMap = numbers => numbers.ma(n => n*2)
console.log(doubleMap([2,3,4]))
// [4,6,8]
```

命令式代码经常使用诸如`for` , `if` , `switch` , `thow`等这样的语句.

而声明式代码更多地依赖于表达式,表达式是一段用于计算某个值的代码,通常是用于产生相应结果值的函数调用,值和运算符的某种组合.

这些表达式的例子:
```
2*2
doubleMap([2,3,4])
Math.max(4,3,2)
```

通常在代码中,你会看到将表达式赋值给标识符,从函数返回或传递给函数,在赋值,返回或传递之前,首先会对表达式求值,然后使用结果值.

---

### **结语**

函数式编程支持:

- 纯函数而非共享状态和产生副作用
- 不可变性替代易变数据
- 函数组合替代命令控制流
- 许多通用的,可利用的工具函数借助高阶函数作用于各种数据类型
- 声明式的代码而不是命令式的代码(做什么而不是怎么做)
- 表达式替代语句
- 窗口和高阶函数替代多态性