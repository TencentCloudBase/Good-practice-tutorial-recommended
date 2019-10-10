## 背景

当你刚上大学的时候，要想不迷失校园，除了依靠不怎么可靠的路边标识外，总会收到那么一张卡通绘制的校园地图：

![QRcode](https://656e-enanyuan-6db383-1257936504.tcb.qcloud.la/showcase/WeChat%20Image_20190928225117.jpg?sign=12681c0289d537a05af976542a18a49a&t=1569682386)

这种静态图片可以让我们快速地了解到所需的地理位置信息，但使用和思考过后，会发现以下问题：

1. 地理位置信息粒度高，而同一个地点通常具有多个服务功能和别名。
2. 地理位置信息变更导致地图信息过时。一旦服务网点迁移或更名，需要重绘地图，带来一定的延迟和信息滞后。
3. 入口较深。存储在手机上的地图并不是那么好找，尤其是随着时间的推移。
4. 无法准确的定位当前所处位置，需要寻找参照物，这是静态地图致命的缺点。
5. 缺乏更为详细的地点介绍，只能在有限的画面里堆积内容。

为此，我设计了一款校园导览应用，用当下流行的微信小程序结合云开发能力，低成本高效能地解决了以上问题。此外，我还根据对市面上的同类应用进行设计上的研究，在界面和交互设计上做功夫。下面我会进行简短的介绍。

## 南苑导览

<div align=center>

![QRcode](https://656e-enanyuan-6db383-1257936504.tcb.qcloud.la/showcase/QRcode.jpg?sign=2f3aa5f8f9a8edc282ff67a20fb6deae&t=1566624511)

</div>

南苑导览是一款由学生独立开发的以地图为载体，提供**中山大学南方学院**（南苑）具体地点的位置信息、导航、校园历史及文化介绍的小程序。旨在解决校园导航标识不到位、地图形式低效单一、信息设计不够好等问题，为来南苑新人和游客提供更加完美的出行体验。

<div align=center>

**仅需修改地图配置文件，即可适配任意场景（校园、景区）的小程序个性化地图定制。**

技术栈：原生小程序 + TypeScript + gulp + vantUI + 云开发能力

2019 微信小程序高校大赛 · 华南赛区二等奖

</div>

## 南苑导览 · 开发

1. config 配置

```js
├─ src
├─── config
├───── index.ts // 入口
├───── cloud.ts // 云开发相关配置
├───── info.ts // 应用介绍信息
├───── markerStyle.ts // 地图marker样式
├───── panorama.ts // 第三方全景地图配置（个人类型无webview权限，默认关闭）
└───── secret.ts // 腾讯地图key等敏感信息（可选）
```

2. 使用云数据库

```js
// markers表 数据格式
{
  _id: "5ce8fe1c29c7a8581bc1e989",  // id，云数据库录入upsert更新用
  type: "生活服务",   // 场景名称
  icon: "shfw",     // marker默认图标，为场景名称拼音缩写
  scale: 15.0,   // 场景在地图上的缩放值，可选。已废弃，用includePoints代替
  position: 0, // 指定在各个场景中的排列顺序
  data: [   // 该场景下的地点markers
    {
      name: "孙中山铜像",   // 地点名称
      short_name: "铜像", // 名称缩写
      desc: "中山铜像...", // 描述信息
      logo: "tx",   // 地点logo，缩写拼音, 如作各院系logo展示
      icon: "tx@2",   // 自定义marker图标，“@”后数字为图标相较于默认图标的缩放值
      images: 3,  // 图片数量，作云存储拼接路径用（cloud://cloudRoot/1教/n.jpg）
      panorama: 0,  // 全景场景id
      latitude: "23.635875",  // 经度
      longitude: "113.678965",  // 纬度
      contact: { phone: "020-123456", address: "出门左转" }   // 联系方式
    }
  ]
}
```

使用 excel 进行数据维护，通过 python pandas 进行数据清洗，使用 jsonlines 库输出符合云数据库的 JSON Lines 格式文件，以 upsert 形式导入数据库。

数据更新流程如下：
![flow](https://656e-enanyuan-6db383-1257936504.tcb.qcloud.la/showcase/flow.svg?sign=abe8b922388e9920ea89b81f64efc20c&t=1566635619)

2. 加载并清洗数据
   使用 request 或云数据库进行异步数据请求时。由于 app.js 中的 onLaunch 和首页 index 的 onLoad 的执行顺序不是固定的，所以如果首页有基于 app.js 请求的数据时要注意生命周期的问题。

```javascript
// index
async loadMarkers() {
  let markers;
  if (app.globalData.config.debug) {
    // 本地
    markers = mockMarkers;
  } else {
    // 云
    await wx.cloud
      .callFunction({
        name: "loadMarkers"
      })
      .then((res: any) => {
        markers = res.result.data;
      });
  }
  app.globalData.markers = markers;
}

clearMarkers(markers: any[]) {
  let num = 0;  // 每个marker都要有一个id
  for (const i of markers) {
    for (const j of i.data) {
      j.id = num;
      num += 1;
      j.iconPath = `/assets/images/markers/${j.icon ? j.icon : i.icon}.png`;

      ...

      // 自定义气泡样式
      j.callout = Object.assign(
        { content: j.short_name ? j.short_name : j.name },
        app.globalData.config.markerStyle.calloutStyle
      );
    }
  }
  return markers;
}
```

3. 巧用 MapContext  
   你不需要去手动地为每个场景设置 scale，用 includePoints 即可让地图视野自动覆盖到当前所有 POI。  
   你也不需要去手动地去获取权限设置用户位置，用 moveToLocation 即可轻松定位。

```javascript
// index
onReady() {
  this.setData!({
    mapContext: wx.createMapContext("map")
  });
}

includePoints(padding: number) {
  this.data.mapContext.includePoints({
    padding: [padding, padding, padding, padding],
    points: this.data.markers
  });
}

locate() {
  this.data.mapContext.moveToLocation();
}
```

4. 使用云存储管理图片  
   添加新图片时，直接修改 images 字段即可，文件夹目录为地点名称。

```html
<!-- 地点详情页 轮播图 -->
<swiper
  indicator-dots="{{imgUrls.length > 1}}"
  autoplay="{{true}}"
  interval="3000"
  duration="1000"
>
  <block wx:for="{{imgUrls}}" wx:key="{{index}}">
    <swiper-item>
      <image
        src="{{item}}"
        class="slide"
        data-id="{{index}}"
        bindtap="previewImage"
      />
    </swiper-item>
  </block>
</swiper>
```

```JavaScript
for (let i = 0; i < marker.images; i++) {
  imgUrls.push(
    this.data.cloudRoot +
      "images/" +
      (marker.short_name || marker.name) +
      "/" +
      i +
      ".jpg"
  );
}
```

## 南苑导览 · 设计

如果你在微信上搜索「导览」二字，看到的小程序大多都是一个模板，页面层级深，界面拥挤，列表式的信息展示并不符合我们日常使用地图 APP 的经验。为此，我做出了多项改良：

![](https://656e-enanyuan-6db383-1257936504.tcb.qcloud.la/showcase/index2.gif?sign=9d0ba8a675ad9d5b30af67295f4639aa&t=1569726789)

1. 更好的视野 - 自定义导航栏与侧边栏  
   因为只有特定的页面需要使用自定义导航栏，所以只需要设置页面级的 config：

```json
  "navigationStyle": "custom"
```

接下来获取胶囊按钮位置信息：

```js
bounding: wx.getMenuButtonBoundingClientRect();
```

![getMenuButtonBoundingClientRect](https://656e-enanyuan-6db383-1257936504.tcb.qcloud.la/showcase/5f551548036b036f1a4e8b7414265e48.jpg?sign=a8159d1cf46374386dd09cdb769bca3b&t=1569690379)

动态地设置样式：

```html
<!-- SIDE MENU -->
<view
  class="sidebar"
  hidden="{{toggleRoutes}}"
  style="top:{{bounding.height + bounding.top + 10}}px"
>
  ...
</view>
```

2. FAB 与侧边栏设计

把最主要的定位、搜索和路线推荐功能在视觉上成为整体，通过点击 FAB 弹出菜单选项。侧边栏的地点场景菜单设计为下拉滚动，注意使用半遮设计来提醒用户滚动。同时，为了让界面更加精简，侧边菜单会在点击 FAB（Float Action Button）和母按钮时 toggle 显示与隐藏。

![](https://656e-enanyuan-6db383-1257936504.tcb.qcloud.la/showcase/index.gif?sign=2dc28e38c0060b820bb12dae1db18ef6&t=1569726719)

1. 用点击代替滚动 - scroll-into-view  
   在路线面板和搜索页中，使用到了 scroll-view 组件，利用其 scroll-into-view 特性，实现点击代替滚动的操作，同时也能起到提醒后置选项的作用。

![](https://656e-enanyuan-6db383-1257936504.tcb.qcloud.la/showcase/index3.gif?sign=dd76455dff713d01c05fb1e54aba8db9&t=1569726817)

```javascript
windowWidth: wx.getSystemInfoSync().screenWidth;
```

```html
<scroll-view class="route" scroll-x scroll-into-view="{{focusPointId}}">
  <view
    class="points"
    style="width:{{routes[routeIndex].count * 140 < windowWidth ? windowWidth : routes[routeIndex].count * 140}}rpx;"
  >
    ...
  </view>
</scroll-view>
```

3. 更好的视角 - 全景功能  
   结合 web-view 和全景服务平台，可以为一款地图导览应用增色不少。

![](https://656e-enanyuan-6db383-1257936504.tcb.qcloud.la/showcase/ezgif-4-e2679b1a3106.gif?sign=6bceb316e44535b7e9bffcca7e234d24&t=1569729322)

## 总结

云开发让小程序开发者无需搭建服务器，使用平台提供的 API 即可快速地进行业务开发、上线和迭代，免费的基础版完全可以满足中小应用的需求。「南苑导览」借助腾讯云开发能力，上线以来，帮助到了许许多多的新生和来客，实现了产品价值。最后，期望官方早日开放自定义地图底图能力，让开发者能够个性化地图，探索出更多的应用场景！

## 源码链接
[https://github.com/Observer-L/NFU-Guide-Map](https://github.com/Observer-L/NFU-Guide-Map)
