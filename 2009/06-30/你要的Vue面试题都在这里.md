## **vue生命周期**

总共分为8个阶段,创建前/后,载入前/后,更新前/后,销毁前/后

- 创建前/后: 在`beforeCreate`阶段,vue实例的挂载元素`el`还没有.
- 载入前/后: 在`beforeMount`阶段,vue实例的`$el和`data`都初始化了,但还是挂载之前为虚拟DOM节点,`data.message`还未替换.在`mounted`阶段,vue实例挂载完成,`data.message`成功渲染.
- 更新前/后: 当`data`变化时,会触发`beforeUpdate`和`updated`方法.
- 销毁前/后: 在执行`destory`方法后,对`data`的改变不会再触发周期函数,说明此时vue实例已经解除了事件监听以及和DOM绑定,但DOM结构依然存在.

---

## **为什么vue组件中的data必须是一个函数**

对象为引用类型,当重用组件时,由于数据对象都指向同一个data对象,当在一个组件中修改data时,其他征用的组件中的data会同时被修改;而使用返回对象的函数,由于每次返回的都是一个新对象,引用地址不同,则不会出现这个问题.

---

## **`active-class是哪个组件的属性`**

`vue-router`模块的`router-link`组件

---

## **vue-rputer有哪几种导航钩子**

一共有三种,一种是全局导航钩子:`router.beforeEach(to,from,next)`,作用是中转前进行判断拦截.第二种是组件内的钩子,第三种是单独路由独享组件.

---

## **vue-loader是什么,使用它的用途有哪些**

解析vue文件的一个加载器,跟template/js/style转换成js模块

---

## **请说出vue/cli项目中src目标每个文件夹和文件的用法**

- assets: 文件夹是放静态资源
- components: 放组件
- router: 是定义路由相关的配置
- view: 视图文件
- app.vue: 是一个应用主文件
- main.js: 入口文件

---

## **计算属性和watch**

- computed: 计算属性是用来声明式的描述一个值依赖了其它的值.当你在模板里把数据绑定到一个计算属性上时,vue会在其依赖的任何值导致该计算属性改变时更新DOM.这个功能非常强大,它可以让你的代码更加声明式,数据驱动并且易于维护.

- watch: 监听的是你定义的变量,当你定义的变量的值发生变化时,调用对应的方法.就好在div写一个表达式name,data里定稿rum和lastname,firstname,在watch里当rum的值发生变化时,就会调用rum方法,方法里面的形参对应的是num的新值和旧值,而计算属性computed计算的是name依赖的值,它不能计算在data中已经定义过的变量.

---

## **prop验证,和默认值**

在父组件给子组件传值的时候,为了避免不必要的错误,可以给prop的值进行类型设定,让父组件给子组件传值的时候,更加准确,prop可以传一个数字,一个布尔值,一个数组,一个对象,以及一个对象的所有属性.组件可以为props指定验证要求.如果未指定验证要求,vue会发出警告.

```
props:{
  visible:{
    type:Boolean,
    default:true,
    required:true
  }
}
```

---

## **vue组件通信**

1. 父传递子  
父:自定义属性名+数据(要传递) => :value="数据"  
子:props['父组件上的自定义属性名'] => 进行数据接收

2. 子传父  
在父组件中注册子组件并在子组件标签上自定义事件的监听  
子: this.$emit('自定义事件名称',数据) 子组件标签上绑定@自定义事件名称='回调函数'  
父: methods:{自定义事件(){处理逻辑}}

3. 兄弟组件  
通过中央通信 `let bus - new Vue()`  
A: methods:{函数{bus.$emit('自定义事件名',数据)}}发送  
B: created(){bus.$on('A发送过来的自定义事件名',函数)}
进行数据接收

---

## **vue路由传参数**

1. 使用query方法传入的参数使用`this.$route.query`接受
2. 使用params方式传入的参数使用`this.$route.params`接受

---

## **vuex是什么,有哪几种属性**

vuex是专门为vue应用程序开发的状态管理模式  
有5种,分别是state,getter,mutation,action,module

1. vuex的store是什么  
vuex就是一个仓库,仓库里放了很多对象.其中state就是数据源存在地,对应一般vue对象里面的datastate里面存在放的响应式的,vue组件从store读取数据,若是store中的数据发生改变,依赖这组数据的组件也会发生更新邮寄费通过mapState把全局和state和getters映射到当前组件的computed计算属性.

2. vuex的getter是什么  
getter可以对state进行计算操作,它就是store的计算属性,虽然在组件内也可以做计算属性,但getters可以在多组件之间复用,如果一个状态只在一个组件内使用,是不可以用使用getters的

3. vuex中的mutation是什么  
更改vuex的store状态的唯一方法是提交mutation

4. vuex的action是什么  
action类似mutation,不同在于action提交的是mutation,而不是直接更改状态. action可以包含任意异步操作.

5. vuex中的module是什么  
面对复杂的应用程序,当管理的状态比较多时,我们需要将vuex和store对象侵害成模块(modules)

6. vue中的ajax请求代码应该写在组件的methods中还是vuex的action 中  
如果请求来的数据不是要被其他组件公用,仅仅在请求的组件内使用,就不需要放入vuex的state里,如果被其他地方利用,请将请求放入action里,方便复用,并包装成promise返回.

---

##**v-show和v-if指令的共同点和不同点**

v-show是通过修改元素的css属性display来让元素显示或隐藏  
v-if是趎销毁和重建DOM达到让元素显示和隐藏的效果

---

## **如何让css只在当前组件中起作用**
奖当前组件中style标签添加scoped属性

---

## **`<keep-alive></keep-alive>`的作用是什么**

`<keep-alive></keep-alive>`包裹动态组件时,会缓存不活动的组件实例,主要用于保留组件状态或避免重新渲染.

以上说成人话就是:比如有一个列表和一个详情,那么用户就会经常执行打开详情->返回列表->打开详情...这样的话列表和详情都是一个频率很高的页面,那么就可以对列表组件使用`<keep-alive></keep-alive>`进行缓存,这样用户每次返回列表时,都能从缓存中快速渲染,而不是重新渲染.

---

## **delete和Vue.delete删除数组的区别**

delete只是被删除的元素变成了empty/undefined,其他的元素的键值还是不变.

Vue.delete直接删除了数组,改变了数组的键值.

```
var a = [1,2,3,4]
var b = [1,2,3,4]

delete.a[0]
console.log(a)  // [empty,2,3,4]

this.$delete(b,0)
console.log(b)  // [2,3,4]
```

---

## **$nextTick是什么**

Vue实现响应式并不是数据发生变化后dom立即变化,而是按照一定的策略来进行dom更新.

$nextTick是在下次dom更新循环结束之后延迟回调,在修改数据之后使用$nextTick,则可以在回调中获取更新后的dom

---

## **v-on监听多个方法**

```
<input type="text" :value:'name' @input="onInput" @focus="onFocus" @blur="onBlur" />
```

---

## **v-on常用修饰符**

- .stop: 将阻止向上冒泡.同埋于调用`event.stopPropagation()`方法
- .prevent: 会阻止当前事件的默认行为.同理于调用`event.preventDefault()`方法
- .self: 当事件是从事件绑定的元素本身触发时才触发回调
- .once: 表示绑定的事件只会被触发一次

---

## **`v-for` key的作用**

当vue用v-for正在更新已渲染过的元素列表时,它默认用"就地利用"策略.如果数据项的顺序被改变,vue将不是移动dom元素来匹配数据项的改变,而是简单利用此处每个元素,并且确保它在特定索引下显示已被渲染的每个元素.为了给vue一个提示,以便它能跟踪每个节点的身份,从而重用和重新排序现有元素,你需要为每项提供一个唯一key属性.key属性的类型只能为string或者number类型.

key的特殊属性产要用在vue的虚拟dom算法,在新旧nodes对比时辨识VNodes,如果不使用key,,vue会使用一种最大限度减少动态元素并且心可能尝试修复/再利用相同类型元素的算法.使用key,它会基于key的变化重新排列元素的顺序,并且会移除key不存在的元素.

---

## **v-for与v-if的优先级**

v-for比v-if优先,如果每一次都需要遍历整个数组,将会影响速度,尤其是当仅需要渲染很小一部分的时候

----

## **vue子组件调用父组件的方法**

- 直接在子组件是通过`this.$parent.event`来调用父组件的方法
- 在子组件里用$emit向父组件触发一个事件,父组件监听这个事件就行

---

## **Promise对象是什么**

1. Promise是异步编程的一种解决方案,它是一个容器,里面保存着某个未来才会结束的事件(通常是一个异步操作)的结果.从语法上说,Promise是一个以无明,从它可以获取异步操作的消息.Promise提供絟的API,各种异步操作都可以用同样的方法进行处理.Promise对象是一个构造函数,用来生成Promise实例.
2. Promise的两个特点: 对象状态不受外界影响;一旦状态改变就不会再变,任何时候都可以得到结果.

---

## **vue中的ref是什么**

ref被用来给元素或子组件注册引用信息.引用信息将会注册在父组件的$ref对象上.如果普通在dom元素上使用,引用指向的就是dom元素,如果用在子组件上,引用就指向组件实例.

---

## **$route和$router的区别**

$route是路由信息对象,包括path , params , hash , query , fullPath , matched , name等路由信息

$router是路由实例对象包括了路由的跳转方法,钩子函数等.

---

## **如何优化SPA应用的首屏加载速度慢的问题**

1. 将公用的JS库通过script标签外部引入,减少app.bundel的大小,让浏览器并行下载资源文件,提高下载速度.
2. 在配置路由时,页面和组件使用懒加载的方式引入,进一步缩小app.bundel的体积,在调用某个组件时再加载对应的js文件
3. 加一个首娘loading图,提升用户体验.

---

## **vue改变数组触发视图更新**

以下方法调用会改变原始数组: push(),pop(),shift(),unshift(),splice(),sort(),reverse(),Vue.set(target,key,value)

调用Vue.set(target,key,value)
- target: 要更改的数据源(可以是对象或者数组)
- key: 要更改的具体数据
- value: 重新赋的值

---

**DOM渲染在哪个周期中已经完成**

在mounted阶段渲染完成

> 注意: mounted不会承诺所有的子组件也都一起被挂载,如果你希望等到整个视图渲染完毕,可以用`vm.$nextTick`替换掉mounted

```
mounted:() => {
  this.$nextTick(() => {
    // ...
  })
}
```

---

## **简述每个周期具体适全哪些场景**

- beforeCreate: 可以在这加个loading事件,在加载实例时触发
- created: 初始化完成时的事件写在这里,如在这结束loading事件,异步请求也适宜在这里调用
- mounted: 挂载元素,获取到dom节点
- updated: 如果对数据统一处理,在这里写上相应函数
- beforeDestroy: 可以做一个确认停止事件的确认框

---

## **第一次加载会触发哪几个钩子**

会触发beforeCreate,created,beforeMount,mounted

## **动态绑定class**

```
<div :class="{acrive:isActive}"></div>
``` 