# *⚙ 页面容器组件* （d2-container）

!> d2-container是构建页面最主要也是最基本的组件

![logo](https://doc.d2admin.fairyever.com/assets/img/space-main@2x.dddc5ae6.png)

## 使用参数

| 参数名 | 介绍 | 必选 | 值类型 | 可选 | 默认 |
| :-----| :------- | :----: | :----: |:----: |:----: |
| type | 容器模式 | 否 |String | full card ghost |full |
| better-scroll | 自定义滚动条配置 | 否 |Boolean | |false |
| better-scroll-options | 使用自定义滚动条 | 否 |Object |better-scroll |见下 |
| scroll-delay | scroll 事件的节流间隔(ms) 只在 better-scroll: false 时有效 | 否 |Number | |10 |
| filename | 当前文件在项目中的路径，更多介绍见下 | 否 |String | | |

?> 页面有较多内容时会在侧面产生滚动条

## header 和 footer

支持 header 和 footer 插槽，两个区域分别会固定在主体区域的顶部和底部，内容压缩至中间

~~~vue
  <template>
    <d2-container>
      <template slot="header">Header</template>
      内容
      <template slot="footer">Footer</template>
    </d2-container>
  </template>
~~~

?> 如果使用插槽后内容超出一屏，滚动条会出现在 header 和 footer 之间

## 自定义滚动条

设置 better-scroll 属性可以启用自定义滚动条，自定义滚动条在内容不满一屏时不会显示
~~~vue
  <template>
    <d2-container better-scroll>
      内容
    </d2-container>
  </template>
~~~

## 模式: card

卡片模式适用于简单的小页面，当内容不够填满时候，只显示内容区域：
~~~vue
  <template>
    <d2-container type="card">
        内容
    </d2-container>
  </template>
~~~

## 模式: full

full 模式会生成一个无论内容多少，都会填满主区域的页面容器。
~~~vue
  <template>
    <d2-container>
        内容
    </d2-container>
  </template>
~~~

## 模式: ghost

ghost 模式适合对页面有定制需求的用户，此模式生成一个没有背景颜色的页面区域
~~~vue
  <template>
    <d2-container type="ghost">
        内容
    </d2-container>
  </template>
~~~

!> 该模式默认没有内边距，示例中的内边距样式需要自行添加，比如可以在组件内嵌添加一层带有 .d2-pt 和 .d2-pb class 的 div，像下面这样

~~~vue
  <template>
    <d2-container type="ghost">
        <div class="d2-pt d2-pb">
          内容
        </div>
      </d2-container>
  </template>
~~~

## scroll事件

发生滚动时触发。

参数：Object
| 参数名 | 介绍 | 必选 | 类型 | 默认值 |
| :-----:| :----: | :----: |
| x | 横向距离 | 是 | Number | 0 |
| y | 纵向距离 | 是 | Number | 0 |

示列
~~~vue
  <template>
    <d2-container @scroll="({x, y}) => { scrollTop = y }">
      {{scrollTop}}
    </d2-container>
  </template>
  
  <script>
  export default {
    data () {
      return {
        scrollTop: 0
      }
    }
  }
  </script>
~~~

## scrollToTop方法

滚动到顶部触发的方法

~~~vue
  <template>
    <d2-container ref="container">
      <el-button @click="$refs.container.scrollToTop">
        返回顶部
      </el-button>
    </d2-container>
  </template>
~~~

## scrollTo方法
滚动到指定位置

?> 示例
~~~vue
  <template>
    <d2-container ref="container">
      <el-button @click="$refs.container.scrollToTop">
        返回顶部
      </el-button>
    </d2-container>
  </template>
~~~

?> 如果使用了滚动优化模式示例
~~~vue
 <template>
   <d2-container ref="container" better-scroll>
     <el-button @click="$refs.container.scrollTo(0, 30, 300)">
       滚动到(0, 30)像素位置 | 动画时长 300 ms
     </el-button>
   </d2-container>
 </template>
~~~
