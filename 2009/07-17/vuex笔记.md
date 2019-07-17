## vuex

vuex是一个专门为vue.js应用程序开发的状态管理模式.  
这个状态我们可以理解为在data中的属性,需要共享给其他组件使用的部分.  
也就是说,是我们需要共享的data,使用vuex进行统一集中式管理.  
vuex周期的大概流程是,在vue实例中调用store下的state后,想要改变state,要先通过store.commit(有异步要先action)来改变state,从而在vue页面中渲染.

![vuex图解](https://segmentfault.com/img/remote/1460000016145588?w=701&h=551)

### **state**

store里的数据源库(一个全局的data),可在全局调用$store.state访问.  
每当state变化的时候,都会重新求取计算属性,并且触发更新相关的DOM

在state里

```
state:{
  name:'张三'
}
```

vue实例

```
<div>用户名: {{$store.state.name}} </div>
```

---

### **Getter**

从store中的state中派生出一些状态.(可以认为是对数据获取之前再次编译store的计算属性).他并不会改变state本身,只有当它的依赖值发生变化时才会被重新计算.(类似vue实例的computed)  
Getter接受state作为其第一个参数,可以访问store里的state

```
getters:{
  doneTodos: state => {
    return state.todos.filter(todo => todo.done)
  }
}
```

第二个参数是getters,可以访问到上下文的getters

```
getters: {
  doneTodooCount: (state, getters) => {
    return getters.doneTodos.length
  }
}
```

假如需要传参,可以通过方法访问

```
getters: {
  getTodoById: state => id => {
    return state.todos.find(todo => todo.id === id)
  }
}
```

在vue补修中访问getters

```
computed: {
  doneTodosCount() {
    return $store.getters.doneTodosCount
  }
}
```

---

### **mutation**

mutation是更改vuex的store中的状态的唯一方法.需要用`store.commit()`触发.  
mutation第一个参数是state,第二个参数是载荷(Payload).即传参在大多数情况下,载荷言是一个对象.  
非常重要的一点,mutation内必须是同步函数.

```
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}
```

执行时`store.commit()`,第一个参数是mutations的名称(字符串),第二个是载荷对象.

```
store.commit('increment', {
  amount: 10
})
```

---

### **action**

action是专门处理store异步操作的  
action提交的是mutation,而不是直接改变状态
action通过`store.dispatch()`触发
action第一个参数是context(上下文),第二个参数是载荷

```
actions: {
  incrementAsync({commit}){
    setTimeout(() => {
      commit('increment')
    },1000)
  }
}
```

vue实例里执行action
```
store.dispatch('incrementAsync', {
  amount: 10
})
```

---

### **module**

由于使用单一状态树,应用的所有状态会集中到一个比较大的对象,当应用变得非常复杂时,store对象就有可能变得相当臃肿  
module可以把store侵害成模块.每个模块拥有自己的store,mutation,action,getter

store.js
```
const moduleA = {
  state: {...},
  mutations: {...},
  actions: {...},
  getters: {...}
}

const moduleB = {
  state: {...},
  mutations: {...},
  actions: {...}
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})
```

在vue实例里访问时

```
$store.state.a  // moduleA的状态
$store.state.b  // moduleB的状态
```

这种写法下state会区分模块,但mutation和action调用时,写法是一样的

```
$store.commit('set')
$store.dispath('set')
```

---

### **命名空间**

假如不同的模块里有出现相同名称的mutations,就会出现问题,为了避免,可以使用命名空间.  
在每个模块下添加`namespaced:true`,即可实现命名空间.

```
const moduleA = {
  namescaped: true,
  state: {...},
  mutations: {...},
  actions: {...},
  getters: {...}
}

const moduleB = {
  namespaced: true,
  state: {...},
  mutations: {...},
  actions: {...}
}

const store = new Vuex.Stroe({
  modules: {
    a: moduleA,
    b: moduleB
  }
})
```

在vue实例中调用时改为

```
$store.commit('a/set')
$store.dispath('b/set')
```

---

### **vuex和data**

vuex是全局存在的.对于一些只在一个页面组件中用到的信息,可以只存在对于的页面组件的data中.  
store中一般存放一些多处会读取或者写的信息,这样可以避免多层父子传值.一般常见的存放在store里的有用户信息,标签信息等.这样可以在不同的地方操作标签.

例子: 在一个项目中有一个右键弹出菜单的组件,组件需要传参来决定传入内容

store.js文件
```
{
  state: {
    rightMenuData:[
      {
        name: '刷新此窗口',
        key:'refresh'
      },
      {
        name: '关闭此窗口',
        key: 'close'
      },
      {
        name: '关闭全部窗口',
        key: 'closeAll'
      }
    ]
  }
}
```

vue页面中

```
<righ-menu
  :menuData="rightMenuData"
  :eventHandler="rightClickHandler"
  :handleSelect="handleRightSelect"
>
</right-menu>
```

例子中的菜单内容存在store里,在一个页面里调用了对应的数据.可是实际上项目只要一个页面需要用到这个数据.这种情况可以把数据只放到页面data里,避免store过于臃肿

另一个例子

vue页面中

```
mounted(){
  let user = global.myLocalStorage.getItem('user')
  if(user){
    user = JSON.parse(user)
    this.sysUserName = user.name || ''
    this.sysUserAvatar = user.avatar || 'agree.bee.png'
  }
}
```

这里页面中用到一个用户信息,由于本地保存的关系,用户数据被存到localstorage里.然后项目在各自拿用户信息的时候会直接从localstorage里拿.这种情况下可以考虑在页面打开时,只从localstorage拿一次数据,存在要store里.在项目运行的过程中,就可以直接在store里访问用户信息.

---

### **多个store运用**

在一些大型应用中,有可能存在多全store的情况,来保持自身的store不受影响.

例子:A实例下面有多个vue子页面b,c,d.它们都需要有自己的store,则在内部vue实例中new一个vuex.store,并存起来(不一定要存$store,可以是其他变量)

```
beforeCreate() {
  this.$store = new Vuex.Store({
    state,
    mutations,
    actions
  })
}
```

此时vue实例b,c,d中
```
this.$store
```
返回的是自己的store

如果想要访问A页面原来的store,可以用$root访问概况vue实例由于他的this.$store并没有被重写,所以指向的还是原来全局的store
```
this.$root.$store
```