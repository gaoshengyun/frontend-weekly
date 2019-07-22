作者：liyao0630  
链接：https://juejin.im/post/5d19e84ef265da1b8a4f3583  
来源：掘金  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

vuex是专为vue.js应用程序开发的状态管理模式.它采用集中式存储管理应用的所有组件的状态,并以相应的规则保证状态以一种可预测的方式发生变化.

## **vuex流程图**

![](https://user-gold-cdn.xitu.io/2019/7/2/16bb35f923255e26?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

1. 组件触发事件dispatch
2. action调用后端接口
3. 调用commit来触发mutations
4. 改变state的状态
5. 最后渲染组件

---

## **使用vuex**

安装模块
```
npm i vuex -S
```

```
// store.js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

let store = new Vuex({
  state: {
    count: 1000
  },
  getters: {
    // 可传递 getter 作为第二个参数
    filter(state) {
      return state.count >= 120 ? 120 : state.count
    }
  },
  mutations: {
    add(state, n, m) {
      state.count = state.count + n + m
    }
    de(state, payload) {
      state.count -= payload.n
    }
  },
  actions: {
    add(context) {
      setTimeout( () => {
        context.commit('add', {n, 5})
      },1000)
    }
  }
})
export default store
```

### **读取state的属性**

```
this.$store.state.count
```

### **读取getters的属性**

```
this.$store.getters.filter
```

### **commit同步改变state**

```
// 多个参数
this.$store.commit('add',5,4)

// 对象参数
this.$store.commit('de',{
  n: 5
})

// 直接使用包含 type 属性的对象
this.$store.commit({
  type: 'de',
  de: 5
})
```

### **dispatch异步改变state**

```
this.$store.dispatch('add')
```

---

## **vuex辅助函数**

- mapState
- mapGetters
- mapMutations
- mapActions
- createNamespacedHelpers

以上方法都接收一个参数,返回对象

```
import { mapState, mapGetters, mapActions, mapMutations } from 'vuex'
export default {
  computed: {
    ...mapGetters({
      num: 'filterCount'
    }),
    ...mapState(['count']),
  },
  methods: {
    ...mapActions({
      addHandle: 'addAction'
    }),
    ...mapMutations({
      deHandle: 'deIncrement'
    })
  }
}

/**
多种写法
computed: {
  ...mapGetters({
    num: state => state.count,
    num: 'count',
    num(state) {
      return state.count + 10
    },
    count: 'count'
  })
},
computed: mapGetters(['count'])
*/
```

如需要传参数可以直接写在绑定事件上

```
<input type="button" value="-" @click="deHandle({de:5})">
```

---

## **vuex模块**

```
// 定义模块
let selectModule = {
  state: {
    title: 'hello123',
    list: []
  },
  getters: {
    // 接受 getter 作为第二个参数
    // 接受 rootState 根 state 作为第三个参数
    filter(state) {
      return state.count > 120 ? 120 ; state.count
    }
  },
  mutations: {
    changeTitle(state, payload) {
      state.title = payload.title
    },
    changeList(state, list) {
      state.list
    }
  },
  actions: {
    getListAction({ state, commit, rootState }) {
      // 发送请求
      axios.get('http://xxx.com')
        .then( data => {
          commit('changeList', data.data) //拿到数据后,提交mutations,改变状态
        })
        .catch( error => {
          console.log(error)
        })
    }
  }
}

let store = new Vuex.Store({
  modules: {
    selectModule
  }
})
```

state里的属性需要如下访问,其他不变

```
this.$store.state.selectModule.title
this.$store.commit('changeTitle', 5, 3)
this.$store.dispatch('getListAction', 5, 3)
```

---

## **插件**

vuex的store接受plugins选项.这个选项暴露出每次mutations的钩子. vuex插件就是一个函数,它接受store作为唯一参数

```
// 定义插件
const myPlugin = store => {
  // 当store初始化后调用
  store.subscribe( mutation, state ) => {
    // 每次 mutation 之后调用
    // mutation 的格式为 { type, payload }
  }
}

// 使用
const store = new Vuex.Store({
  // ...
  plugins: [myPlugin]
})
```

---

## **严格模式**

在严格模式下,无论何时发生了状态改变并且不是由mutation函数引起的,将会抛出错误.这能保证所有的状态变更都能被调试工具跟踪到.

```
const store = new Vuex.Store({
  // ...
  strict: true
})
```

---

## **devtools**

为某个特定的vuex实例打开或关闭devtools

```
{
  devtoos: false
}
```

---

## **module命名空间**

默认情况下,模块内部的action, mutation 和 getter 是注册在全局命名空间的,这样使得多个模块能够对同一 mutation 或 action 作出响应.

通过添加 `namespaced: true` 的方式使其成为带命名空间的模块.当模块被注册后,它的所有getter, action 及 mutation 都会自动根据模块注册的路径调整命名

### **带命名空间的模块内访问全局**

- 希望使用全局state和getter, rootState 和 rootGetter会作为第三和第四参数传入 getter, dispatch 也会通过 context 对象的属性传入 action  
- 若需要在全局命名空间内分发 action 或者提交 mutation, 将`{root: true}`作为第三参数传给 dispatch 或 commit 即可

```
dispatch('actionName')
// -> 'module/actionName'

dispatch('action', null, {root: true})
// -> 'rootStoreActionName'

commit('mutationName')
// -> 'module/mutationName'

commit('mutationName', null, {root: true})
// -> 'rootStoreMutationName'
```

### **在带命名空间的模块注册全局action**

需要在带命名空间的模块注册全局action,你可以添加 `{root: true}`,并将这个 action 的定义放在函数的 handler 中

```
module: {
  foo: {
    namespaced: true,
    action: {
      someAction: {
        root: true,
        handler(namespacedContext, payload) {
          // ...
        }
      }
    }
  }
}
```

### **带命名空间的绑定函数**

```
computed: {
  ...mapState({
    a: state => state.some.nested.module.a,
    b: state => state.some.nested.module.b
  })
},
methods: {
  ...mapActions([
    'some/nested/module/foo', // -> this['some/nested/module/foo']()
    'some/nested/module/bar' // -> this['some/nested/module/bar']()
  ])
}

// 将模块的空间名称字符串作为第一个参数传递给辅助函数,这样所有绑定都会自动将该模块作为上下文.

computeed: {
  ...mapState('somenested/module',{
    a: state => state.a,
    b: state => state.b
  })
},
methods: {
  ...mapActions('some/nested/module',[
    'foo', // -> this.foo()
    'bar', // -> this.bar()
  ])
}

//通过使用 createNamespacedHelpers 创建基于某个命名空间辅助函数。它返回一个对象，对象里有新的绑定在给定命名空间值上的组件绑定辅助函数：

import { createNamespacedHelpers } from 'vuex'

const { mapState, mapActions } = createNamespacedHelpers('some/nested/module')

export default {
  computed: {
    // 在 `some/nested/module` 中查找
    ...mapState({
      a: state => state.a,
      b: state => state.b
    })
  },
  methods: {
    // 在 `some/nested/module` 中查找
    ...mapActions([
      'foo',
      'bar'
    ])
  }
}
```

### **模块动态注册**

- 在store创建之前,你可以使用store.regsiterModule方法注册模块
- store.unregisterModule(moduleName)来动态卸载模块(创建store声明时的模块不能卸载)

### **保留state**

在注册一个新module时,你很有可能想保留过去的state,例如从一个服务端渲染的应保留state.

你可以通过preserveState选项将基归档: store.registerModule('a', module, {preverState: true})

当设置prevserveState: true时,该模块会被注册, action, mutation 和 getter 会被添加到 store 中,但是 state 不会.这里假设 store 的 state 已经包含了这个 module 和 state 并且你不希望将基覆盖.

## **父子组件之间改变状态**

- 1.x版本上的:propertyName.sync传递参数到子组件,当子组件的属性值改变时,父组件也会变
- 2.0版本废除
- 2.3版本之后又恢复了sync

所以使用:propertyName.sync,绑定属性时的属性值sync即可

父组件

```
<select-input :is-show.sync="listShow"></select-input>
<list v-show="listShow"></list>
```

子组件:

```
<button @click="showListHandle"></button>

<script>
export default {
  rops:['isShow'],
  computed: {
    initShow() {
      return this.isShow
    }
  },
  methods: {
    showListHandle() {
      this.$emit('update:isShow',!this.initShow)
    }
  }
}
</script>
```

---

## **兄弟组件之间改变状态**

需要使用父组件作为通道,父组件传入子组件自定义的事件,在子组件触发某个事件时,使用$emit来触发父组件传入的自定义事件,来改变兄弟组件有状态

需求:
- 父组件的title值传入了select-input组件作为input的值
- list组件的列表项点击时需要改变title值
- 以改变兄弟组件select-input的input值

实现:

1. 父组件传入list组件自定义事件changeTitle
2. 在list组件有列表项点击时,触发getTitleHandle事件处理函数,然后再来触发自定义事件changeTitle来改变父组件的title值

父组件

```
<select-input :is-show.sync="listShow" :title="title"></select-input>
<list v-show="listShow" @changeTitle="titleHandle"></list>
<script>
exprot default{
    data(){
        return {
            listShow: flase;
            title:’’
        }
    },
    methods:{
        titleHeadle(title){
            this.title=title;
        }
    }
}
</script>
```

select-input组件

```
<input @click="showListHandle" :value="this.props.title">
```

list组件
```
<template>
<ul class="list">
    <li v-for="item in data" @click="getTitleHandle(item.title)">{{item.title}}</li>
</ul>
</template>
<script>
exprot default{
    data(){},
    methods:{
        getTitleHandle(title){
            this.$emit('changeTitle',title)
        }
    }
}
</script>
```
