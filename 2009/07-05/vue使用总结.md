从以下几个方面整理

- vue用法
- vue-router用法
- webpack打包
- 遇到的坑

## **Vue用法**

### **. 通常写的.vue文件,实则是写在vue的配置文件,最终将执行`new Vue()`的操作,传入这些配置**

```
new Vue({
  data(){
    return {
      name:'xx'
    }
  }
})
```

```
// 常编写的.vue文件
export default {
  data(){
    return {
      name:'xx'
    }
  }
}
```

---

### **. 常用组件可以在全局注册**

组件代码, TestComponent.vue

```
<template>
  <div>
    test component
  </div>
</template>
<script>
  export default {}
</script>
```

注册组件, Vue.component注册

```
// main.js
import Vue from 'Vue'
import TestComponent fron './TestComponent.vue'
Vue.component('test-component',TestComponent)
```

> 以上main.js中也可以通过Vue.use注册,其实实质上还是调用`Vue.component`,对应的use的需要有一个install函数,使用Vue.use将触发install,然后在install中执行Vue.component操作

```
// my-components.js

import TestComponent from './TestComponent.vue'
export default {
  install(Vue){
    Vue.component('test-component',TestComponent)
    // 这里还可以有多个注册 Vue.component , Vue.directive , Vue.filter等
  }
}

// main.js
import Vue from 'vue'
import MyComponents from './my-component.vue'
Vue.use(MyComponents)
```

---

### ***. 用的比较多的指令,过滤器,mixin,可以在全局配置*

```
// 注册指令
Vue.directive('')

// 注册过滤器
Vue.filter('filterName',(val,...args) => {
  console.log(args)
  return val
})
// 使用过滤器 {{ 'abc' | filterName(1,2,3) }}
// value = 'abc'   args为[1,2,3]

// 全局注册mixin(不建议)
Vue.mixin({
  mounted(){
    console.log('每个组件中都将执行这里')
  }
})
```

---

### **. keep-alive的使用**

keep-alive可以缓存组件,可以通过exclude排除不需要的组件,include包含需要缓存的,max定义最大缓存的组件数. exclude , include传入一个数组,元素对应组件的name属性,max传入数字

```
<keep-alive :include"['home','list']" :max="3">
  <router-view />
</keep-alive>
```

---

### **滚动条**

在页面跳转时,新打开的页面滚动条可能缓存了滚动条的位置,在中间的位置,进入页面或者路由钩子的afterEach中执行window.scroll(0,0)重置到顶部.

---

### **非直接父子级之间的通信**

使用一个空vue对象实现通信

```
// bus.js
import Vue from 'Vue'
export const Bus = new Vue()

// a.vue
import Bus from './bus.js'
Bus.$emit('event_add_cart',1)

// b.vue
import Bus from './bus.js'
Bus.$on('event_add_cart',num => {
  this.cartNum += num
})
```

> a.vue和b.vue,引用bus.js,Bus为同一个实例,并不会重复创建,bus.js就相当于一个高度中心,可以无数个组件,都和它连接,发布事件,然后bus将发布到每上个建立链接的组件

---

### **computed**

computed只有在依赖的data里面的值改变时,才会改变,优于直接在模板里执行表达式和方法,因为模板中的方法和模板,在每次Vue刷新视图时,都瘵重新执行一遍.

---

### **data**

data需要是一个函数返回的对象,如果直接赋值一个对象时,做不到隔离同一个组件的数据效果,都鼗公用同一份数据.

---

### **beforeDestroy**

在vue组件销毁(beforeDestroy)前,清理定时器和DOM事件绑定.

---

### **key**

在循环中使用key,提升渲染性能,key尽量不 用数组下标,有key则diff算法就比较key值,最大程度上使用组件,而不是销毁原来的组件后重建.

---

### **v-for和v-if优先级**

v-for可以和v-if同时使用,先循环再判断,所以可以有选择的渲染数组中的元素,而angular中是不支持同一个标签上同时存在for和if指令的.可能eslint会提示摄氏不建议洽使用,eslint建议使用改变数组源数据的方式来实现,可以在computed中使用数组的filter过滤掉不想要的数据.

---

### **模板中template的使用**

当几个同级元素,需要依赖同一个指令时,又不想添加额外的标签将这些元素包裹时,可以使用template,template将不会渲染.在template使用v-for时,key值需要添加在子元素上.

```
<template>
  <div>
    <template v-for="(item,index) in [1,2,3]">
      <div :key="index">
        {{item}}
      </div>
    </template>
    <template v-if="2>1">
      <div>2大于1</div>
    </template>
  </div>
</template>
```

---

### **data初始化时,对象类型的数据,如果不给初始化,单独改变值时,页面是不会响应值的变化的**

```
<template>
  <div>
    {{info.name}}
    {{info.age}}
    <div v-for="(item.index) in arr" :key="index">{{item}}</div>
  </div>
</template>
<script>
  export default {
    data(){
      return {
        info:{age:1},
        arr:[]
      }
    },
    mounted(){
      this.info.name = 'xx' //并不能触发页面更新
      this.info.age = 2 //可以触发页面更新
      this.info = {name:'xx'} //可以触发页面更新
    }

    // 同理数组
    this.arr[0] = 1 //不能触发页面更新
    this.arr = [1]  //可以触发页面更新
  }
</script>
```

> 因为Vue在初始化时需要对data进行使用`defineProperty`进行set和get的劫持,如果对象中的值为空,那就不会存在相应的set和get,所以两者方式,一个给对象里设置初始值,二个将对象改为一个新对象,而不是直接在上面添加属性和值

基于以上,还有一个使用技巧,则是,一些表彰如果依赖后台返回的一些数据初始化选择列表那么可以在赋值前,先在返回数组中加上一个属性,例如`isChecke`,然后再赋值给data

```
<template>
  <div>
    <template v-for="(item,index) in checkboxes">
      <input type="checkbox" v-model="item.isChecked" :key="index">
    </template>
  </div>
<template>
<script>
export default {
  data(){
    return {
      checkboxes:[]
    }
  },
  methods:{
    getData(){
      // 请求数据代码 ...

      let data = [name:'zs',name:'ls']  // 原请求返回数据
      this.checkboxes = data.forEach(item => Object.assign(item,{isChecked:false}))
    }
  }
}
</script>
```

---

### **开发组件时,可以用solt占位,内容在使用组件的地方填充,使用更加灵活**

---

### **在开发管理后台时,可以将通用的数据和方法,写在mixin里面,然后在页面中注入,如通用表格的页码配置数据,加载中状态值,还有获取一页,下一页的方法**

---

### **在查看大图的场景中,点开大力,安桌物理按键返回时,需要关闭大图,而不进行页面的跳转,则可以在点开大力的时候,pushState在history中添加一个路由,如果页面需要歧见关闭的功能,则需要在点关闭按钮时,手动触发history.go(-1)一下**

---

### **如果在例如一个操作结果页(比如支付完成),然后又不能手动配置分享内容的时候,又需要分享的是购买的商品时,可以通过replaceState方法改变路由但是又不会触发页面刷新,再分享时,app或者浏览器则会自动抓取到替换后的地址分享出去了.**

---

### **父组件可以通过@hook:created  @hook:moounted监听到子组件的生命周期**

---

### **errCaptured捕获子孙组件的错误**

---

### **slot插槽使用**

匿名插槽
```
// child.vue
<template>
  <header>
    <slot> all header</slot>
  </header>
</template>

// parent.vue
<template>
  <child>
    <div>this is custom header</div>
  </child>
</template>
```

> slot中可以有默认值

具名插槽

```
// child.vue
<template>
  <header>
    <slot name="left">left</slot>
    <slot name="right">right</slot>
  </header>
</template>

// parent.vue
<template>
  <child>
    <div slot="left">custom left</div>
    <div slot="right">custom right</div>
    <div slot="right">custom right2</div>
  </child>
</template>
```

> 具名插槽可以有一个以上同样name的填充.注意组件中用slot+name,使用时slot=name,这里容易混淆

带值传递插槽,slot-scope,也就是子组件的值传递给父组件

```
// child.vue
<template>
  <header>
    <slot :user="userinfo" :adress="adress"></slot>
  </header>
</template>
<script>
  export default {
    data(){
      return {
        userinfo:{name:'xx'},
        adress:{city:'sh'}
      }
    }
  }
</script>

// parent.vue
<template>
  <div>
    <Child>
      <template slot-scope="row">
        {{JSON.stringify(row)}} == {"user":{"name":"xx"},"adress":{"city":"sh"}}
      </template>
    </Child>
  </div>
</template>
```

带值传递插槽的使用场景,在element-ui中使用较多,当在非父组件中循环时,而是向子组件传递值,子组件去遍历时,父组件中的插槽无法拿到遍历的当前值,需要子组件在遍历的时候把值又给附加上solt上

```
// List.vue
<template>
  <ul class="table">
    <li class="row" v-for="(item,index) in dataList" :key="index">
      <slot :row="item">
        {{item}}
      </slot>
    </li>
  </ul>
</template>
<script>
  export default {
    props:['dataList']
  }
</script>

// parent.vue
<template>
  <div>
    <TestList :dataList="[1,2,3,4,5,6]">
      <template slot-scope="scope">
        {{scope.row*2}}
      </template>
    </TestList>
  </div>
</template>
```

> 于是就实现了,子组件反射又向父组件传递值

---

### **v-once进行渲染优化,v-once只会初始化一次,之后页面数据发生变化,v-once内的内容也不会发生变化**

```
<template>
  <div>
    <span v-once>{{dateNow}}</span>
    <span>{{dateNow}}</span>
  </div>
</template>
<script>
export default {
  data(){
    return {
      dateNow:Date.now()
    }
  },
  mounted(){
    setInterval(() => {
      this.dateNow = Date.now()
    },1000)
  }
}
</script>
```

> 测试可以看一以只有没有加`v-once`的时间在变

---

### **关于在created还是在mouonted生命金子里发送数据请求**

有些说法是放在created中会好一些,可以避免多渲染一次.基理论可能是数据如果加载得快,那么就不render默认初始值,而是直接拿到获取到的数据,然后render. 先给出结论,created和mounted中基本无差,两都的渲染次数是一致的.

```
<script>
export default {
  created(){
    this.name = 'hello'
    setTimeout(() => {
      this.name='xx'
    },0)
  },
  mounted(){
    this.name = 'xiao'
    setTimeout(() => {
      this.name='xx'
    },0)
  },
  render(h){
    onsole.log('执行渲染',this.name)
    return h('div',{},this.name)
  }
}
</script>
```

> 以上测试可知,执行渲染将进行一肉次,也就是说再快的数据返回,也要进行两次render的,因为请求和setTimeout一样都是异步的,所以它的执行结果是在事件队列中等着的,而render是当前执行栈中的同步方法,它是执行在事件队列中的方法之前的.

> 注释created,放开mounted,render方法则会执行三遍

>但是created和mounted中直接赋值则是有差别的,因为render会发生在mounted之前一次.也就是初始化时, created -> render -> mounted(右有更改,再次render -> 请求返回 -> 再次render)

> 所以最佳的处理方式是,同步更改数据,放在created中,这样一些初值在第一次渲染就能正确呈现,且比在mounted中少执行一遍render,异步更改的无所谓.

---

### **is的使用**

在一些特定的结构中使用组件,如ul下的li

```
//UserItem.js
<template>
  <li>this is useritem</li>
</template>

// container.js
<template>
  <ul>
    <li is="user-item"></li>
  </ul>
</template>
<script>
  import UserItem from './UserItem.js'
  export default {
    components:{
      'user-item':UserItem
    }
  }
</script>
```

---

## **Vue-Router的使用**

### **路由的简单使用**

```
import Vue from 'vue'
import VueRouter from 'vue-router'

impoprt App from './app.vue'
import Home from './home.vue'

Vue.use(VueRpiter)    // 注册 router-view,view-link组件

new Vue({
  el:'#app',
  router:new VueRouter({
    routes:[
      path:'/home',
      component:Home
    ]
  }),
  render:h => h(App)
})
```

> 以上仅是简单使用,在实际项目中,可以将 `new VueRouter()` 路由配置提出去,然后入口页引入

```
// router.config.js
export const RouterConf = new VueRouter({
  mode:'hash', // 默认hash,切换使用 # 模式, #/home; history模式没有 # , 需要服务器配合
  routes:[
    path:'/home',
    compomemt:Home
  ]
})
// 以上还可以把toutes的值数组单独出去放一个文件

// main.js
import {RouterConf} from './router.config.js'
new Vue({
  router:RouterConf
})
```

---

### **路由守卫的使用**

**全局路由守卫**

beforeEach所有页面进入前,可以在这里做权限管理,保存滚动条位置,保存当前页面数据状态,调用起显示页面加载动画等

```
// 接上面代码, router.config.js
import RouterConf from './router.config.js'

// to,将要跳转的地址信息; now,现在路由配置信息; next,执行跳转的方法,next(false)和没有next页面都将不跳转,可以`next('./login')`验证用户登录然后跳转登录,也可以next(false)阻止用户进入没有权限的页面.
RouterConf.beforeEach((to,now,next) => {
  let token = localStorage.getItem('token')
  if(!token && to.name!=='login'){
    next('/login')
  }else{
    next()
  }
})
```

afterEach,所有页面进入后.可以做一些页面标题的设置,初始化加到顶部等.

```
import Vue from 'vue'

// 拉上面代码,router.config.js
import RouterConf from './router.config.js'

//now表示当前路由配置对象,from表示上一个路由配置对象
RouterConf.afterEach((now,from) => {
  document.tite = now.meta.pageTitle
  // 如果需要在页面渲染完之后处理一些事情
  Vue.nextTick(() => {
    window.scroll(0,0)
  })
})
```

---

**组件路由守卫**

- beforeRouteEnter(to,from,next),组件加载进入时
- beforeRouteUpdate(to,from,next),路由地址更新后执行
- beforeRouteLeave(to,from,next),可以通过next(false)阻止页面离开.

```
<script>
  export default {
    mounted(){
      console.log('home mounted')
    },
    beforeRouteUpdate(to,from,next){
      next()
    },
    beforeRouteEnter(to,from,next){
      next()
    },
    beforeRouteLeave(to,from,next){
      // 可以弹个提示窗,用户是否确认离开,确认才离开当前页面,更严谨一点,需要判断是往后还是push新页面提交等等具体情况具体分析使用
      let confirmStatus = this.confirmStatus
      confirmStatus && next()
    }
  }
</script>
```

> 切记beforeRouteUpdate, beforeRouteLeave,都需要手动调用next才会进行下一步跳转

---

### **路由组件使用**

**router-view**

定义路由容器,路由引起的变化的内容都鼗在这个容器之中,配合keep-alive使用可缓存页面

```
<template>
  <div id="app">
    <div>
      不会随路由变化而变化的内容,可以存放一些全局使用的内容,如提示信息,通过使用vuex传递消息,或BusService传递消息,使得内容显示或者隐藏以及具体显示内容
    </div>
    <keep-alive :includes="['home','list']">
      <router-view></router-view>
    </keep-alive>
  </div>
</template>
```

**router-link**

定义路由跳转,传入to属性,为一个对象,可以有name(字符串),path(字符串),query(字符串),params(对象)

```
<!-- 假如有配置路由 {
  name:'detail',
  path:'/detail/:id'
} -->

<template>
  <router-link :to="{name:'detail',params:{id,1},query:{name:'xx'}}"></router-link>
  // 上面渲染出的链接 #/detail/1?name=xx
  <router-link :to="{path:'/detail',params:{id:1},query:{name:'xx'}}"></router-link>
  // 上面渲染出的链接 #detail?name=xx
  <router-link :to="{name:'detail',path:'/detail',params:{id:1},query:{name:'xx'}}">/<router-link>
  // 上面渲染出的链接 #/detail/1?name=xx
</template>
```

> 注意使用name时,vue将根据路由配置找到对应的name,然后还原出path,这时候如果路由配置的path是'/detail/:id'类似的形式,则将按这规则,还原完整地址.也就是当有path时,只照顾query参数

> 有name时,对应路由配置信息找出path,然后用params填充path,再拼上query

> name和path共存时,按name的规则走(尽量避免name和path同时出现)

---

### **路由方法的使用, $router $ $route**

**$router,常用的以下几个方法**

- push,参数可以是字符串,表示要跳转的地址,或一个对象,类似router-link的to的值
- replace,替换一个地址,参数和push相同
- back,回退上一个页面
- go, 回退或前进n个页面

```
export default {
  mounted(){
    this.$router.push('/detail/1')  //直接字符串形式
    this.$router.push({ // 对象形式
      name:'detail',
      query:{a:1},
      params:{id:1}
    })
    this.$route.back() // 后退一个页面
    this.$router.go(-2) // 后退两个页面
    this.$router.go(2)  // 前面两个页面
  }
}
```

**$route,路由参数信息对象(响应式)**

组件中获取跌路由参数,通过this.$route获取

- params.对应路由中的`detail/:id`配置,路由为`#detail/1`,则`this.$route.params.id`为`
- query,对应路由中的 ? 后的值,如路由为`#detail/1?name=xx`,则`this.$route.query.name`为xx

```
// 楞以在createed周期内获取
export default {
  data(){
    return {
      id;'',
      name:''
    }
  },
  created(){
    let {id} = this.$route.params
    let {name} = this.$route.query
    this.id = id || ''
    this.name = name || ''
  }
}
```

---

### **路由懒加载配置**

可以直接使用`()=>import`实现

```
// router.config.js
export const RouterConf = [
  {
    path:'/home',
    component:()=>import(/*webpackChunkName:"home"*/ './home.vue')
  }
]
```
webpack打包时,将以home命名 