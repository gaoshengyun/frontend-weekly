## **书写习惯**

### **不需要做內式的数据,不要放在data中**

响应式数据:每个Vue实例都会代理其data对象里所有的属性,只有这些被代理的数据昌响应式的,在其数据改变时视图也会随之更新.

在每个Vue组件中都有一个data对象.为要把所有数据都放在data中,只把需要的做响应式的数据放在data对象中,原因是:如果一个数据存在于data中,数据会被劫持,vue会给数据添加一个getter(获取数据),一个setter(设置数据),性能不会高.

### **在模板里不要写太多表达式**

```
v-if="isShow && isAdmin && (a || b)"
{{haorooms ? haorooms : (resource ? resource : "haoroomsresource")}}
```

在如上的表达式中,虽然没有出现语法错误或其它错误,但当表达式过多时,可以 通过其他方式,例如通过 methods 或 computed 封装方法.

用方法的好处是方便我们在多处判断相同的表达式,其他权限相同的元素再判断展示的时候调用同一个方法即可.

### **能拆分的组件尽量拆分**

将能够拆分的组件尽可能拆分,可使得粒度尽可能的小,其优点在于有得提高利用性,增加代码的可维护性,减少不必要的渲染.

### **SPA(Single Page Application)**

SPA(Single Page Application)也就是指的单页,通过js代码写的路由来实现,将不同的组件扔到app里面

其特点如下:  
1. 当它被打包之后 index.html 中只存在一行代码

```
<div id="app"></div>
```

2. 虽然打包后只有一行代码,但仍然能渲染出页面,是因为内容是通过加载js代码,然后通过webpack打包插入到div中的

3. 单面不得SEO(搜索引擎优化),因为路由是动态加载的,当不使用路由时,div标签中是啥都没有的,不得爬虫.(可利用SSR技术解决)

4. keep-alive组件,它可以实现组件的缓存,把组件中的结构和数据全部缓存到内存.

### **v-if和v-show区别与联系**

v-show控制的是display,不管显示是否都会渲染的; v-if若是不显示根本就不会渲染.注意点: v-show一个组件,可以让组件重新渲染,通过v-if,我们要能让组件按照一定执行循序执行.

结论: 能使用v-if的时候尽量使用v-if,频繁让组件显示或隐藏使用v-show,不频繁让组件显示或隐藏使用v-if

### **循环调用子组件时添加key**

在循环调用子组件时,使用到v-for,必须加上key,否则编辑器会出现警告.一般来说,循环key最好是不要使用index作为key,尽量使用id作为key.使用id作为key可以避免重复渲染.

vue是虚拟DOM,最终渲染时还是要求找到真实DOM,浏览器是真实DOM,在加上key后,当dom树上的节点发生改变时,可以迅速定位到那里,否则需要对其进行遍历.

### **尽量不要使用float布局**

尽量不要使用float布局,心可能去使用flex或者grid布局

### **Object.freeze作用**

Object.freeze会实现数据劫持,进行冻结,当页面上的数据不想经常改变时,可以进行冻结,不再使用getter和setter.

Object.freeze()可以去除Observer监听,对于长期不变的大数据,去除监听,利用Object.freeze方法改变数据时,视图也会更新(改变数据,改变DOM)

```
export default {
  name: 'haorooms test',
  data() {
    return {
      list: Object.freeze([
        {value: 1},
        {value: 2}
      ])
    }
  },
  created() {
    this.list[0].value = 100
    console.log(this.list)
    this.list = []
    console.log(this.list)
  }
}
```

虽然Object.freeze可以取消监听,但还是可以改变数据,改变DOM,并且当组合list重新赋值之后,又恢复了监听.

### **vue-router路由优化**

路由懒加载可以实现路由的优化

```
import home from './view/home' 直接在组件中引入完毕,然后珍该组件

() => import('./view/user')
```

### **动态导入组件**

为项目优化,将组件可使用reqire或者import按需导入

```
原始导入:
  导入: import home from './component/home'
  注册: component: {home}

动态导入: components: {home: () => ('./component/home')}
```

### **vue中使用run time**

Runtime + Compiler 意思是运行时 + 编译器,是不在打包时进行编译,而是在浏览器(客户端)运行时进行编译,所以需要使用带编译器的完整版本

Runtime-only意思是只包含运行时版本,是在构建时通过webpack的vue-loader工具把模板预编译成JavaScript,也就是进行了预编译好的,在浏览器中可直接运行.

---

## **加载**

### **图片的懒加载**

并非在一开始就将图片加载完成,而是在用到的时候再去请求数据,从而加载出来图片

### **第三方模块按需导入**

根据第三方模块网站说明的使用方法来引入模块,需要的话可能还需要在webpack中进行配置.

---

## **提升用户体验**

### **骨架屏**

数据加载loading完成之前存在着虚框,就好像数据要请求回来一样;当数据请求回来以后,骨架屏也就不存在了.骨架屏是用第三方模块[vue-skeleton-webpack-plugin](https://www.npmjs.com/package/vue-skeleton-webpack-plugin)

### **PWA**

PWA是用来做缓存的,以提升用户体验,一般网页只能使用明讲打开,但通过PWA可用手机打开.

在弱网情况下(2G,3G),需要PWA进行缓存,现将原来缓存的数据显示出来,等加载完毕再用新数据来替换原本缓存过的旧数据.

### shell

做一个网站,本质上是一个页面,通过浏览器打开,想要使其像传统app一样,可以下载,安装,上传到用户商店.加上shell这样一个壳之后,它本质上跑的是H5,不过可以下载,安装,性能与原生app来相比较的话,没有原生的好.

---

## **SEO搜索引擎优化**

### **预渲染**

在最初的时候,通过html代码,使用axios去请求数据,然后再渲染,网络环境不好的时候可能会出现白屏.预渲染是在真实数据请求回来之前,已渲染出数据(假数据),而当请求回来正式数据之后再奖其进行替换.

### **SSR服务端渲染**

SSR服务端渲染可解决SEO优化的问题,因为SEO爬虫在爬取页面信息的时候,会发送HTTP请求来获取网页内容,而我们SSR服务端渲染道光的数据是后端返回的,返回的时候已经是渲染好的内容等信息了,便于爬虫抓取内容.

---

## **从前端角度来优化**

### **浏览器缓存**
[http总结](https://juejin.im/post/5d97efe251882502c5534b0e)

### **浏览器的压缩**

压缩优势: 让资源文件更小,加快文件在网络的传输,让网页更快的展现,降低宽带和流量和开销

压缩方式: js, css, 图片, html的代码压缩, gzip压缩

1. js代码压缩,一般就是去除多余的空格和回车,替换长变量名,简化一些代码写法等;其常用的压缩工具: UglifyJS(压缩, 语法检查, 美化代码, 代码缩减, 转化), YUI Compressor(只有压缩), Closure(和UglifyJS类似,压缩方式不一样), Compiler

2. css压缩,去除空白符,注释优化一些css主义规则,其常用的压缩工具: YUI Compressor(只有压缩), CSS Compressor

3. HTML压缩,不建议使用代码压缩,建议使用gzip压缩

4. 图片压缩, tinypng, jpegMini, ImageOption

5. gzip压缩, 它需要在ngix中配置压缩文件