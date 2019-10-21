### **require.context()**

1. 场景: 如页面需要导入多上组件,原始写法:   
```
import titleCom from '@/components/home/titleCom'
import bannerCom from '@/components/home/bannerCom'
import cellCom from '@/components/home/cellCom'

export default {
  components: {titleCom, bannerCom, cellCom}
}
```

2. 这样就写了大量重复的代码,利用 reuire.context 可以写成  
```
const path = require('path')
const files = require.context('@/components/home', false, /\.vue$/)
cpmst modules = {}
files.keys().forEach(key => {
  const name = path.baseName(key, '.vue')
  modules[name] = files(key).default || files(key)
})

export default {
  components: modules
}
```  
这样不管页面引入多少组件,都可以使用这人方法.

3. API 方法  
实际上是 webpack 的方法, vue 工程一般基于 webpack, 所以可以使用 require.context(directory, useSubdirectories, regExp)  
接收三个参数  
    - directory: 说明需要检索的目录
    - useSubdirectories: 是否检索子目录
    - regExp: 匹配文件的正则表达式, 一般是文件名

---

### **watch**

1. 常用方法  
场景:表格初始进来需要调用查询接口 getList(), 然后 input 改变会重新查询  
```
created() {
  this.getList()
},
watch: {
  inputVal() {
    this.getList()
  }
}
```

2. 立即执行  
可以直接利用 watch 的 immediate 和 handler 属性简写  
```
watch: {
  inpVal: {
    handler: 'getList',
    immediate: true
  }
}
```

3. 深度监听  
watch 在 deep 属性, 深度监听, 也就是监听复杂数据类型  
```
watch: {
  inpValObj: {
    handler(newVal, oldVal) {
      console.log(newVal)
      console.log(oldVal)
    },
    deep: true
  }
}
```  
此时发现 oldVal 和 newVal 值一样;  
因为它们索引同一个对象/数组, Vue不会保留修改之前值的副本;  
所以深度监听虽然可以监听到对象的变化,但是无法监听到具体对象里面哪个属性的变化.

---

### **组件通信**

1. props  
这个属性非常常用, 就是父传子的属性;  
props 值可以是一个数组或对象;  
```
// 数组: 不建议使用
props: []

// 对象
props: {
  inpVal: {
    type: Number, //传入值限定类型
    // type 值可为 String, Number, Boolean, Array, Object, Date, Functions, Symbol
    // type 还可以是一个自定义的构造函数, 并且通过 instanceof 来进行检查确认
    required: true, // 是否必传
    defalt: 200,  // 默认值, 对象或数组默认值必须从一个工厂函数获取 default: () => []
    validator: (value) {
      // 这个值避孕套匹配下列字符串中的一个
      return ['success', 'warning', 'danger'].indexOf(value) !== -1
    }
  }
}
```

3. $emit  
这个也是非常常用的,子组件触发父组件给自己绑定的事件, 也就是子传父的方法  
```
// 父组件
<home @title="title">

// 子组件
this.$emit('title', [{title: '这是title'}])
```

3. vuex  
这个也是非常常用的, vuex 是一个状态管理器;  
是一个独立的插件, 适合数据共享多的项目里面, 因为如果只是简单的通讯,使用起来比较重  

    - state: 定义存贮数据的仓库, 可通过 `this.$store.state` 或 `mapState` 访问
    - getter: 获取 store 值, 可认为是 store 的计算属性, 可通过 `thhis.$store.getter` 或 '`mapgetters` 访问
    - mutation: 同步改变 store 值, 为什么会设计成同步? 因为 mutation 是直接改变 store 值, vue 对操作进行了记录, 如果是异步无法追踪改变,可 通过 `mapMutations` 调用.
    - 异步调用函数执行 mutation, 进而改变 store 值, 可通过 `this.$dispatch` 或 `mapActions` 访问.
    - modules: 模块, 如果状态过多, 可以拆分成模块, 最后在入口通过 `...` 解构引入.

4. $attrs 和 $listeners  
2.4.0新增  
这两个不是常用属性,但是高级用法很常见  
**$attrs**  
场景: 如果父传子有很多值, 那么在子组件需要定义多个 props  
解决: $attrs 获取子传父中未在 props 定义的值  
```
// 父组件
<home title="这是标题" width="80" height="80" imgUrl="imgUrl">

// 子组件
mounted() {
  console.log(this.$attrs)
  // {title: '这是标题', width: "80", height: "80", imgUrl: "imgUrl"}
}
```  
相对就的,如果子组件定义了 props, 打印的值就是剔除定义的属性  
```
props: {
  width: {
    type: String,
    default: ''
  }
},
mounted() {
  console.log(this.$attrs)
  // {title: "这是标题", height: "80", imgUrl: "imgUrl"}
}
```

**$listeners**  
场景: 子组件需要调用父组件的方法  
解决: 父组件的方法可以通过 `v-on="$listeners"` 传入内部组件, 在创建更高层次的组件时非常有用  
```
// 父组件
<home @change="change">
// 子组件
mounted() {
  console.log(this.$listeners)
  // 即可拿到 change 事件
}
```  
如果是孙组件要访问父组件的属性和调用方法, 直接一级一级传下去就可以.  

**inheritAttrs**  
```
// 父组件
<home title="这是标题" width="80" height="80" imgUrl="imgUrl">
// 子组件
mounted() {
  console.log(this.$attrs)
}
```  
inheriAttrs 默认值为 true, true 的意思是将父组件中除了 props 外的属性添加到子组件的根节点上 (说明, 即使设置为 true, 子组件仍然可以通过 $attr 获取到 props 意外的属性), 将 inheritAttrs: false 后, 属性就不会显示在根节点上了

**provide 和 inject**  
2.2.0新增  
描述:  
provide 和 inject 主要是高阶插件/组件库提供用例. 并不推荐直接在用于应用程序代码中;  
并且这对选项需要一起使用;  
允许一个祖先组件向其所有子孙后代注入一个依赖, 不论组件层次有多深, 并在起上下游关系成立的时间里始终有效.  

```
// 父组件
provide: {  // provide 是一个对象, 提供一个属性或方法
  foo: '这是foo',
  fooMethod: () => {
    console.log('父组件 fooMethod 被调用')
  }
},
// 子或孙组件
inject: ['foo', 'fooMethod'], // 数组或对象,注入到组件
mounted() {
  this.fooMethod()
  console.log(this.foo)
}
// 在父组件下面所有的子组件都可以利用 inject
```  

provide 和 inject 绑定并不是可响应的. 这是官方刻意而为.  
然而,如果传入了一个可监听对象, 那么其对象的属性还是可以可响应的, 对象是因为是引用类型  

```
// 父组件
provide: {
  foo: '这是foo'
},
mounted() {
  this.foo = '这是新的 foo'
}

// 子或孙组件
inject: ['foo'],
mounted() {
  console.log(this.foo) // 子组件打印的还是 '这是 foo'
}
```

**$parent 和 $children**  
$parent: 父实例  
$children: 子实例  

```
// 父组件
mounted() {
  console.log(this.$parent) // 可以拿到 parent 的属性和方法
}
```  
$children 和 $parent 并不保证顺序, 也不是响应式的  
只能拿到一级父组件和子组件

**$refs**  
```
// 父组件
<home ref="home">

mounted() {
  console.log(this.$refs.home)
  // 即可拿到子组件的实例, 就可以直接操作 data 和 methods
}
```

**$root**   
```
// 父组件
methods() {
  console.log(this.$root) 
  // 获取根实例, 最后所有组件都是挂载到根实例上
  console.log(this.$root.$children[0])
  // 获取根实例的一级子组件
  console.log(this.$root.$children[0].$children[0])
  // 获取根实例的二级子组件
}
```

**sync**  
在 vue@1.x 的时候曾作为双向绑定功能存在, 即子组件可以修改父组件中的值  
在 vue@2.0 由于违背单项数据流的设计被干掉了  
在 vue@2.3.0+ 以上版本又重新引入这个 .sync 修饰符  

```
// 父组件
<home :title.sync="title" />
// 编译时会被扩展为
<home :title="title" @update:title="val => title = val" />

// 子组件
// 子组件可以通过 $emit 触发 update 方法改变
mounted() {
  this.$emit('update: title', '这是新的title')
}
```

**v-slot**  
2.6.0新增  
1. slot, slot-scope, scope 在2.6.0 中都被废弃,但未被移除
2. 作用就是将父组件的template传入子组件

匿名插件(也叫默认值插槽): 没有命名, 有且只有一个  

```
// 父组件
<todo-list>
  <template>
    任意内容
    <p>这是匿名插槽</p>
  </template>
</todo-list>

// 子组件
<slot>默认值</slot>
// v-slot: default 写法上感觉和具名写法比较统一, 容易理解, 也可以不用写
```  

具名插槽: 相对匿名插槽组件 slot 标签带 name 命名的  

```
// 父组件
<todo-list>
  <template v-slot:todo>
    任意内容
    <p>匿名插槽</p>
  </template>
</todo-list>

// 子组件
<slot name="todo">默认值</slot>
```  
作用域插槽: 子组件内数据可以被父页面拿到(解决了数据只能从父页面传递给子组件)  

```
// 父组件
<todo-list>
  <template v-slot:todo="slotPops">
    {{slotProps.user.firstName}}
  </template>
</todo-list>
// slotProps 可以随意命名
// slotProps 接取的是子组件标签 slot 上属性数据的集合所有 v-bind:user="user"

// 子组件
<slot name="todo" :user="user" :test="test">
  {{user.lastName}}
</slot>

data() {
  return {
    user: {
      firstName: 'Zhang',
      lastName: 'san'
    },
    test: [1,2,3,4]
  }
}
// {{user.lastName}} 是默认数据, v-slot: todo 当父页面没有(="slotProps")
```