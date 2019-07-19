# [原文地址](https://alligator.io/js/deep-cloning-javascript-objects/)

- 原作者: [Chris Chu](https://alligator.io/author/chris-chu)
- 翻译: gaoshengyun

> 如果你打算用JavaScript进行编程,那么你必须了解对象是如何工作的. 对象是JavaScript中最重要的元素之一,深入地理解对象有且于你更好的用JavaScript编程.尤其是在对象克隆的时候,它并不像上看去那么简单.

如果你不想改变原来的对象,那你必须克隆一个对象.

我们创建一个JavaScript对象

```
let testObject = {
  a: 1,
  b: 2,
  c: 3
}
```
在上面的代码片段中,我们初始化了一个新的变量,并赋值给了一个变量`testObject`. 现在对大多数初学者来说,他们会试着将`testObject`赋值给一个新变量来创建这个对象的副本.但是很抱歉,这个方法并不起作用.

下面的代码说明了为什么不起作用

```
let testObject - {
  a: 1,
  b: 2,
  c: 3
}

// 将testObject赋值给另一个新变量
let testObjectCopy = testObject

testObject.a = 9
console.log(testObjectCopy.a)
// 9
```

如上代码所示,创建一个新变量`testObjectCopy`并没有创建`testObject`的副本.取而代之的是他只是简单的引用了`testObject`.你在所谓的副本中做的所有的发动都会影响到原来的对象.

循环遍历对象并将每个属性复制到新对象也不起作用

```
const copyObject = object => {
  // 这是存储原始对象属性的对象
  letcoiedObj = {}

  for(let key in object) {
    // 这里将每个属性从原始对象复制到复制对象
    copyiedObj[key] = object[key]
  }

  return copiedObj
}

const testObject = {
  a: 5,
  b: 5,
  c: {
    d: 4
  }
}

copyObject(testObject)
```

上述方法有以下几个问题

1. 将每个属性复制到新对象的循环只会复制对象上的可枚举属性.可枚举属性是将要出现在for循环和`Object.keys`中的属性.
2. 复制的对象有一个新的`Object.prototype`方法,这不是复制对象时所需的方法.
3. 如果对象具有用为对象的属性,则复制的对象实际上将会引用原始对象而不是创建副本.这意味着如果更改复制对象中的嵌套对象,原始对象也会更改.
4. 不复制任何属性描述.如果将config或writable设置不false,则复制对象中的属性描述将会默认为true.

---

## **那么应该怎么样正确复制对象?**

对于仅存储基本类型(如数字和字符串)的简单对象,上述浅层复制方法将想作用.但是如果对象具有对其他嵌套的对象的引用,则不会复制实际对象.你只会复制对其的引用.

对于深层复制,最简单的选择是使用可靠的外部库,如[Lodash](https://lodash.com/)

---

## **使用Lodash和clone和clonedeep**

Lodash提供两种不同的功能,允许你进行浅拷贝和深拷贝,它们是clone和clonedeep.Lodash的优点在于你可以单独导入它的每个函数,而无需将整个库放入你的项目中,这可以大大的减少依赖的大小.

```
const clone = require('lodash/clone')
const clonedeep = require('lodash/clonedeep')
```
你也可以这样做
```
const clone = require('lodash.clone')
const clonedeep = require('lodash.clonedeep')
```
以上两种方法都可以,取决于你的风格

现在就用clone和clonedeep函数做一些尝试

```
const clone = require('lodash/clone')
const clonedeep = require('lodash/clonedeep')

const externalObject = {
  animal:'Gator'
}

const originalObject = {
  a: 1,
  b: 'string',
  c: false,
  d: externalObject
}

const shallowClonedObject = clone(originalObject)

externalObject.animal = 'Crocodile'

console.log(originalObject)
console.log(shallowClonedObject)
// originalObject 和 shallowClonedObject 中的animal是同时被改变的,因为它是一个浅拷贝的副本

const deepClonedObject - clonedeep(originalObject)

externalObject.animal = 'Lizard'

console.log(originalObject)
console.log(deepClonedObject)
// 原始对象中的animal属性发生了变化,但对于deepClonedObject,它复制后仍然是Crocodile,对象是独立的而不是复制引用的
```

上面代码中,我们创建了一个名为`originalObject`的对象,它存储了7个属性,每个属性都有不同的值,属性`d`引用我们的`externalObject`,它具有值为`Gator`的`animal`的属性.

当从Lodash执行`clone`函数时,它会创建一个对象的浅层副本,我们将其分配给`shallowClonedObject`.在`externalObject`中为`animal`属性赋值一个新值将改变`originalObject`和`shallowClonedObject`,因为浅拷贝只能将引用复制到`externalObject`并它没有为自己创造一个全新的对象.

这就是clonedeep函数的用武之地,如果你对`deepClonedObject`执行相同的处理,那么`originalObject`的`d`属性是唯一要改变的属性.