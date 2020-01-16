# Vue2.x高效还原美团外卖App

## 介绍

这是一个利用vue+express仿美团外卖app的项目。需要有[vue](https://cn.vuejs.org/)和[es6](http://es6.ruanyifeng.com/)的基础，[express](http://expressjs.com/zh-cn/)会一点点就行。

本代码来源于腾讯课堂米斯特吴的课程，如有需要，请[购买课程](https://ke.qq.com/course/464832#term_id=100556268)

## 最终效果展示

先上最终效果图：
![](../../data/posts/20200115_vue_app/display.gif)

## 初始化框架

这是Vue CLI 2.X的创建方式，建议用最新的 [Vue CLI](https://cli.vuejs.org/zh/guide/creating-a-project.html)

`vue init webpack mt-app`

## mock数据及配置路由

### SVG转成图标

[将SVG矢量图转成图标](https://icomoon.io/app/#/select)

将下载的 `style.css` 放在 `mt-app/src/common/css/icon.css` 中  
`fonts` 文件夹拷贝到 `mt-app/src/common` 下

### mock数据

在 `mt-app/build/webpack.dev.conf.js` 中加载data数据

```js
cd mt-app
npm install
npm run dev
```

访问以下网址可查看mock的数据  
[localhost:8080/api/goods](localhost:8080/api/goods)  
[localhost:8080/api/ratings](localhost:8080/api/ratings)  
[localhost:8080/api/seller](localhost:8080/api/seller)  

### css样式重置

[css样式重置](https://meyerweb.com/eric/tools/css/reset/)

```bash
mkdir static/css
touch static/css/reset.css
```

在 `mt-app/index.html` 中添加 `<link rel="stylesheet" href="static/css/reset.css">`

### 配置路由

在 `mt-app/src/components` 中添加5个组件
``` bash
mkdir src/components/header src/components/nav src/components/goods src/components/ratings src/components/seller
touch src/components/header/Header.vue
touch src/components/nav/Nav.vue
touch src/components/goods/Goods.vue
touch src/components/ratings/Ratings.vue
touch src/components/seller/Seller.vue
```

在 `mt-app/src/App.vue` 中引入 `Header` 和 `Nav`，并注册组件  
`npm install vue-router --save`  
在 `mt-app/src/main.js` 中
```js
import VueRouter from 'vue-router';
Vue.use(VueRouter)
```

实例化组件
```js
import Goods from '@/components/goods/Goods';
import Ratings from '@/components/ratings/Ratings';
import Seller from '@/components/seller/Seller';
```

创建Routes
```js
const routes = [
  {path:"/",redirect:"/goods"}, // 重定向
  {path:"/goods",component:Goods},
  {path:"/ratings",component:Ratings},
  {path:"/seller",component:Seller}
]
```

实例化router
```js
const router = new VueRouter({
  routes
})
```

挂载实例
```js
new Vue({
  el: '#app',
  router,
  components: { App },
  template: '<App/>'
})
```

为了实现点击可跳转，设置tab页选中的样式，设置tab的下划线。
需要在 `mt-app/src/main.js` 中添加属性 `linkActiveClass`
```js
const router = new VueRouter({
  routes,
  linkActiveClass:"active" // 点击选项卡颜色变化
})
```
代码见 `mt-app/src/components/nav/Nav.vue`

## 头部结构和样式设计

引入下载好的css `@import url(../../common/css/icon.css);`  
此时会报错：找不到font，需要修改 `icon.css`，在5个 `fonts` 前加上 `../` 即可

### 顶部通栏
响应式导航，搜索框的宽度占100%，左右两侧用padding撑开，放置按钮
1. 向左的返回箭头
2. 搜索框
3. 拼单 & 更多

### 主题内容
1. 麦当劳图标
2. 餐厅名称
3. 五角星图片 & 收藏

### 公告内容 
1. "首"字
2. 公告信息
3. 有?个活动 & 右箭头，点击可展开【公告详情】

### 背景内容
1. 背景图片
2. 利用computed返回 `background-image`

### 公告详情
1. 过渡动画 `transition`
2. 蒙版，用 `v-show` 来控制蒙版的显示/隐藏
3. 相关内容容器
     + M图片
     + 餐厅名称
     + 星级评价，引入了 `app-star` 组件
     + 起送、配送、30分钟
     + 配送时间
     + 新用户折扣信息
4. 关闭内容容器

代码见 `mt-app/src/components/header/Header.vue` 和 `mt-app/src/components/star/Star.vue`

## 点菜页面设计(核心功能)

分类列表为左侧的导航栏，商品列表为右侧的食物列表。分类列表和商品列表均由**专场**和**具体分类**组成

### 分类列表

1. 整体布局采用 `flex`  
2. 高度设置原则：距离顶部190px，底部51px，超出的部分hidden  
3. 左侧导航，宽度固定为85px，菜单名字超过2行部分，用...代替
```css
-webkit-line-clamp: 2;
display: -webkit-box;
-webkit-box-orient: vertical;
overflow: hidden;
```
4. 右侧食物的宽度，会随着屏幕宽度自适应拉伸  
5. 设置icon大小

### 商品列表

滚动组件：[better-scroll](https://github.com/ustbhuangyi/better-scroll)

`npm install better-scroll --save`

```js
import BScroll from 'better-scroll'
// 实例化滚动容器，fetch数据后，执行滚动方法
this.menuScroll = new BScroll(this.$refs.menuScroll)
this.foodScroll = new BScroll(this.$refs.foodScroll,{
  probeType:3, // 不仅在屏幕滑动的过程中，而且在momentum滚动动画运行过程中实时派发scroll事件
  click:true // better-scroll默认会阻止浏览器的原生click事件。当设置为true，better-scroll会派发一个click事件，激活+/-组件的点击事件
})
// foodScroll 监听事件
this.foodScroll.on("scroll",(pos) => {
  this.scrollY = Math.abs(Math.round(pos.y))
})
```

右侧滚动联动左侧菜单, 需在 `this.$nextTick()` 中执行
1. 计算分类的区间高度
2. 监听滚动的位置,注意[scroll](https://ustbhuangyi.github.io/better-scroll/doc/zh-hans/events.html#scroll)事件中的 `probeType`以及[click](https://ustbhuangyi.github.io/better-scroll/doc/zh-hans/options.html#click)事件
3. 根据滚动位置确认下标,与左侧对应(类似轮播图),见 `currentIndex` 属性
4. 通过下标实现点击左侧,滚动右侧(类似轮播图),见 `@click="selectMenu()"` 方法,重点关注[scrollToElement](https://ustbhuangyi.github.io/better-scroll/doc/zh-hans/api.html#scrolltoelementel-time-offsetx-offsety-easing)方法

代码见 `mt-app/src/components/goods/Goods.vue`

## 购物车(核心功能)

### 创建购物车组件

```bash
# 创建购物车组件
mkdir src/components/shopcart
touch src/components/shopcart/Shopcart.vue
```

购物车，使用flex布局，底部左侧 `content-left` 随着屏幕宽度拉伸，底部右侧 `content-right`固定宽度110px，数据从 `Goods.vue` 传递给 `Shopcart.vue`

**底部左侧**：  
购物车icon & 另需配送费，需引入图标库 `@import url(../../common/css/icon.css);`   
样式有两种：购物车为空(灰色)/不为空(黄色)  

**底部右侧**：  
0元起送/去结算

### 创建＋/－组件

```bash
# 创建+/-组件
mkdir src/components/cartcontrol
touch src/components/cartcontrol/CartControl.vue
```

在 `Goods.vue` 中注册 `app-cart-control` 组件，并进行父子组件传值。  
因为 `better-scroll` 默认会阻止浏览器的原生click事件，所以为了激活+/-组件的点击事件，必须给`this.foodScroll` 显示的指定 `click:true`。  
`+号` 底色为黑色，`-号` 底色为灰色。  
当 `count` 属性不存在时，利用 `Vue.set(this.food,"count",1)` 将count添加到 `this.food` 这个对象中，而且是响应式的。  
当购物数量减为0时，通过 `v-show` 让 `-号` 和 `数量` 消失，通过 `transition` 让-号产生滚动的动画。

### 让＋/－组件与购物车联动

联动通过 `Goods.vue` 中的 `selectFoods` 计算属性来实现。再利用父子组件传值的方式，将 `selectFoods` 传递给购物车组件 `app-shopcart`，并计算出加入购物车的总个数 `totalCount` 以及加入购物车的总价格 `totalPrice`，总个数和总价格分别用 `v-show="totalCount"` 和 `v-show="totalPrice"` 控制显示/隐藏。

### 让＋/－组件与左侧导航栏联动

当购物车数量增减后，想让左侧导航栏右上角的数字产生联动效果，需修改`Goods.vue` 中的分类列表——除【专场】外的导航——`calculateCount(item.spus)`。此时，导航栏右上角会显示的单个分类的购物车总数量。父级需加上相对定位，不然影响导航栏右上角的数字的显示位置，也就是 `menu-item` 的样式需要加上 `position: relative;`

代码见 `mt-app/src/components/goods/Goods.vue` 和 `mt-app/src/components/shopcart/Shopcart.vue` 和 `mt-app/src/components/cartcontrol/CartControl.vue`

## 购物车列表(核心功能)

点击购物车，弹框显示购物列表。用 `@click="toggleList"` 和 `v-show="listShow"` 控制购物车列表的显示/隐藏。点击购物车时，购物车列表上方会出现一个蒙版，点击蒙版，会隐藏购物车列表。给父级加上 `class="shopcart"`，蒙版的样式 `class="shopcart-mask"`, 通过 `v-show="listShow"` 和 `@click="hideMask"` 控制蒙版的显示/隐藏。蒙版的 `z-index: 98;`, `shopcart-wrapper` 的 `z-index: 99;`

在 `Shopcart.vue` 中添加【购物车列表】即可，一共由4个部分组成  
1. 顶部 `list-top`，包括新用户优惠详情，用 `v-if="poiInfo.discounts2"` 控制显示/隐藏
2. 头部 `list-header`，包括1号口袋和清空购物车。清空购物车可通过 `@click="clearAll"` 实现。
3. 内容 `list-content`，包括左侧的商品名称、单位/描述(2选1显示)、价格以及右侧的+/-号组件 `CartControl.vue`。使用 `<app-cart-control :food="food"></app-cart-control>` 传参。 商品名称、单位/描述(2选1显示)、价格可通过遍历 `selectFoods` 拿到数据。其中 `transform: translateY(-100%);` 表示内容的高度=列表撑开的高度，但是列表的最大高度为360px，超出的部分用 `better-scroll` 滚动显示。
4. 底部 `list-bottom`，这个好像没用到，注释也没啥影响

代码见 `mt-app/src/components/shopcart/Shopcart.vue`

## 商品详情页面(核心功能)

点击单个食物，会跳转到商品详情页面

在 `Goods.vue` ——具体的商品列表中添加点击事件 `@click="showDetail(food)"`，再传递给 `<app-product-detail :food="selectFood" ref="foodView"></app-product-detail>` 组件。利用 `ref="foodView"`属性实现父级调用子级的方法。

```bash
mkdir /src/components/productDetail
touch /src/components/productDetail/ProductDetail.vue
```

商品详情页面的动画依旧使用了 `transition`

```html
<transition name="food-detail">
  <div class="food">
    <div class="food-wrapper">
      <div class="food-content">
        ……
      </div>
    </div>
  </div>
</transition>
```

页面结构：  
1. 4个图片：食物大图、关闭按钮、分享按钮、更多按钮
2. 商品名称、月售、网友推荐(用v-show控制显示/隐藏)、价格、计价单位。  
   如果已加入购物车(count>0)，就显示+/-号组件，否则显示【选规格】。【选规格】绑定了一个点击事件 `@click="addProduct"` 去加载+/-号组件，需利用 `Vue.set(this.food,"count",1)` 添加 `count` 属性。
3. 外卖评价

这里有两个坑要处理：  
1. 在父页面点击+/-号组件时，也会跳转到商品详情页面(此时你只是想加购物车，并不想跳转到商品详情页面)。因此，我们需要阻止当前的冒泡事件。需修改 `mt-app/src/components/cartcontrol/CartControl.vue` 中+号的点击事件为 `@click.stop.prevent="increaseCart"`。也就是点击+号之外的任何地方，都会跳转到商品详情页面，而点击+号只会加入购物车，不进行页面的跳转。关闭按钮添加一个点击事件 `@click="closeView"` 即可。
2. 网络请求的图片，需要在跳转到商品详情页面前，给img容器预留空间。当图片没有网络请求下来前，预留位置，请求下来后，将图片填充到预留位置上。否则底下的名称、价格、评价等会往上跑，布局会很丑。

```css
.food .food-wrapper .food-content .img-wrapper{
	position: relative;
	width: 100%;
    /* 会根据当前的宽度，撑开一个等宽的高度，形成一个正方形的容器 */
	height: 0; 
	padding-top: 100%;
}
```

代码见 `mt-app/src/components/goods/Goods.vue` 和 `mt-app/src/components/productDetail/ProductDetail.vue`

## 商品详情(评价列表)

商品详情页面中，当商品无评价时，显示空白。有评价时，显示相应的评价信息

待解决的问题：  
1. 评价过多，需滚动显示，在 `showView()` 中初始化 `better-scroll`，需设置 `ref="foodView"`
2. 评价和内容之间，需添加分割线。而分割线有很多地方要用，将分割线抽离成一个组件比较合适

```bash
# 新建分割线组件
mkdir src/components/split
touch src/components/split/Split.vue
```

在 `ProductDetail.vue` 中引入并注册组件后，可通过 `<Split />` 插入分割线  
外卖评价的结构，均需要加上 `v-if="food.rating"` 做判断，否则拿不到数据，因为数据嵌套层数过深(unbelievable)
```html
<div class="rating-wrapper">
  <!-- 评价头部 -->
  <div class="rating-title">
    <!-- 外卖评价 & 好评度 100% -->
    <div class="like-ratio">

    </div>
    <!-- ?条评论 & ＞符号 -->
    <div class="snd-title">
      
    </div>
  </div>
  <!-- 评价列表 -->
  <ul class="rating-content">
    <!-- 用户头像图片/默认头像图片 -->
    <!-- 用户id -->
    <!-- 评价时间 -->
    <!-- 评价内容 -->
  </ul>
</div>
```

代码见 `mt-app/src/components/productDetail/ProductDetail.vue` 和 `mt-app/src/components/split/Split.vue`，。

## 评价页面

评价的Tab页，标题上需显示数量。需要在进入app时，就获取到值。所以要在 `App.vue` 的 `created()` 中请求 `ratings`，再将 `commentNum` 传递给组件 `<app-nav :commentNum="commentNum"></app-nav>` 

### 评价页面的结构

```html
<div class="ratings-wrapper">
  <!-- 头部 -->
  <div class="overview">
    <div class="overview-left">
      <!-- 商家评分、口味、包装、五角星组件 -->
    </div>
    <div class="overview-right">
      <!-- 配送评分 -->
    </div>
  </div>

  <!-- 分割线 -->
  <Split />

  <!-- 评价内容 -->
  <div class="content">
    <div class="rating-select" v-if="ratings.tab">
      <!-- 全部/有图/点评，3个Tab切换 -->
    </div>
    
    <div class="labels-view">
      <!-- 标签 -->
    </div>
    
    <ul class="rating-list">
      <!-- 评价列表 -->
    </ul>
  </div>
</div>
```

五角星组件需要传递 `score` 值：`<Star :score="ratings.quality_score" class='star'></Star>`

### 3个Tab页

设置3个常量：  
const ALL = 2 // 全部  
const PICTURE = 1 // 有图  
const COMMENT = 0 // 点评  

对应的样式分别为：
```html
:class="{'active':selectType==2}"
:class="{'active':selectType==1}"
:class="{'active':selectType==0}"
```

通过 `selectTypeFn()` 方法来确定哪个Tab页的样式需要高亮。好聪明的做法~~~

当切换到【点评】Tab时，点评前的图片为黑色，否则为黄色。用 `v-show="selectType == 0` 控制样式

### 标签

只展示 `label_star > 0` 的标签，遍历 `labels` 数组即可。

### 评价列表

当全部/有图/点评3个Tab切换时，评价列表的内容也要跟着改变，通过计算属性 `selectComments` 来实现，过滤一下数据即可。
内容过多时，需滚动显示。还是熟悉的配方，还是熟悉的套路
```js
this.$nextTick(()=>{
  if(!this.scroll){
    this.scroll = new BScroll(this.$refs.ratingView,{
      click:true
    })
  }else{
    this.scroll.refresh()
  }
})
```

评价列表项跟 `ProductDetail.vue` 几乎一模一样，copy过来，改下遍历的数据源就行。  
评价时间戳需利用 `selectComments()` 转换成 `yyyy.MM.dd` 格式  
代码见 `mt-app/src/App.vue` 和 `mt-app/src/components/ratings/Ratings.vue`

## 商家页面

### 功能点

商家店铺图片，实现横向滚动
```js
this.$nextTick(() => {
  // 判断当前数组是否有图片
  if(this.seller.poi_env.thumbnails_url_list){
    // 获取单张图片的可视宽度
    let imgW = this.$refs.picsItem[0].clientWidth
    // margin-right
    let marginR = 11
    // 整个横向滚动条的宽度 = (单张图片的宽度 + margin-right的宽度) * 图片的张数
    let width = (imgW + marginR) * this.seller.poi_env.thumbnails_url_list.length

    this.$refs.picsList.style.width = width + "px"

    this.scroll = new BScroll(this.$refs.picsView,{
      // 横向滚动
      scrollX:true
    })
  }

  // 让整个商家页面，可以纵向滚动
  this.sellerView = new BScroll(this.$refs.sellerView)
})
```

### 商家页面的结构
```html
<div class="seller">
  <div class="seller-wrapper">
    <Split></Split>
    <div class="seller-view">
      <div class="address-wrapper">
        <!-- 地址 -->
      </div>
      
      <div class="pics-wrapper">
        <!-- 横向图片滚动，v-for遍历即可，注意有4个ref -->
      </div>
      
      <div class="safety-wrapper">
        <!-- 查看商品安全档案 -->
      </div>
    </div>

    <Split></Split>
    <div class="tip-wrapper">
      <!-- 配送服务 -->
      <!-- 配送时间 -->
    </div>

    <Split></Split>
    <div class="other-wrapper">
      <!-- 商家服务 -->
      <!-- 优惠信息 -->
    </div>
  </div>
</div>
```

## 路由优化和项目调试

优化点：
1. 在 `App.vue` 中添加 `keep-alive` (用于保留组件状态或避免重新渲染)，可避免每次tab页切换都去请求网络数据(F12——Network——刷新) & 切换Tab页后，在返回上一个Tab时，购物车数据已清空。相当于用户点菜后，看了一眼评价信息，就要重新加入购物车，重新点菜，这谁受得了！
```html
<!-- content -->
<keep-alive>
  <router-view></router-view>
</keep-alive>
```
2. 非模拟器调试时，点击左侧导航栏，右侧会联动，但是联动错位了。需修改 `Goods.vue` 中 `better-scroll` 的初始化条件
```js
this.menuScroll = new BScroll(this.$refs.menuScroll,{
  click:true
})
```
3. 非模拟器调试时，点击 `-号`，会弹开商品详情页面(本来应该只减少购物车的数量)，需修改 `CartControl.vue` 中 `-号` 的点击事件中，阻止冒泡事件，即 `@click.stop.prevent="decreaseCart"`
4. 模拟器测试时，点击左侧导航栏后，右侧没有联动，说明左侧导航栏没有添加点击事件。修改同第2点

## 真机调试

你可能需要了解一下：[移动端调试神器](https://juejin.im/post/5b72e1f66fb9a009d018fb94)

真机测试，要想获取远程的IP地址(在手机的浏览器中输入该IP地址可访问app)，需要将电脑和手机处于同一WiFi环境下。

MAC：系统偏好设置——网络——状态，就能看到一个IP地址，如：`192.168.0.102`  
此时你在浏览器输入`192.168.0.102:8081`，是不能直接访问的。

需要修改 `mt-app/config/index.js` 文件  
1. 将 `port` 改成 `8081` (因为利用weinre进行真机测试时端口号会冲突，weinre的默认端口为8080)
2. 将 `host` 改成 `0.0.0.0`
3. 修改config后，必须要重启项目
4. 此时在电脑和手机的浏览器中输入：`192.168.0.102:8081` 均可以看到app页面了

假设模拟器测试正常(浏览器可通过F12标记元素)，而真机测试发现有问题，你就需要借助移动端调试的工具了。以 `weinre` 举例

```bash
sudo npm install -g weinre
weinre --boundHost 192.168.0.102
# 回车会显示一个地址：http://192.168.0.102:8080
```

拷贝【Target Script】——【Example】底下的 `<script src="http://192.168.0.102:8080/target/target-script-min.js#anonymous"></script>` 至代码根目录的 `index.html` 文件的 `<head></head>` 中。远程调试后，记得删除该 `<script></script>`
   
刷新浏览器，点击【Access Points】——【debug client user interface】后的 `http://192.168.0.102:8080/client/#anonymous`，刷新手机，【Targets】底下就会出现内容了。此时你就可以在电脑上操控手机了，点击【Elements】可调试，跟浏览器F12的效果差不多。

## 项目打包并运行调试

打包项目前，先检查 `mt-app/config/index.js` 下 `build` 环境的参数，将`productionSourceMap` 改成 `false` ，避免打包的时候生成很多sourcemap的、协助我们调试的内容，毕竟很占空间。  
`npm run build` 后会生成一个 `dist` 文件夹。  
此时会有一个提示，打包的文件需要在服务环境下运行，直接打开 `index.html` 无效。
```bash
Tip: built files are meant to be served over an HTTP server. Opening index.html over file://won't work
```

在项目根目录创建一个 `app-server.js` 
```js
var express = require('express');
var port = 8088;

var app = express();

var router = express.Router();
router.get('/', function(req,res,next){
	req.url = '/index.html';
	next();
});

app.use(router);

// 接口数据

// 1、读取json数据
var goods = require("./data/goods.json")
var ratings = require("./data/ratings.json")
var seller = require("./data/seller.json")

// 2、路由
var routes = express.Router();

// 3、编写接口
routes.get('/goods', (req,res) => {
	res.json(goods);
});
routes.get('/ratings', (req,res) => {
	res.json(ratings);
});

routes.get('/seller', (req,res) => {
	res.json(seller);
});

app.use('/api',routes);

// 定义static目录，指向./dist目录
app.use(express.static('./dist'));

// 启动express
module.express = app.listen(port, function(err){
	if(err){
		console.log(err);
		return;
	}
	
	console.log('http://localhost:' + port + '/n');
});
```

运行 `node app-server.js`，回车后在浏览器打开 `http://localhost:8088` 进行调试
