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