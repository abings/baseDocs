# 权限模型
  ![权限模型](/_media/2019-01-05-10-33-03.png)
  
  * 实现了RBAC模型权限控制
  * 菜单与路由独立管理，完全由后端返回
  * **user**存储用户
  * **admin**标识用户是否为系统管理员
  * **role**存储角色信息
  * **roleUser**存储用户与角色的关联关系
  * **menu**存储菜单信息，类型分为`菜单`与`功能`，一个菜单下可以有多个功能，`菜单`类型的`permission`字段标识访问这个菜单需要的功能权限，`功能`类型的`permission`字段相当于此功能的别称，所以`菜单`类型的`permission`字段为其某个`功能`类型子节点的`permission`值
  * **permission**存储角色与功能的关联关系
  * **interface**存储接口信息
  * **functionInterface**存储功能与接口关联关系，通过查找用户所属角色，再查找相关角色所具备的功能权限，再通过相关功能就可以查出用户所能访问的接口
  * **route**存储前端路由信息，通过`permission`字段过滤出用户所能访问的路由  

# 系统菜单及路由相关

  系统菜单 和 路由是两个不同的概念，虽然说有些系统被设计成一体，但是本系统为了更好的扩展，将其解耦。菜单可能是三级甚至五级，但是路由往往就两级。
  
  系统的菜单和路由 都分为两部分组成，第一部分为固定的，所有使用者都可以使用的内容。其菜单部分在：src/menu 中，路由部分 在 src/router/routes 中.
  菜单的形式为：
  ~~~javascript
   { path: '/index', title: '首页', icon: 'home' },
    {
      path: '/demo',
      title: '演示页面',
      icon: 'folder-o',
      children: [
        { path: '/demo/page1', title: '页面 1' },
        { path: '/demo/page2', title: '页面 2' },
        { path: '/demo/page3', title: '页面 3' },
        {
          title: 'page4',
          children: [
            { path: '/demo/page4', title: 'page4' }
          ]
        }
      ]
    }
  ~~~
  
  路由的形式为：
  
  ~~~javascript
  [
    {
      path: '/',
      redirect: { name: 'index' },
      component: layoutHeaderAside,
      children: [
        // 首页
        {
          path: 'index',
          name: 'index',
          meta: {
            auth: true
          },
          component: _import('system/index')
        },
        // 演示页面
        {
          path: 'page1',
          name: 'page1',
          meta: {
            title: '页面 1',
            auth: true
          },
          component: _import('demo/page1')
        },
        // 系统 前端日志
        {
          path: 'log',
          name: 'log',
          meta: {
            title: '前端日志',
            auth: true
          },
          component: _import('system/log')
        },
        // 刷新页面 必须保留
        {
          path: 'refresh',
          name: 'refresh',
          hidden: true,
          component: _import('system/function/refresh')
        },
        // 页面重定向 必须保留
        {
          path: 'redirect/:route*',
          name: 'redirect',
          hidden: true,
          component: _import('system/function/redirect')
        }
      ]
    }
  ]
  ~~~
  
  这里菜单又分为 顶部菜单:header 和 侧边菜单: aside

  后端需要返回的权限信息包括权限过滤后的角色编码集合，功能编码集合，接口信息集合，菜单列表，路由列表，以及是否系统管理员标识。格式如下

  ~~~javascript
  accessMenus: [
        {
          'title': '测试页11',
          'path': '/demo',
          'icon': 'cogs',
          'children': [
            {
              'title': '页面22',
              'icon': 'cogs',
              'path': '/demo/page2'
            },
            {
              'title': '页面33',
              'icon': 'cogs',
              'path': '/demo/page3'
            }
  
          ]
        }
      ],
      accessRoutes: [
        {
          'name': 'System',
          'path': '/demo',
          'component': 'layoutHeaderAside',
          'componentPath': 'layout/header-aside/layout',
          'meta': {
            'title': '测试页面',
            'cache': true
          },
          'children': [
            {
              'name': 'page2',
              'path': '/demo/page2',
              'component': 'page2',
              'componentPath': 'demo/page2',
              'meta': {
                'title': '页面2',
                'cache': true
              }
            },
            {
              'name': 'page3',
              'path': '/demo/page3',
              'component': 'page3',
              'componentPath': 'demo/page3',
              'meta': {
                'title': '页面3',
                'cache': true
              }
            }
          ]
        }
      ]
  ~~~

#### 设置菜单
  
  每个路由末端都会有对应的组件（页面），他们的对应关系 在 routerMapComponents 里面体现，通过路由的 component 属性来匹配载入，routerMapComponents的形式如下：
  ~~~javascript
    let componentMaps = {
        "layoutHeaderAside": layoutHeaderAside,
        "menu": () => import(/* webpackChunkName: "menu" */'@/pages/sys/menu'),
        "route": () => import(/* webpackChunkName: "route" */'@/pages/sys/route'),
        "role": () => import(/* webpackChunkName: "role" */'@/pages/sys/role'),
        "user": () => import(/* webpackChunkName: "user" */'@/pages/sys/user'),
        "interface": () => import(/* webpackChunkName: "interface" */'@/pages/sys/interface')
    }
  ~~~
  
  每一个动态载入页面必须要以以上形式注册，这样才能让路由正确载入.

  配置好友，系统会在路由拦截（router.beforeEach）部分，将本地 菜单（路由）和动态加入的菜单 (路由) 合并，然后更新到UI里面
  
```javascript
  let allMenuAside = [...menuAside, ...permissionMenu]
    let allMenuHeader = [...menuHeader, ...permissionMenu]
  
    formatRoutes(permissionRouter)
    console.log('动态添加路由', permissionRouter)
    router.addRoutes(permissionRouter)
    // 处理路由 得到每一级的路由设置
    store.commit('d2admin/page/init', [...frameInRoutes, ...permissionRouter])
    // 设置顶栏菜单
    store.commit('d2admin/menu/headerSet', allMenuHeader)
    // 设置侧边栏菜单
    store.commit('d2admin/menu/asideSet', allMenuAside)
    // 初始化菜单搜索功能
    store.commit('d2admin/search/init', allMenuHeader)
    // 设置权限信息
    // store.commit('d2admin/permission/set', permission)
    // 加载上次退出时的多页列表
    store.dispatch('d2admin/page/openedLoad')
```

> 注意： 路由 和 菜单 的path 必须对应才能正确跳转

### 页面元素权限控制
 
  使用指令v-permission：

  ~~~javascript
  <el-button
      v-permission:function.all="['p_menu_edit']"
      type="primary"
      icon="el-icon-edit"
      size="mini"
      @click="batchEdit"
      >批量编辑</el-button>
  ~~~
  
  参数可为function、role，表明以功能编码或角色编码进行校验，为空则使用两者进行校验。
  
  修饰符all，表示必须全部匹配指令值中所有的编码。
  
  源码位置在plugin/permission/index.js，根据自己实际需求进行修改。
  
  使用v-if+全局方法
  
  ~~~javascript
  <el-button
      v-if="canAdd"
      type="primary"
      icon="el-icon-circle-plus-outline"
      size="mini"
      @click="add"
      >添加</el-button>
  ~~~

 ~~~javascript
  data() {
      return {
        canAdd: this.hasPermissions(["p_menu_edit"])
      };
    },
  ~~~
  
  默认同时使用角色编码与功能编码进行校验，有一项匹配即可。
  
  类似的方法还要hasFunctions，hasRoles。
  
  源码位置在plugin/permission/index.js，根据自己实际需求进行修改。
  
  >不要使用v-if="hasPermissions(['p_menu_edit'])"这种方式，会导致方法多次执行
  
  也可以直接在组件中从vuex store读取权限信息进行校验。
  
  
  ### 总结
  
  1、页面级别的组件放到pages/目录下，并且在routerMapCompnonents/index.js中以key-value的形式导出
  
  2、不需要权限控制的固定菜单放到menu/aside.js和menu/header.js中
  
  3、不需要权限控制的路由放到router/routes.js frameIn内
  
  4、需要权限控制的菜单与路由通过界面的管理功能进行添加，确保菜单的path与路由的path相对应，路由的name与页面组件的name一致才能使keep-alive生效，路由的component在routerMapCompnonents/index.js中能通过key匹配到。
  
  5、开发阶段菜单与路由的添加可由开发人员自行维护，并维护一份清单，上线后将清单交给相关的人去维护即可。

    
