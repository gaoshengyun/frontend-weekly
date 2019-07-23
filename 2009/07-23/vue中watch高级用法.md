假设有如下代码

```
<div id="root">
  <p>FullName: {{fullName}} </p>
  <p>FirstName: <input type="text" v-model="firstName"></p>
</div>

new Vue({
  el: '#root',
  data: {
    firstName: 'Dawei',
    lastName: 'Lou',
    fullName: ''
  },
  watch: {
    fitstName(newName, oldName) {
      this.fullName = newName + ' ' + this.lastName
    }
  }
})
```

上面的代码效果是,当我们输入firstName后,watch监听每次修改变化的新值,然后计算出fullName

---

## **handler方法和immediate属性**

这里watch的一个特点是,最初绑定的时候是不会执行的,要等到firstName改变时才执行监听计算.那我们想要一开始就让他最初绑定的时候就执行该怎么办呢?我们需要改一下我们的watch方法,修改后的watch方法如下

```
watch: {
  firstName: {
    handler(newName, oldName) {
      this.fullName = newName + ' ' + this.lastName
    }
    // 代表在watch里声明了firstName这个方法之年立即先去执行handler方法
    immediate: true
  }
}
```

注意到 handler 方法了吗,我们给 firstName 绑定了一个 handler 方法,之前我们写的 watch 方法其实默认写的就是 handler , vue会去处理这个逻辑,最终编译出来其实就是这个handler

而 `immediate: true` 代表如果在watch声明了 firstName 之后,就会立即执行里面的 handler 方法,如果为 false 就跟我们以前的效果一样,不会在绑定的时候就执行.

---

## **deep属性**

watch 里面还有一个属性 deep , 默认值就是 false , 代表是否深度监听, 比如我们 data 里有一个 obj 属性

```
<div id="root">
  <p>obj.a: {{obj.a}}</p>
  <p>obj.a: <input type="text" v-model="obj.a"></p>
</div>

new Vue({
  el: '#root',
  data: {
    obj: {
      a: 123
    }
  },
  watch: {
    obj: {
      handler(newName, oldName) {
        console.log('obja changed')
      },
      immediate: true
    }
  }
})
```

当我们在输入框中输入数据试图改变obj.a的值时,我们发现是无效的.受现在JavaScript的限制(以及废弃的Object.observe), vue不能检测到对象属性的添加或删除.由于vue会在初始化实例时对属性执行getter/setter转化过程,所以属性必须在data对象上存在才能让vue转换它,这样才能让它是响应的.

默认情况下handler只监听obj这个属性它的引用的变化,我们只有给obj周全的时候它地和会监听到,比如我们在mounted事件钩子函数中对obj进行重新赋值

```
mounted() {
  this.obj = {
    a: '456'
  }
}
```

这样我们的handler才会执行,打印obj.a changed

相反,如果我们需要监听obj里的属性a的值呢?这时候deep属性就派上用场了

```
watch: {
  obj:{
    handler(newValue, oldValue) {
      console.log('obj.a changed')
    },
    immediate: true,
    deep: true
  }
}
```

deep的意思就是深入观察,监听器会一层层往下遍历,给对象的所有属性都加上这个监听器,但是这样性能开销就会非常大了,任何修改obj里面任何一个属性都会触发这个监听器里的handler

优化,我们可以使用字符串形式监听

```
watch: {
  'obj.a': {
    handler(newValue, oldValue) {
      console.log('obj.a changed')
    },
    immediate: true
  }
}
```

这样vue才会一层一层解析下去,直到遇到属性a,然后才给a设置监听函数

---

## **注销watch**


为什么要注销watch? 因为我们的组件是经常要被注销的,比如我们跳一个跌幅,从一个页面跳到另外一个页面,原来的页面和watch其实就没用了,这时候我们应该注销掉原来页面的watch的,不然的话可能会导致内存泄露.好在我们平时watch都是写在组件的选项中,他会随着组件的销毁而销毁.

```
const app = new Vue({
  twmplate: `<div id="root">{{text}}</div>`,
  data: {
    text: 0
  },
  watch: {
    text(newVal, oldVal) {
      console.log(`${newVal} : ${oldVal}`)
    }
  }
})
```

但是,如果我们使用下面这样的写法写watch,那么就要手动注销了,这种注销其实也很简单

```
const unWatch = app.$watch('text',(newVal, oldVal) => {
  console.log(`${newVal} : ${oldVal}`)
})
unWatch() // 手动注销
```

app.$watch调用后会返回一个值,就是unWatch方法,你要注销watch只要调用unWatch方法就可以了.