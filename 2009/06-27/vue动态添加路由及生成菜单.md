## **动态生成路由**

利用vue-router的addRoutes方法可以动态添加路由

官方文档:  
**router.addRoutes**  
```
router.addRoutes(routes:Array<RouteConfig>)
```

动态添加更多的路由规则.参数必须是一个符合routes选项要求的数组.

举个例子:

```
const router = new Router({
  routes:[
    {
      path:'/login',
      name:'login',
      component: () => import('@/compemtnts/login.vue')
    },
    {
      path:'/',
      redirect:'/home'
    }
  ]
})
```

上面的代码和下面的代码效果是一样的

```
const router = new Router({
  routes:[
    {
      path:'/',
      redirect:'/home'
    }
  ]
})

router.addRoutes({
  {
    path:'/login',
    name:'login',
    compenent: () => import('@/components/login.vue')
  }
})
```

在动态添加路由的过程中,如果有404页面,一定要放在最后添加,否则在登陆的时候添加完页面会重定向到404页面.

类似于这样,这种规则一定要最后添加

```
{path:'*',redirect:'/404'}
```

## **动态生成菜单**

假设后台返回 来的数据长这样

```
// 左侧后台返回来的数据长这样

menuItems:[
  {
    name:'home',//要跳转的路由名称,不是路径
    size:18,//icon大小
    type:'md-home',//icon类型
    text:'主页'//文本内容
  },
  {
    text:'二级菜单',
    type:'ios-paper',
    children:[
      {
        type:'ios-grid',
        name:'t1',
        text:'表格'
      },
      {
        text:'三级菜单',
        type:'ios-paper',
        children:[
          {
            type: 'ios-notifications-outline',
            name: 'msg',
            text: '查看消息'
          },
          {
            type: 'md-lock',
            name: 'password',
            text: '修改密码'
          },
          {
            type: 'md-person',
            name: 'userinfo',
            text: '基本资料'
          }
        ]
      }
    ]
  }
]
```

来看看怎么将它转化为菜单栏,这里使用了iview组件,不用重复造轮子

```
<!-- 菜单栏 -->
<Menu ref="asideMenu" theme="dark" width="100%" @on-select="gotoPage" 
accordion :open-names="openMenus" :active-name="currentPage" @on-open-change="menuChange">
    <!-- 动态菜单 -->
    <div v-for="(item, index) in menuItems" :key="index">
        <Submenu v-if="item.children" :name="index">
            <template slot="title">
                <Icon :size="item.size" :type="item.type"/>
                <span v-show="isShowAsideTitle">{{item.text}}</span>
            </template>
            <div v-for="(subItem, i) in item.children" :key="index + i">
                <Submenu v-if="subItem.children" :name="index + '-' + i">
                    <template slot="title">
                        <Icon :size="subItem.size" :type="subItem.type"/>
                        <span v-show="isShowAsideTitle">{{subItem.text}}</span>
                    </template>
                    <MenuItem class="menu-level-3" v-for="(threeItem, k) in subItem.children" :name="threeItem.name" :key="index + i + k">
                        <Icon :size="threeItem.size" :type="threeItem.type"/>
                        <span v-show="isShowAsideTitle">{{threeItem.text}}</span>
                    </MenuItem>
                </Submenu>
                <MenuItem v-else v-show="isShowAsideTitle" :name="subItem.name">
                    <Icon :size="subItem.size" :type="subItem.type"/>
                    <span v-show="isShowAsideTitle">{{subItem.text}}</span>
                </MenuItem>
            </div>
        </Submenu>
        <MenuItem v-else :name="item.name">
            <Icon :size="item.size" :type="item.type" />
            <span v-show="isShowAsideTitle">{{item.text}}</span>
        </MenuItem>
    </div>
</Menu>
```

代码不用看得太仔细,理解原理即可,其实就是通过三次`v-for`不停的对子级数进行循环,生成三级菜单.动态菜单就这样可以实现了.

**动态路由**,因为上面已经说过了用addRoutes来实现,现在看看具体怎么做

首先,要把项目所有的页面路由都列出来,再用后台返回的数据动态匹配,能匹配上的就把路由加上,不能匹配上的就不加,最后把这个新生的路由用addRoutes添加到路由表里

```
const asyncRoutes = {
  'home':{
    path:'home',
    name:'home',
    component:() => import('@/components/home.vue')
  },
  't1':{
    path:'t1',
    name:'t1',
    component:() => import('@/components/t1.vue')
  },
  'passwoed':{
    path:'password',
    name:'password',
    component:() => import('@/components/password.vue')
  },
  'msg':{
    path:'msg',
    name:'msg',
    component:() => import('@/components/msg.vue')
  },
  'userinfo':{
    path:'userinfo',
    name:'userinfo',
    component:() => import('@/components/userinfo.vue')
  }
}

// 传入后台数据,生成跌幅列表
menusToRoutes(menusData)

// 将菜单信息围成对应的路由信息,动态添加
function menusToRoutes(data){
  const result = []
  const children = []

  result.push({
    path:'/',
    component:() => import('@/coponent/Index.vue'),
    children
  })

  data.forEach(item => {
    generatorRoutes(children,item)
  })

  children.push({
    path:'error',
    name:'error',
    component:() => import('@/component/Error.vue')
  })

  // 最后添加404页面,否则会在登陆成功后跳到404页面
  result.push(
    {
      path:'*',
      redirect:'/error'
    }
  )

  return route

  function generateRoutes(children, item) {
    if (item.name) {
        children.push(asyncRoutes[item.name])
    } else if (item.children) {
        item.children.forEach(e => {
            generateRoutes(children, e)
        })
    }
  }
}
```