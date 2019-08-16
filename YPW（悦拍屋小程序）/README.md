创意来源于生活，之所以开发这个校园约拍小程序，是因为在摄影选修课上常听老师抱怨外出写生老找不到模特，许多大学生都想拥有一套专属自己记忆的摄影作品，记录下不会磨灭的美好回忆，可如何找到让自己满意的摄影师是他们的难题。悦拍屋是一个校园摄影o2o的约拍平台，提供全方位的约拍服务，同时提供一个自我展示，学习交流，互动娱乐的平台。接下来我将结合项目的讲解给大家分享一些实用技术和对于云开发的一些经验，希望对正在学习小程序的你有帮助。

# 前言
在开发一个项目之前首先要进行技术选型从而降低产品开发的技术风险和提高开发效率，技术选型必须得紧紧围绕着业务场景来选择。

- 产品原型设计：墨刀
- UI组件库  
    1.微信原生样式库`WeUI`,让用户使用感知更加统一  
    2.注重视觉交互体验的`ColorUI`组件库，在感知统一的基础上视觉元素多样化
- 前端  
    1.小程序原生语法以及`API`  
    2.`Promise`实现异步调用  
    3.`ES6`编写页面交互逻辑
- 后端  
    1.云函数：无需自建服务器，在云端运行的代码，微信私有协议天然鉴权，开发者只需编写自身业务逻辑代码  
    2.云数据库：无需自建数据库，一个既可在小程序前端操作，也能在云函数中读写的 `JSON` 数据库  
    3.云存储：实现小程序前端直接上传/下载云端文件，在云开发控制台可视化管理  
    4.云调用：由原生微信服务集成，基于云函数免鉴权使用小程序开放接口的能力，包括服务端调用、获取开放数据等能力
- 其他  
    1.使用微信提供的云测试对未上线的小程序进行缺陷测试、性能数据分析、机型覆盖测试，确保小程序上线后正常运营  
    2.使用基于云开发的`AI视觉能力`-身份证识别实现实名认证，智能鉴黄结合人工完成发布信息的审核  
    3.开发工具:微信开发者工具、VScode  
    4.部分图标使用自阿里巴巴矢量图标库
# 总体设计
## 功能结构图
大家可以通过此图了解整个项目的主要功能点

![功能结构图](https://user-gold-cdn.xitu.io/2019/6/28/16b9d47f9ce9f0a2?w=1312&h=936&f=png&s=94563)

## 产品原型图
此处给出一张主页原型图示例，墨刀还是挺好用的

![主页原型图](https://user-gold-cdn.xitu.io/2019/6/28/16b9d4a362b38a95?w=332&h=591&f=png&s=44050)

## 色彩设计图
悦拍屋的整体色调为浅蓝色，各位小伙伴在开发自己项目的时候可以根据色彩标准搭配来设计项目所采用的色彩，合适的色彩搭配可以给用户良好的视觉体验

![色彩设计图](https://user-gold-cdn.xitu.io/2019/6/28/16b9d54317dd1896?w=1102&h=318&f=png&s=30919)

# 功能模块详解
接下来我会对部分功能模块以图文结合的形式详细描述，将其中涉及的技术、知识分享给大家
## 约拍邀请
用户可在首页查看约拍需求，并点击查看需求详情，用户在了解需求后，若自己符合条件即可提交约拍信息，等待发布者的回复，可将此需求收藏方便查看

![](https://user-gold-cdn.xitu.io/2019/6/28/16b9e475ac6879b1?w=397&h=700&f=gif&s=2054353)

### 技术分享：自定义顶部导航栏  
官方默认的导航栏只能对背景颜色进行更改，对于想要在导航栏添加一些比较酷炫的效果则需要通过自定义导航栏实现 

实现原理：通过设置`app.json`中页面配置的`navigationStyle`(导航栏样式)配置项的值为`custom`,即可实现自定义导航
```app.json
"window":{
    "navigationStyle":"custom"
}
```
本项目的部分页面自定义导航栏实现使用了`ColorUI`的导航栏组件，在完成上一步属性设置后再引入导航栏组件即可
```app.json
"usingComponents":{
    "cu-custom":"/colorui/components/cu-custom"  //该路径替换为自己项目内ColorUI组件所在位置
}
```
主页自定义导航栏通过设置背景图片加上GIF波浪效果
```index.html
  <view class='page__bd'>
    <view class="bg-img padding-tb-xl" style="background-image:url('http://wx4.sinaimg.cn/mw690/006UdlVNgy1g2v2t1ih8jj31hc0p0qej.jpg');background-size:cover;">
      <view class="cu-bar">
        <view class="content text-bold text-white">
          悦拍屋
        </view>
      </view>
    </view>
    <view class="shadow-blur">
      <image src="https://image.weilanwl.com/gif/wave.gif" mode="scaleToFill" class="gif-black response" style="height:100rpx;margin-top:-100rpx;"></image>
    </view>
  </view>
```
效果图

![](https://user-gold-cdn.xitu.io/2019/6/28/16b9e85af5e5db08?w=397&h=120&f=gif&s=150182)

使用组件定义的导航栏
```
<cu-custom bgImage="https://s2.ax1x.com/2019/05/02/Etiyng.jpg" isBack="{{true}}">
  <view slot="backText">返回</view>
  <view slot="content">认证信息说明
  </view>
</cu-custom>
```
效果图

![](https://user-gold-cdn.xitu.io/2019/6/28/16b9e88d755efb48?w=391&h=50&f=png&s=23342)

```!
特别提醒1：使用自定义导航后，页面的返回需要在自定义导航栏中自行设置
```
```!
特别提醒2：导航栏组件需要自行引入ColorUI组件库后才能使用，具体引入教程地址在附录中给出
```
## 发布约拍
选择发布约拍功能填写约拍需求，提交审核通过后可在首页实时查看发布结果
![发布约拍](https://user-gold-cdn.xitu.io/2019/6/29/16ba0fc5309b428e?w=397&h=697&f=gif&s=1176370)
### 技术分享：入场动画  
额。。录制可能略微有点卡顿，实际效果挺流畅的，各位大佬有什么好的录制工具推荐可以在评论中回复 

实现原理：通过`toggleDelay`的布尔值为真动态添加动画类名，在生命周期函数`onReady`中控制`toggleDelay`的值从而控制整个动画过程(原理与`Vue`的动态类名相似)
```javascript
data:{
    toggleDelay；false
},
onReady:function(){
    let that = this
    //toggleDelay的值为真，动画开始
    that.setData({
      toggleDelay: true
    })
    //控制整个动画的时长
    setTimeout(function() {
      that.setData({
        toggleDelay: false
      })
    }, 2000)
}
```
```wxml
<view class="padding-xs {{toggleDelay?'animation-slide-bottom':''}}" style="animation-delay: {{item.time}}s;" wx:for="{{list}}" wx:key="{{index}}">
  <image class="img" id='img{{index}}' src="{{item.src}}" mode="widthFix" />
</view>
```
```wxss
//所有动画的定义
[class*=animation-] {
    animation-duration: .5s;
    animation-timing-function: ease-out;
    animation-fill-mode: both
}
//animatioon-slide-bottom所定义的动画
.animation-slide-bottom {
    animation-name: slide-bottom
}
//动画效果
@keyframes slide-bottom {
    0% {
        opacity: 0;
        transform: translateY(100%)
    }
    100% {
        opacity: 1;
        transform: translateY(0)
    }
}
```
`animation-slide-bottom`是动画类名，`animation-delay`是每一个卡片动画执行的延迟时间，每一个动画的执行时长为0.5s,所以延迟时间是以0.5s递增的，三个卡片的动画总时长就为2s,即2s后就执行`onReady`中的`settimeout`事件结束动画
```!
特别提醒：动画的延迟时间，执行时间可以自行设计，动画效果过渡自然即可
```
```!
特别提醒：由于触发动画的钩子函数定义在页面初次渲染的生命周期函数中，故只有在页面初次渲染时才执行，避免每次显示页面时加载动画造成用户的视觉疲劳
```
## 智能推荐约拍对象
系统会根据约拍需求自动推荐约拍对象(个人开发精力有限，推荐算法后续推出。。。)

![](https://user-gold-cdn.xitu.io/2019/6/29/16ba13bfa125f13c?w=397&h=697&f=gif&s=4662891)

### 技术分享：CSS3实现酷炫搜索动画
在模态框内放置两个`view`标签，以下是标签定义
```wxml
 <view id='preloader'>               //外围的圆形框定义
    <view id='loader'></view>       //内部的线条定义
</view>
```
```wxss
#preloader {
  width: 150px;
  height: 150px;
  border-radius: 50%;
  border: 1px solid #97b2ff;
}
#loader {         //中间线条定义
  display: block;
  position: relative;
  left: 50%;
  top: 50%;
  width: 150px;
  height: 150px;
  margin: -75px 0 0 -75px;
  border-radius: 50%;
  border: 3px solid transparent;
  border-top-color: #97b2ff;
  -webkit-animation: spin 2s linear infinite;
  animation: spin 2s linear infinite;
}
#loader:before {          //通过伪类元素定义外围线条
  content: "";
  position: absolute;
  top: 5px;
  left: 5px;
  right: 5px;
  bottom: 5px;
  border-radius: 50%;
  border: 3px solid transparent;
  border-top-color: #97b2ff;
  -webkit-animation: spin 3s linear infinite;
  animation: spin 3s linear infinite;
}
#loader:after {       //通过伪类元素定义最内部线条
  content: "";
  position: absolute;
  top: 15px;
  left: 15px;
  right: 15px;
  bottom: 15px;
  border-radius: 50%;
  border: 3px solid transparent;
  border-top-color: #97b2ff;
  -webkit-animation: spin 1.5s linear infinite;
  animation: spin 1.5s linear infinite;
}
```
## 实名认证

![](https://user-gold-cdn.xitu.io/2019/6/29/16ba233da0455a4f?w=397&h=697&f=gif&s=331988)

嘿嘿，由于懒得给个人信息打码，就暂时不给大家演示认证过程了。。
### 技术分享：Ai视觉能力
很多小伙伴都有过在自己项目中使用AI技术的想法，但又因为入门AI的难度比较大，并且需要的时间较长就放弃了，现在给大家安利一个可以直接使用的AI服务，让AI不再具有神秘感(AI大佬可以忽略此部分。。)  
- 方案一  
在腾讯云中搜索身份证识别，上面会有详细的API文档以及测试工具帮助你快速使用

![身份证识别](https://user-gold-cdn.xitu.io/2019/6/29/16ba1eef2a39d1b8?w=1920&h=902&f=png&s=119398)

点击查看[腾讯云-身份证识别](https://cloud.tencent.com/document/product/866/33524)
- 方案二  
方案一是以提供API接口的形式提供身份证识别服务，而接下来要介绍的方案真的就比较简单了，在腾讯云中搜索智能图像,其中的增值服务AI智能图像能力，你可以通过云函数和云存储实现相应功能，基于小程序云开发的 AI DEMO中开发好了部分功能，你只需通过教程将云函数和组件引入你的项目即可使用

![](https://user-gold-cdn.xitu.io/2019/6/29/16ba20c7ba9b3aa9?w=1789&h=901&f=png&s=138973)

点击查看[腾讯云-智能图像](https://cloud.tencent.com/document/product/876/32345)  
点击查看[基于小程序云开发的 AI DEMO](https://github.com/TencentCloudBase/tcb-demo-ai)  
```!
特别提醒：当然使用这些服务也并非是完整的解决方案，对于身份证信息的加密、存储方案、安全协议等还是需要各位小伙伴自行设计解决方案哦。
```
## 云开发
> 云开发为开发者提供完整的原生云端支持和微信服务支持，弱化后端和运维概念，无需搭建服务器，使用平台提供的 API 进行核心业务开发，即可实现快速上线和迭代，同时这一能力，同开发者已经使用的云服务相互兼容，并不互斥。

官方文档中API被分为了小程序端和服务端，一开始看过两端的API之后，感觉好像没有什么不同啊，在查阅相关资料以及实际开发中某些业务的处理总结出一些经验后才明白了两者的不同，下面给各位具体说说两者的不同之处，应该能帮助大家在使用云开发实战时少踩一点坑

### 初始化的不同
#### 小程序端
全局声明一次
```app.js
if (!wx.cloud) {
      console.error('请使用 2.2.3 或以上的基础库以使用云能力')
    } else {
      wx.cloud.init({
        env:'xxx',
        traceUser: true,
      })
    }
```
#### 服务端
每个云函数中声明一次
```javascript
const cloud = require('wx-server-sdk')
cloud.init()
```
### 权限不同
#### 小程序端
在小程序端可以选择直接操作数据库，但由于是前端操作数据库存在一些安全问题，有较多的权限限制，在云控制中可对每个集合进行权限设置，这也就是为什么有小伙伴在小程序端对某些数据进行更新，显示更新成功但并未更新数据，就是因为小程序端默认只能更新当前用户写入的数据
![](https://user-gold-cdn.xitu.io/2019/6/29/16ba28cfebd5f487?w=1308&h=584&f=png&s=54194)
```!
特别提醒：在小程序端使用创建者的权限对数据进行修改时一定要确保该集合中有_openid字段，否则系统在权限判断时是没有办法识别当前操作为创建者的，数据修改无法执行
```
#### 服务端
服务端拥有管理员的权限，对所有数据拥有读写权限
### 语法支持不同
#### 小程序端
在微信开发者工具里，以及Android端手机（浏览器内核是QQ浏览器的X5），`async`/`await`是天然支持的，但 iOS 端手机在较低版本则不支持，因此需要引入额外的`polyfill`。可以在有使用`async`/`await` 的文件当中引入`polyfill`文件。
```javasccript
const runtime = require('相对路径/lib/runtime')
```
#### 服务端
在云函数里，由于 Node 版本最低是 8.9，因此是天然支持 async/await 语法的  
示例：获取约拍需求列表
```javascript
//云函数入口文件
const cloud = require('wx-server-sdk')
//初始化
cloud.init()
//连接数据库
const db = cloud.database()
async function getAll(){
    const result = await db.collection('ypList')
    .orderBy('cameraInfo.launchTime','desc').where({}).get()
    return result
}
// 云函数入口函数
exports.main = async (event, context) => {
    //此处的action是用来判断该调用哪一个方法
    if(event.action === 'getAll'){
        return getAll()
    }
}
```
# 结语
一个人手撸个全栈项目确实很辛苦，但收获也很多。至少对于小程序的实战开发更为熟练了，对MVVM的思想的理解也更加深刻了。技术发展得很快，学习一项技术如果不深入其本质，那么技术是学不完的。深入学习就是个解决问题的过程，或是帮助别人解决问题，或是借助他人的力量解决问题。目前在正在学习Vue、React、TypeScript等技术，后续会推出相关技术的项目解析文章，希望对于同样在学习的你有帮助。
```!
特别说明：本项目已参加2019届中国高校计算机-微信应用开发赛完，开源至github，感兴趣的小伙伴可以看看
```

# 附录
在此提供一些本项目涉及到的技术、工具等链接供大家学习使用  
- 产品原型设计工具：[墨刀](https://modao.cc/)   
- 色彩搭配设计：[配色网](http://www.peise.net/palette/)  
- 在线作图：[ProcessOn](https://www.processon.com/)
- UI样式库：[WeUI](https://github.com/Tencent/weui)  
- UI样式库：[ColorUI](https://github.com/weilanwl/ColorUI)  
- 图标库：[Iconfont阿里巴巴矢量图标库](https://www.iconfont.cn/)  
- 开发工具：[微信开发者工具](https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html)  
- 开发者工具：[Vscode](https://code.visualstudio.com/)
- 腾讯云服务：[身份证识别](https://user-gold-cdn.xitu.io/2019/6/29/16ba1eef2a39d1b8?w=1920&h=902&f=png&s=119398)  
- 腾讯云服务：[智能图像](https://cloud.tencent.com/document/product/876/32345)  
- API文档：[微信官方文档.小程序](https://developers.weixin.qq.com/miniprogram/dev/framework/)  
- 技术文档：[ES6](https://es6.ruanyifeng.com/)

# 源码链接
[https://github.com/xmc1214/YPW](https://github.com/xmc1214/YPW)
