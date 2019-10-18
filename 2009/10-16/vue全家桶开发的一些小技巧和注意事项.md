### **css的scoped属性**

vue为了防止css污染,当组件的style标签有scoped属性时,它的css只作用于当前组件中的元素.实现原理是给当前组件中的每个标签都加上唯一的自定义属性: `data-v` , 然后css选择器都加上属性选择器,这样这个css只会匹配到当前页面的这个元素.

> 注意: 每个组件的最外层的标签都会 带上父组件的`datq-v`属性,也就是这个标签会被父组件的样式匹配到,所以父组件尽量不要使用标签选择器,这个标签不要使用父组件中的id或者class

在父组件中想修改子组件的css,可以使用尝试作用选择器 `>>>`
> 注意: 在sass或less中可能无法识别尝试选择器,可以使用/deep/选择器

---

### **父子组件的生命周期钩子函数执行先后顺序**
组件的彺周期钩子函数是某个到了生命周期点就会触发,而不是在这个金子函数中进行生命周期. 比如说DOM加载好了,就会触发mounted钩子函数,所以在created里写一个延迟定时器,mounted金子不会等定时器执行.

各个周期钩子函数触发的时间点参数(图片来源于网络)
![](https://user-gold-cdn.xitu.io/2019/9/26/16d6c8f3c5cace22?imageslim)

关于父子组件的生命周期: 不同的钩子函数有不同的表现.父组件的虚拟DOM先初始化好了(beforeMount),才会去初始化子组件的虚拟DOM,而mounted事件,等价于window.onload,子组件DOM没加载好,父组件DOM永远不可能加载好,所以基本生命周期钩子函数执行顺序是: 父beforeCreate -> 父created -> 父beforeMounted -> 子beforeCreate -> 子created -> 子beforeMount -> 子mounted -> 父mounted

父子组件的update和beforeUpdate执行先后顺序: 数据修改 + 虚拟DOM准备好会触发beforeUpdate,换句话说beforeUpdate等价于beforeMoounted,而update等价于mounted.所以先后顺序是: 父beforeUpdate -> 子beforeUpdate -> 子update -> 父update.

同埋,beforeDestory和destory的先后顺序是: 父beforeDestory -> 子beforeDestory -> 子destory -> 父destory

生命周期钩子函数其实也可以写成数组形式: mounted: [mounted1,mounted2],同一个生命周期可以触发多个函数,这也是mixin混入的原理,mixin里面也可以写生命周期钩子,最终会和组件里面的生命周期钩子函数一起变成数组形式,mixin里面的钩子函数会先执行.

---

### **异步请求数据在哪个函数中执行比较好**

生命周期钩子函数中的异步会放入事件队列中,而不会在这个钩子函数中执行.也就是说你在created和mounted中请求数据是一样的,都不会立即更新数据,所以不会导致虚拟DOM重新加载,也不影响页面中静态部分的加载.生命周期钩子函数中的异步赋值,vue会在一遍流程走完之后再执行update.另外,给数据赋值然后更新DOM也是异步的,侦听到数据变化,Vue将开启一个队列,并缓冲在同一事件循环中发生的所有数据变更,去掉重复赋值然后更新.

生命钩子函数中异步行为测试

```
export default {
    data(){
        return {
            list:[],
        }
    },
    methods:{
        getData(){
            //生成指定范围的随机整数
            const randomNum = (min, max) => Math.floor(Math.random() * (max - min + 1)) + min; 
            //生成固定长度的非空数组
            const randomArr = length => Array.from({ length }, (item, index) => index * 2); 
            const time = randomNum(100,3000);//模拟请求时间
            console.log('getData start');
            return new Promise(resolve => {
                setTimeout(() => {
                    const arr = randomArr(10);
                    resolve(arr);
                },time)
            })
        }
    },
    async created(){
        console.log('created');
        this.list = await this.getData();
        console.log('getData end');
    },
    beforeMount() {
        console.log('beforeMount');
    },
    mounted(){
        console.log('mounted');
    },
    updated(){
        console.log('updated');
    }
}

```

结果: 

```
created
getGata start
beforeMount
mounted
updated
getData end
updated
```

所以在created中和mounted中请求数据,数据的更新时间是一样的,在created中发起请求,可以更早的请求到数据,并使用服务端渲染SSR的时候,mounted钩子不会加载.

---

### **父组件监听子组件的生命周期**

可以写自定义事件,然后在子组件的生命周期函数中触发为个自定义事件,但是不优雅,我们可以使用hook

```
<child @hook:created="childCreated"></child>
```

从A页面切换到B页面,A页面中有一个定时器,到了B页面用不上,需要在离开A页面的时候清除掉,办法很简单,在A页面的生命周期钩子函数beforeDestory或者路由钩子函数beforeRouterLeave里面清除就行了，但是问题来了，怎么拿到定时器呢？把定时器写到data里面，可行但是不优雅，我们有如下写法

```
// 在初始化定时器之后
this.$once('hook:beforeDestory', () => {
  clearInterval(timer)
})
```

---

### **is属性**

由于HTML标签的限制，tr标签里面只能有th，td标签，而写自定义标签则会被解析到tr标签外层，所以这时候我们可以用is属性

```
<tr>
  <td is="child"></td>
</tr>
```

---

### **给事件传额外参数**

原生DOM事件绑定的函数的第一个参数都会是事件对象event，但是有时候我们想给这个函数传递其他参数,直接传会覆盖掉event,我们可以这么写

```
<div @click="clickDiv(params,$event)"></div>
```

变量$event就代表对象.

如果要传的变量不是事件对象呢?在使用elementUI的时候碰到一个情况,在表格中使用了下拉菜单组件

```
<el-table-column label="日期" width="180">
    <template v-slot="{row}">
        <el-dropdown-item @command="handleCommand">
            <span>
                下拉菜单<i class="el-icon-arrow-down el-icon--right"></i>
            </span>
            <template #dropdown>
                <el-dropdown-menu>
                    <el-dropdown-item command="a">黄金糕</el-dropdown-item>
                    <el-dropdown-item command="b">狮子头</el-dropdown-item>
                </el-dropdown-menu>
            </template>
        </el-dropdown-item>
    </template>
</el-table-column>
```

下拉菜单事件command函数自带一个参数,为下拉先中的值,这个时候我们想把表格数据传过去,如果`@command="handleCommand(row)"`这样写,就会覆盖掉自带的参数,该怎么办?这时候我们可以借助箭头函数: `@command="command => handleCommand(row,command)"` 完美解决传参问题.

顺便说一下, elementUI的表格可以用变量 $index 代表当前的列数,和 $event 一样的使用

```
<el-table-column label="操作">
    <template v-slot="{ row, $index }">
        <el-button @click="handleEdit($index, row)">编辑</el-button>
    </template>
</el-table-column>
```

还可以使用扩展运算符
```
@current-change="{...defaultArgs} => treeClick(otherArgs, ...defaultArgs)"
```

---

### **v-slot语法**

v-slot 的用法(slot语法已废弃): 相当于在组件中留一个空位,使用该组件的时候可以传一些标签过去,插入到对应的空位,可以有多个空位,取不同的名字即可.默认是default,同时还可以将一些数据传过去,简写是#

```
<!-- 子组件 -->
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>

<!-- 父组件 -->
<base-layout>
    <!-- 插槽可以简写为# -->
    <template #header="data">
        <h1>Here might be a page title</h1>
    </template>
    <!-- v-slot:default可省略 -->
    <div v-slot:default>
        <p>A paragraph for the main content.</p>
        <p>And another one.</p>
    </div>
    <!-- 可以使用解构 -->
    <template #footer="{ user }">
        <p>Here's some contact info</p>
    </template>
</base-layout>
```

总结:
- 其他名称的slot(非default)仅能用于template标签
- 插槽里面的标签拿不到传给子标签的数据(插槽相当于孙子组件)

```
<child :data="data">
  <div>这里访问不到data数据</div>
</child>
```

- 插槽可以使用解构语法 `v-slot="{user}"`

---

### **子组件修改父组件传过来的值**

v-model 在使用的时候很向双向绑定,但是vue的单向数据流, v-model只是语法粮而已: 父组件用v-bind将值传给子组件,子组件通过change/input事件触发修改父组件的值

```
<input v-model="inputValue">
<!-- 等价于 -->
<input :value="inputValue" @change="inputValue = $event.target.value">
```

v-model 不仅仅能用在input上用,在组件上也能使用

vue组件间传递数据是单向的,即数据总是由父组件传递到子组件,子组件在其内部可以有自己维护的数据,但它无权修改父组件传递给它的数据,我们可以参照v-mode语法糖修改父组件的值,但是每次都这样写太麻烦,vue提供了一个修饰符 .sync, 用法如下:

```
<child :value.sync="inputValue"></child>
<!-- 子组件 -->
<script>
export default {
    props: {
        //props可以设置值得类型，默认值，是否必传以及校验函数
        value: {
            type: [String, Number],
            required: true,
        },
    },
    //用一个变量中转，子组件中就用_value就不会直接修改父组件的值
    computed: {
        _value: {
            get() {
                return this.value;
            },
            set(val) {
                this.$emit('update:value', val);
            },
        },
    },
};
</script>

```

---

### **父组件通过ref访问到子组件**

虽然vue提供了$parent和$children来访问父/子组件,但是组件的父组件/子组件存在很多不确定性,例如组件被利用,他的父组件有多种情况,我们可以通过ref访问到子组件的数据和方法

```
<child ref="myChikd"></child>

<script>
export default {
  async mounted() {
    await this.$nextTick()
    console..dir(this.$refs.myChild)
  }
}
</script>
```

注意:

- ref必须等DOM加载好了才可以访问
- 虽然mounted生命周期DOM已经加载好了,但是为了以防万一,我们可以使用$nextTick函数

---

### **背景图, css的@import使用路径别名**

在用 webpack 处理打包时,可以将某一目录配置一个别名,代码中就能使用与别名的相对路径引用资源

但是在css文件中,如less,sass,stylus中,使用 `@import "@/style/theme"` 的语法引用相对 `@` 的目录会报错.解决办法是在引用路径的字符串前加上 `~` 符号.

- css module 中: `@import "~@/style/theme.less"`
- css属性中: `background: url("~@/assets/xxx.png")`
- html标签中: `<img src="~@/assets/xxx.png">`

---

### **vue-router 的 hash 模式和 history 模式**

vue是单页应用,打包之后只有一个index.html文件,将他部署到服务器上之年,访问对应文件的目录就是访问这个文件.

hash 模式: 网址后面跟着 hash 值, hash 值对应每一个 router 的名称, hash 值改变意味着 router 改变, 监听 onhashchange 事件, 来替换页面内容.

优缺点:

1. hash 模式的 # 号很丑, 使用的是 onhashchange 事件切换路由, 兼容性会好一点, 不需要服务器配合
2. history 模式好看点, 但本地开发, 网站上线, 都需要服务器额外配置, 并且还需要自己写404页面, 使用 HTML5 的 history API, 兼容性差一点.

两者区别在于:

```
const router = new VueRouter({
    mode: 'history',	// hash 模式是默认的,无需配置
		base: '/',	//默认配置
		routes: [...]
})
```

vue-cli3 的 vue.config.js 配置:

```
module.exports = {
	publicPath: './',	// hash模式打包用
	// publicPath: '/',	// history模式打包用
	devServer: {
		open: true,
		port: 8080,
		// historyApiFallBack: true	// history 模式本地开发用
	}
}
```

如果是网站根目录, router 的 base 就不用填. 如果整个单页应服务在 /app/下, 然后 base 就应该设为 '/app/', 同时打包配置的 publicPath 也应该设置成 '/app/'

vue-cli3 生成新项目的时候会有选择路由的模式,如果选择 history 模式就会帮你都配置好.

---

### **vue-router的钩子函数**

钩子函数分三种: 组件内钩子, 全局钩子, 路由独享钩子.

APP.vue 没有组件内钩子函数,因为 APP.vue 是页面入口, 这个组件是必定会加载的, 而使用组件内钩子函数可以阻止组件加载.

全局钩子主要用于路由鉴权, 但是消耗很大. 组件内的钩子 `beforeRouterLeave` 主要用于用户离开前的提示 (比如说有未保存的文章), 这个钩子有一些坑: hash模式下, 浏览器的后退按钮无法触发这个钩子函数. 同时我们还可以监听用户的关闭当前窗口/浏览器事件

```
window.onbefreunload = e => '确定离开当前页面?你的修改将不会被保存!'
```

为了防止恶意网站, 用户关闭窗口/浏览器事件是不可阻止的,只能昧,而且不同的浏览器兼容性也不同.

---

### **Vuex持久化存储**

Vuex中的数据, 刷新页面之后就会丢失. 要实现持久化存储需要借助本地存储 (cookie 或 storage等), 一般是登录之后返回的数据 (角色, 权限, token等) 需要存储到Vuex, 所以我们可以在登录页面将数据存储到本地, 而在主页面 (除了登录页, 其他所有页面的入口) 进入之前 (beforeCreate 或者路由钩子 beforeRouterEnter) 读取出来, 并提交到 Vuex 就好了. 这样即使刷新, 也会触发主页面的进入钩子函数, 会被提交到 Vuex.

```
beforeRouterEnter(to, from, next) {
	const token = localStorage.getItem('token')
	let right = localStorage.getItem('right')
	try {
		right = JSON.parse(right)
	} ctach {
		next(vm => {
			// 弹空采用elementUI
			vm.$alert('获取权限失败').finally(() => {
				vm.$router.replace({name: 'login'})
			})
		})
	}

	if(!right || !token) {
		next({name: 'login', replace: true})
	} else {
		next(vm => {
			// 这里的事件会在mounted之后触发
			vm.$store.commit('setToken', token)
			vm.$store.commit('setRight', rght)
		})
	}
}
```

beforeRouteEnter 的回调会在 mounted 钩子之后触发, 这就比较蛋疼了. 而主页的 mounted 会在所有子组件的 mounted 之后触发,所以我们可以这样写:

```
import store from '^/store'	// 将实例化的store引入进来

beforeRouterEnter(to, from, next) {
	const token = localStorage.getItem('token')
	if(!token) {
		next({name: 'login', replace: true})
	} else {
		store.commit('setToken', token)
		next()
	}
}
```

要想实现数据修改之后仍能持久化, 我们可以先把数据存到 localStorage, 然后监听 window.onstorage 事件, 数据有修改提交到 Vuex.

---

### **mutations 里面触发 action**

mutation 是同步修改 status 的值, 假如另一个值是异步获取 (action) 的, 依赖于这个同步的值的修改, 需要在 mutations 里面赋值之前触发 action 里面的事件, 我们可以给实例化的 Vuex 命名, 在 mutations 里面拿到 store 对象.

```
const store = new Vuex.Store({
	state: {
		age: 18,
		name: 'zhangsan',
	},
	mutations: {
		setAge(state, val) {
			// 假如age变化之后, name也要跟着变化
			// 需要在每次给age赋值的时候, 同步触发action里面的getName
			state.age = val
			store.dispatch('getName')
		},
		setName(state, val) {
			state.name = val
		}
	},
	actions: {
		getName({commit}) {
			const name = feth('name')	// 从接口异步获取
			commit('setName', name)
		}
	}
})
```

---

### **Vue.observable进行组件通信**

如果项目很小, 为需要用到 vuex, 可以用 Vue.observable 来模拟一个:

```
// store.js
import Vue from 'vue'

const store = Vue.observable({name: '张三', age: 20})
const mutations = {
	setAge(age) {
		store.age = age
	},
	setName(name) {
		store.name = name
	}
}
export {store, mutations}
```

---

### **axios 的 qs 插件**

get请求的数据放在 url 里面, 类似于 http://www.xxx.com?a=1&b=2, 其中 a=1&b=2 就是get的参数, 而对于 post 请求, 参数放到 body 里面, 常用的数据格式有表单数据和 json 数据, 两者的差异就是数据格式不同, 表单数据编码和 get 一样, 只不过是放在 body 里面, 而 json 数据则是 json 字符串.

qs 基本使用

```
import qs from 'qs' // qs 是 axios 里面自带的, 所以直接引入就可以了

const data = qa.stringify({
	username: this.formDara.username,
	oldPassword: this.formData.oldPassword,
	newPassword: this.formData.newPassword1
})
this.$http.post('/changePassword.php', data)
```

qs.parse() 是将 URL 解析成对象的形式, qs.stringify() 是将对象序列化成 URL 的形式, 以&进行拼接,而对于不同的数据格式, axios 坐自动设置对应的 conent-type, 不需要手动设置

一个接口需要用表单传一个数组,假设数据是 `arr = [1,2,3]` 如果直接使用qs.stringify(), 则数据会变成 `arr[]=1&arr[]=2&arr[]=3`, 很容易看出来多了个 `[]`, 让接口把参数改名成 `arr[]` 就能使用, 但这样不好, 但是这样不好, 不过可以发现, 表单数组的本质就是同名参数传多次, 这时候我们也可以这样

```
const data = new FormData()
arr.forEach(item => {
	data.append('arr', item)
})
```

完美解决, qs 转换支持第二个参数, 完美解决我们的问题

```
const data = qs.stringify(arr, {arryFormat: 'repeat'})
```