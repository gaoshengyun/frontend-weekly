## JavaScript 43道面试题

### **1. 下面代码输出什么**
```
function sayHi() {
  console.log(name)
  console.log(age)
  var name = 'Lee'
  let age = 22
}
sayHi()
``` 
  - A: `Lee`和`undefined`
  - B: `Lee`和`ReferenceError`
  - C: `ReferenceError`和`22`
  - D: `undefined`和`ReferenceError`

## 答案: D 
在函数中,我们首先使用`var`关键字声明了`name`变量.这意味着变量在创建阶段会被提升(JavaScript会在创建变量阶段为其分配内存空间),默认值为`undefined`,直到我们实际执行到使用该变量的行.我们还没有为`name`变量赋值,所以它仍然保持`undefined`的值.

使用`let`关键字和`const`关键字声明的变量提升,但与`var`不同,初始化没有被提升.在我们声明(初始化)它们之前,它们是不可访问的.这被称为暂时死区.当我们在声明变量之前尝试访问变量时,JavaScript会排除一个`ReferenceError`.  

---

### **2. 下面代码输出什么**
```
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i),100)
}

for(let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i),100)
}
```
- A: `0  1  2` 和 `0  1  2`
- B: `0  1  2` 和 `3  3  3`
- C: `3  3  3` 和 `0  1  2`

## 答案: C

由于JavaScript中的事件执行机制,`setTimeout`函数真正被执行时,循环已经走完了.由于第一个循环中的变量`i`是使用`var`关键字声明的,因此该值是全局的.在循环期间,我们每次使用一元运算符`++`都会将`i`的值增加`1`因此在第一个例子中,当调用`setTimeout`函数时,`i`已经被赋值为`3`

第二个循环中,使用`let`关键字声明变量`i`,使用`let`和`const`关键字声明的亦是是具有块作用域的(块是`{}`之间在任何东西).在每次迭代期间,`i`将被创建为一个新值,并且每个值都会在于于循环的内的块级作用域.

---
## **3. 下面代码输出什么**
```
const shape = {
  radius: 10,
  diameter() {
    return this.radius * 2
  },
  perimeter: () => 2 * Math.PI * this.radius
}

shape.diameter()
shape.perimeter()
```
- A: 20 和 62.83185307179586
- B: 20 和 NaN
- C: 20 和 63
- D: NaN 和 63

## 答案: B

`diameter`是普通函数,而`perimeter`是箭头函数

对于箭头函数,`this`关键字指向是它所在上下文(定义时的位置)的环境,与普通函数不同~这意味着当我们调用`perimeter`时,它不是指向`shape`对象,而是指其定义时的环境(window).没有值`radius`的属性,返回`undefined`

---

## **4. 下面代码输出什么**
```
+true
!'Lee'
```
- A: `1` 和 `false`
- B: `false` 和 `NaN`
- C: `false` 和 `false`
## 答案: A

一元加号会尝试将`boolean`类型转换为数字类型. `true`被转换为`,`false`被转换为`0`

字符串`Lee`是一个真值.我们实际上要问的是`这个值是假的吗?`.这会返回`false`

---

## **5. 哪个选项是不正确的**
```
const bird = {
  size: 'small'
}

const mouse = {
  name: 'Mickey',
  small : true
}
```
- A: mouse.bird.size
- B: mouse[bird.size]
- C: mouse[bird['size]]
## 答案: A

在JavaScript中,所有对象键都是字符串(除了`Symbol`).尽管有时我们可能不会给定字符串类型,但它们总是会被转换为字符串.

JavaScript解释语句.当我们使用方括号表示法时,安会看到第一个左括号`[`,然后继续,直到找到右括号`]`.只有在那个时候,它才会对这个语句求值.

`mouse[bird.size]` : 首先它会对`bird.size`求值,得到`small`.`mouse['small']返回`true.

但是,使用点表示法,这不会发生.`mouse`没有名为`bird`的键,这意味着`mouse.bird`是`undefined`.然后,我们使用点符号来询问`size`,`mouse.bird.size`.这是无效的并抛出`Cannot read property 'size' if undefined`

---

## **6. 下面代码输出什么**
```
let c = { gtrtting: '`hey!' }
let d

d = c

c.greeting = 'Hello'
console.log(d.greeting)
```
- A: Hello
- B: undefined
- C: ReferenceError
- D: TypeError

## 答案: A

在JavaScript中,当设置它们彼此相等时,所有对象都通过引用进行交互

首先变量`c`为对象保存一个值.之后,我们将`d`指定为`c`与对象相同的引用.

更改一个对象时,可以更改所有对象.

---

## **7. 下面代码输出什么**
```
let a = 3
let b = new Number(3)
let c = 3

console.log(a == b)
console.log(a === b)
console.log(b === c)
```
- A: true  false  true
- B: false false true
- C: false true true
## 答案: C

`new Number()`是一个内置的函数构造函数,虽然它看起来像一个数字,但它并不是一个真正的数字,它有一堆额外在功能,是一个对象

当我们使用`==`运算符时,它只检查它是否具有相同的值.它们都有3的值,所以它返回`true`

> `==`会引发隐式类型转换,右侧的对象类型会自动转换为`Number`类型.

然而当我们使用`===`操作符时,类型和值都需要相等,`new Number()`不是一个数字,是一个对象类型.两者都返回`false`

---

## **8. 下面代码输出什么**
```
class Chameleon {
  static color Change(newColor) {
    this.newColor = newColor
  }

  constructor( {newColor= 'green'} = {} ){
    this.newColor = newColor
  }
}
const freddie = new Chameleon({newColor:'purple'})
freddie.colorChange('orange')
```
- A: orange
- B: purple
- C: green
- D: TypeError
## 答案: D

`colorChange`方法是静态的.静态方法仅在创建它们的构造函数中存在,并且不能传递给任何子级.由于`freddie`是一个子级对象,函数不会传递,所以在`freddie`实例上不存在`freddie`方法,抛出`TypeError`.

---

## **9. 下面代码输出人什么**
```
let greeting
greetign = {}
console.log(greetign)
```
- A: {}
- B: RefereceError: greeting is node defined
- C: undefined
## 答案: A

控制台会输出空对象,因为我们刚刚在全局对象上创建了一个空对象!当我们错误地将`greeting`输入为`greetign`时,JavaScript解释器实际上在浏览器中视为`global.greetign = {}`或`window.greetign = {}`

为了避免这种情况,我们可以使用`use strict`.这可以确保在将变量赋值之前必须声明变量.

---

## **10. 当我们这样做会发生什么**
```
function bark() {
  console.log('woof')
}
bark.animal = 'dog'
```
- A: noting,this is total fine.
- B: SyntaxError.you cannot add properties to a function this way
- C: undefined
- D: ReferenceError
## 答案: A

这在JavaScript中是可能的,因为函数也是对象!(原始对象之外的所有东西都是对象)

函数是一种特殊类型的对象.您自己编写的代码并不是实际的函数.该函数是具有属性的对象,因此可以调用的.

---

## **11. 下面代码输出什么**
```
function Person(firstName,lastName) {
  this.firstName = firstName
  this.lastName = lastName
}

const member = new Person('Lydia','Hallie')
Person.getFullName = () => this.firstName + this.lastName

console.log(member.getFullName())
```
- A: TpeError
- B: SyntaxError
- C: Lydia Hallie
- D: undefined undefined
## 答案: A

您不能像使用常规对象那样向构造函数添加属性.如果要一次向所有对象头一回功能,必须使用原型.所以在这种情况下应该这样写
```
Person.prototype.getFullName = function(){
  return `${this.firstName} ${this.lastName}`
}
```
这样会使`member.getFullName()`是可用的,为什么这样是对的?假设我们将此方法添加到构造函数本身,也许不是每个`Person`实例都需要这种方法.这会浪费大量的内在空间,因为它们仍然具有该属性,这占用了每个实例的内在空间.相反,如果我们只将它添加到原型中,我们只需将它放在内在中的一个益,但它们都可以访问它!

---

## **12. 下面代码输出什么**

```
function Person(firstName,lastName) {
  this.firstName = firstName
  this.lastName = lastName
}

const lydia = new Person('Lydia','Hallie')
const sarah = Person('Sarah','Smith')

console.log(lydia)
console.log(sarah)
```
- A: `Person {firstName:'Lydia',lastName:'Hallie'}` 和 undefined
- B: `Person {firstName:'Lydia',lastName:'Hallie'}` 和 `Person {firstName:'Sarah',lastName:'Smith'}`
- C: `Person {firstName:'Lydia',lastName:'Hallie'}` 和 {}
- D: `Person {firstName:'Lydia',lastName:'Hallie'}` 和 ReferenceError
## 答案: A

对于`sarah`,我们没有使用`new`关键字.使用`new`时,它指的是我们创建的新空对象!但是,如果你不添加`new`它指的是全局对象.

我们指定了`this.firstName`等于`Sarah`和`this.lastName`等于`Smith`.我们实际做的定义是`global.firstName = 'Sarah'`和`global.lastName = 'Smith'`.`Sarah`本身的返回值是undefined

---

## **事件传播的三个阶段是什么**
- A: 目标 > 捕获 > 冒泡
- B: 冒泡 > 目标 > 捕获
- C: 目标 > 冒泡 > 捕获
- D: 捕获 > 目标 > 冒泡
## 答案: D

在捕获阶段,事件通过父元素向下传递到目标元素,然后它一达目标元素,冒泡开始.

---

## **13. 所有对象都有原型**
- A: 对
- B: 错误
## 答案: B

除基础对象外,所有对象都有原型.基础对象可以访问某些方法和属性,例如`.toString`.这就是你可以使用内置JavaScript方法的原因.所有这些方法都可以在原型上找到.虽然JavaScript无法直接在您的对象上找到它,但它会沿着原型链向下寻找并在那里找到它,这使你可以访问到它.

> 基础对象指原型链终点的对象.基础对象的原型是null

---

## **14. 下面代码的输出是什么**
```
function sum(a,b) {
  return a + b
}
sum (1,'2')
```
- A: NaN
- B: TypeError
- C: '12'
- D: 3
## 答案: C

JavaScript是一种动态类型的语言,我们没有指定某变量的类型.在不知道的情况下,值可以自动转换为另一种类型,称为隐式转换.强制从一种类型转换为另一种割开.

在此示例中,JavaScript将数字`1`转换为字符串,以使函数有意义并返回值.在让数字类型`1`和数字类型`2`相加时,该数字被视为字符串,我们可以连接字符串,`'1' + '2'`返回`12`.

---

## **15. 下面代码输出什么**
```
let number = 0
console.log(number++)
console.log(++number)
console.log(number)
```
- A: 1  2  3
- B: 1  2  2
- C: 0  2  2
- D: 0  1  2
## 答案: C

后缀一元运算符 `++`:  
1. 返回值(返回0)
2. 增加值(数字现在是1)

前缀一无运算符 `++`:
1. 增加值(数字现在是2)
2. 返回值(返回2)

所以返回0 2 2

---

## **16. 下面代码输出什么**
```
function getPersonInfo(one, two, three) {
  console.log(one)
  console.log(two)
  console.log(three)
}

const person = 'Lydia'
const age = 21

getPersonInfo`${person} is ${age} years old`
```
- A: Lydia 21 ['','is','years old']
- B: ['','is','years old'] Lydia 21
- C: Lydia ['','is','years old'] 21
## 答案: B

如果使用标记模板字符串,则第一个参数的值始终是字符串的数组.其余参数获取传递到模板字符串中的表达式的值!

---

## **17. 下面代码输出什么**
```
function checkAge(data){
  if(data === {age:18}){
    console.log('you are an adult')
  }else if(data == {age:18}){
    console.log('you are still an adult')
  }else{
    console.log('hmm...you do not have an age i guess')
  }
}

checkAge({age:18})
```
- A: you are an adult
- B: you are still an adult
- C: hmm...you do not have an age i guess
## 答案: C

在比较相等性,原始类型通过它们的值进行比较,而对象 通过它们的引用进行比较.JavaScript检查对象是否具有对内存中相同位置的引用.

我们作为参数传递的对象和我们用于检查相等性的对象在内在中位于不同位置,所以它们的引用是不同的.

这就是为什么`{age:18} === {age:18}`和`{age:18} == {age:18}`返回false的原因.

---

## **18. 下面代码输出什么**
```
function getAge(..args){
  console.log(typeof args)
}

getAge(21)
```
- A: 'number'
- B: 'array'
- C: 'object'
- D: 'NaN'
## 答案: C

扩展运算符(`...args`)返回一个带参数的数组.数组是一个以无明,因此`typeof args`返回`object`

---

## **20. 下面代码输出什么**
```
function getAge(){
  'use strict'
  age = 21
  console.log(age)
}

getAge()
```
- A: 21
- B: undefined
- C: ReferenceError
- D: TypeError
## 答案: C

使用`'use strict'`,可以确保不会意外地声明全局变量.我们从未声明过变量`age`,因为我们使用`'use strict'`,它就引发一个`ReferenceError`.如果我们不使用`'use strict'`,它就会起作用,因为属性`age`会被添加到全局对象中.

---

## **21. 下面代码输出什么**
```
cons sum = eval('10*10+5')
```
- A: 105
- B: '105'
- C: TypeError
- D: '10*10+5'
## 答案: A

`eval`会字符串传递的代码求值.如果它是一个表达式,就像在这种情况下一样,它会计算表达式.表达式为`10*10+5`计算得到的105

---

## **22. cool_secret可以访问多长时间**
```
sessionStirage.setTime('cool_secret',123)
```
- A: 永远,数据不会丢失
- B: 用户关闭选项卡时
- C: 当用户关闭整个浏览器时,不仅是选项卡
- D: 当用户关闭计算机时
## 答案: B

关闭选项卡后,将删除存储在`sessionStorage`中的数据

如果使用`localStorage`,数据将永远存在,除非例如调用`localStorage.clear()`

---

## **23. 下面代码输出什么**
```
var num = 8
var num = 10

console.log(num)
```
- A: 8
- B: 10
- C: SyntaxError
- D: ReferenceError
## 答案: B

使用关键字`var`,可以用相同的名称声明多个变量,然后变量将保存最新的值.

---

## **24. 下面代码输出什么**
```
const obj = {1:'a',2:'b',3:'c'}
const set = new Set([1,2,3,4,5])

obj.hasOwnProperty('1')
obj.hasOwnProperty(1)
set.has('1')
set.has(1)
```
- A: false true false true
- B: false true true true
- C: true true false true
- D: true true true true
## 答案: C

所有对象键(不包括`Symbols`)都会被存储为字符串,即使你没有给定字符串类型的键.这不是为什么`obj.hasOwnProperty('1')`也会返回true

上面的说法不适用于`Set`.在我们的`Set`中没有'1',set.has('1')返回`false.它有数字类型1,`set.has(1)`返回true

## **25. 下面代码输出什么**
```
const obj = {a:'one',b:'two',a:'three'}
console.log(obj)
```
- A: {a:'one',b:'two'}
- B: {b:'two',a:'three'}
- C: {a:'three',b:'two'}
- D: TyntaxError
## 答案: C

如果对象有两个具有相同名称的键,刚将替前面的键.它仍将处于第一个位置,但具有最后指定的值.

---

## **26. JavaScript全局执行上下文为你创建了两个东西:全局对象和this关键字**
- A: 对
- B: 错误
- C: 视情况而定
## 答案: A

基于执行上下文全局执行上下文:它是代码中随处可以访问的内容

---

## **27. 下面代码输出什么**
```
for(let i=0;i<5;i++){
  if(i===3) continue
  console.log(i)
}
```
- A: 1 2
- B: 1 2 3
- C: 1 2 4
- D: 1 3 4
## 答案: C

如果某个条件返回true,则continue语句跳过迭代

---

## **下面代码输出什么**
```
String.prototype.giveLydiaPizza = () => {
  return 'Just give Lydia pizza already!'
}
const name = 'Lydia'

name.giveLydiaPizza()
```
- A: 'Just give Lydia pizza already!'
- B: TypeError: not a function
- C: SyntaxError
- D: undefined
## 答案: A

`String`是一个内置的构造函数,我们可以为它添加属性.我刚给它的原型添加了一个方法.原始类型的字符串自动转换字符串对象,由字符串原型函数生成.因此,所有字符串都可以访问该方法!

---

## **29. 下面代码输出什么**
```
const a = {}
const b = { key: "b" }
const c = { key: "c" }

a[b] = 123
a[c] = 456

console.log(a[b])
```
- A: 123
- B: 456
- C: undefined
- D: ReferenceError
## 答案: B

对象键自动转换为字符串.我们试图将一个对象设置为对象a的键,其值为123

但是,当对象自动转换为字符串化时,它变成了[object object].所以我们在这里说的是a['object object'] = 123.然后我们可以尝试再次做同样的事情.c对象同样会发生隐式类型转换.那么,a['object object'] = 456

然后,我们打印a[b],实际上是a['object object'].我们将其设置为456,因此返回456

---

## **30. 下面代码输出什么**
```
const foo = () => console.log('First')
const bar = () => setTimeout(() => console.log('Second'))
const baz = () => console.log('Third')

bar()
foo()
baz()
```
- A: First Second Third
- B: First Third Second
- C: Second First Three
- D: Second Three First
## 答案: B

我们有一个`setTimeout`函数并首先调用它.然而最后打印了它.

这是因为在浏览器中,我们不只有运行时引擎,还有一个叫做`webAPI`的东西.`webAPI`为我们提供了`setTimeout`函数,例如`DOM`.

将`callback`推送到`webAPI`后,`setTimeout`函数本身(但不是回调!)人堆栈中弹出.

---

## **31. 单击按钮时event.target是什么**
```
<div onclick="console.log('first div')">
  <div onclick="console.log('second div')">
    <button onclick="console.log(button)">
      click!
    </button>
  </div>
</div>
```
- A: div外部
- B: div内部
- C: button
- D: 所有嵌套元素的数组
## 答案: C

导致事件的最深嵌套元素是事件在目标.你可以通过`event.stopPropagation`停止冒泡

---

## **32. 单击下面的html片段打印的内容是什么**
```
<div onclick="console.log('div')">
  <p onclick="console.log('p')">
    click here
  </p>
</div>
```
- A: p div
- B: div p
- C: p
- D: div
## 答案: A

如果我们单击p,我们会看到两个日志:p和div.在事件传播期间,有三个阶段:捕获,目标和冒泡.默认情况下,事件处理程序在冒泡阶段执行(除非鼗`useCapture`设置为true).它从最深的嵌套元素向外延伸.

---

## **33. 下面代码输出什么**
```
const person = {name:'Lydia'}

function sayHi(age){
  console.log(`${this.name} is ${age}`)
}

sayHi.call(person,21)
sayHi.bind(person,21)
```
- A: undefined is 21  Lydia is 21
- B: function function
- C: Lydia is 21 Lydia is 21
- D: Lydia is 21 function
## 答案: D

使用两者,我们可以传递我们想要this关键字引用的对象,但是`.call`方法会立即执行

`.bind`方法会返回函数的拷贝值.但带有绑定的上下文!它不会立即执行.

---

## **34. 下面代码输出什么**
```
function sayHi(){
  return (() => 0)()
}
type of sayHi()
```
- A: object
- B: number
- C: function
- D: undefined
## 答案:B

`sayHi`函数返回立即调用的函数的返回值,该函数返回0,类型为number

>只有7种内置类型: null , undefined , boolean , number , string , object 和 symbol. function 不是一个类型,因为函数是对象,它的类型是object

---

## **35. 下面这些值哪些是假值**
```
0
new Number(0)
('')
(' ')
new Boolean(false)
undefined
```
- A: 0 , '' , undefined
- B: 0 , new Number(0) , '' , new Boolean(false) , undefined
- C: 0 , '' , new Boolean(false) , undefined
- D: 所有都是假值
## 答案: A

JavaScript中只有6个假值
  - undefined
  - null
  - NaN
  - 0
  - ''(空字符串)
  - false

函数构造函数,如`new Number`和`new Boolean`都是真值

---

## **36. 下面代码的输出是什么**
```
console.log(typeof typeof 1)
```
- A: number
- B: string
- C: object
- D: undefined
## 答案: B

typeof 1返回 number  
typeof 'number' 返回 string

---

## **37. 下面代码输出什么**
```
const numbers = [1,2,3]
numbers[10] = 11
console.log(numbers)
```
- A: [1, 2, 3, 7 x null, 11]
- B: [1,2,3,11]
- C: [1,2,3,7 x empty, 11]
- D: SyntaxError
## 答案: C

当你为数组中的元素设置一个超过数组长度的值时,JavaScript会创建一个名为空插槽的东西,这些东西的值实际上是undefined,但你会看到类似的东西:
[1,2,3,7 x empty, 11]
这取决于你运行它的位置(每个浏览器有可能不同)

---

## **38. 下面代码输出什么**
```
(() => {
  let x,y
  try{
    throw new Error()
  } catch(x) {
    (x=1),(y=2)
    console.log(x)
  }
  console.log(x)
  console.log(y)
})()
```
- A: 1 undefined 2
- B: undefined undefined undefined
- C: 1 1 2
- D: 1 undefined undefined
## 答案: A

catch块接收参数x,当我们传递参数时,这与变量的x不同.这个变量x是属于catch作用域

之后,我们将这个块级作用域的变量设置为1,并没有设置y的变量,现在我们打印块级作用域的变量x,它等于1

在catch之外,x仍然是undefined,而y是2.当我们想在catch块之外的console.log(x)时,它会返回undefined,而y返回2

---

## **39. JavaScript中的所有内容都是()**
- A: 原始类型或对象
- B: 函数或对象
- C: 技巧问题!只有对象
- D: 数字或对象
## 答案: A

JavaScript只有原始类型和对象

原始类型是boolean , null , undefinded , object , number , string和symbol

---

## **40. 下面代码输出什么**
```
[[0,1],[2,3]].reduce(
  (acc,cur) => {
    return acc.contact(cur)
  },
  [1,2]
)
```
- A: [0,1,2,3,1,2]
- B: [6,1,2]
- C: [1,2,0,1,2,3]
- D: [1,2,6]
## 答案: C

[1,2]是我们的初始值.这是我们开始执行reduce函数的初始值,以及第一个acc的值,在第一轮中acc是[1,2],cur是[0,1].我们将它们连接起来,结果是[1,2,0,1]

然后,acc的值为[1,2,0,1],cur的值为[2,3].我们将它们连接起来,得到[1,2,0,1,2,3].

---

## **41. 下面代码输出什么**
```
!!null
!!''
!!1
```
- A: false false false
- B: false false true
- C: false true true
- D: true true false
## 答案: B

---
## **42. setInterval方法返回值是什么**
```
setInterval(() => console.log('Hi'),1000)
```
- A: 一个唯一的id
- B: 指定的毫秒数
- C: 传递的函数
- D: undefined
## 答案: A

它返回一个唯一的id,此id可用于使用clearInterval()函数清除该定时器

---

## **43. 下面代码返回什么**
```
[...'Lee']
```
- A: ['L','e','e']
- B: ['Lee']
- C: [[],'Lee']
- D; [['L','e','e']]
## 答案: A

字符串是可迭代的.扩展运算符将迭代的每个字符映射到一个元素

