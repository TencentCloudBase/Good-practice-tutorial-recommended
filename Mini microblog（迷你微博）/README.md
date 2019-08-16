# 迷你微博
### 0. 前言

本文将手把手教你如何写出迷你版微博的一行行代码，迷你版微博包含以下功能：

- Feed 流：关注动态、所有动态
- 发送图文动态
- 搜索用户
- 关注系统
- 点赞动态
- 个人主页

使用到的云开发能力：

- 云数据库
- 云存储
- 云函数
- 云调用

没错，几乎是所有的云开发能力。也就是说，读完这篇实战，你就相当于完全入门了云开发！

咳咳，当然，实际上这里只是介绍核心逻辑和重点代码片段，完整代码建议下载查看。

### 1. 取得授权

作为一个社交平台，首先要做的肯定是经过用户授权，获取用户信息，小程序提供了很方便的接口：

```html
<button open-type="getUserInfo" bindgetuserinfo="getUserInfo">
  进入小圈圈
</button>
```

这个 `button` 有个 `open-type` 属性，这个属性是专门用来使用小程序的开放能力的，而 `getUserInfo` 则表示 **获取用户信息，可以从`bindgetuserinfo`回调中获取到用户信息**。

于是我们可以在 wxml 里放入这个 `button` 后，在相应的 js 里写如下代码：

```js
Page({
  ...

  getUserInfo: function(e) {
    wx.navigateTo({
      url: "/pages/circle/circle"
    })
  },

  ...
})
```

这样在成功获取到用户信息后，我们就能跳转到迷你微博页面了。

需要注意，不能使用 `wx.authorize({scope: "scope.userInfo"})` 来获取读取用户信息的权限，因为它不会跳出授权弹窗。目前只能使用上面所述的方式实现。

### 2. 主页设计

社交平台的主页大同小异，主要由三个部分组成：

- Feed 流
- 消息
- 个人信息

那么很容易就能想到这样的布局（注意新建一个 Page 哦，路径：`pages/circle/circle.wxml`）：

```html
<view class="circle-container">
  <view
    style="display:{{currentPage === 'main' ? 'block' : 'none'}}"
    class="main-area"
  >
  </view>

  <view
    style="display:{{currentPage === 'msg' ? 'flex' : 'none'}}"
    class="msg-area"
  >
  </view>

  <view
    style="display:{{currentPage === 'me' ? 'flex' : 'none'}}"
    class="me-area"
  >
  </view>

  <view class="footer">
    <view class="footer-item">
      <button
        class="footer-btn"
        bindtap="onPageMainTap"
        style="background: {{currentPage === 'main' ? '#111' : 'rgba(0,0,0,0)'}}; color: {{currentPage === 'main' ? '#fff' : '#000'}}"
      >
        首页
      </button>
    </view>
    <view class="footer-item">
      <button
        class="footer-btn"
        bindtap="onPageMsgTap"
        style="background: {{currentPage === 'msg' ? '#111' : 'rgba(0,0,0,0)'}}; color: {{currentPage === 'msg' ? '#fff' : '#000'}}"
      >
        消息
      </button>
    </view>
    <view class="footer-item">
      <button
        class="footer-btn"
        bindtap="onPageMeTap"
        style="background: {{currentPage === 'me' ? '#111' : 'rgba(0,0,0,0)'}}; color: {{currentPage === 'me' ? '#fff' : '#000'}}"
      >
        个人
      </button>
    </view>
  </view>
</view>
```

很好理解，画面主要被分为上下两个部分：上面的部分是主要内容，下面的部分是三个 Tab 组成的 Footer。重点 WXSS 实现（完整的 WXSS 可以下载源码查看）：

```css
.footer {
  box-shadow: 0 0 15rpx #ccc;
  display: flex;
  position: fixed;
  height: 120rpx;
  bottom: 0;
  width: 100%;
  flex-direction: row;
  justify-content: center;
  z-index: 100;
  background: #fff;
}

.footer-item {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  width: 33.33%;
  color: #333;
}

.footer-item:nth-child(2) {
  border-left: 3rpx solid #aaa;
  border-right: 3rpx solid #aaa;
  flex-grow: 1;
}

.footer-btn {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 0;
  font-size: 30rpx;
}
```

核心逻辑是通过 `position: fixed` 来让 Footer 一直在下方。

读者会发现有一个 `currentPage` 的 data ，这个 data 的作用其实很直观：通过判断它的值是 `main`/`msg`/`me` 中的哪一个来决定主要内容。同时，为了让首次使用的用户知道自己在哪个 Tab，Footer 中相应的 `button` 也会从白底黑字黑底白字，与另外两个 Tab 形成对比。

现在我们来看看 `main` 部分的代码（在上面代码的基础上扩充）:

```html
...
<view
  class="main-header"
  style="display:{{currentPage === 'main' ? 'flex' : 'none'}};max-height:{{mainHeaderMaxHeight}}"
>
  <view class="group-picker-wrapper">
    <picker
      bindchange="bindGroupPickerChange"
      value="{{groupArrayIndex}}"
      range="{{groupArray}}"
      class="group-picker"
    >
      <button class="group-picker-inner">
        {{groupArray[groupArrayIndex]}}
      </button>
    </picker>
  </view>
  <view class="search-btn-wrapper">
    <button class="search-btn" bindtap="onSearchTap">搜索用户</button>
  </view>
</view>
<view
  class="main-area"
  style="display:{{currentPage === 'main' ? 'block' : 'none'}};height: {{mainAreaHeight}};margin-top:{{mainAreaMarginTop}}"
>
  <scroll-view scroll-y class="main-area-scroll" bindscroll="onMainPageScroll">
    <block
      wx:for="{{pageMainData}}"
      wx:for-index="idx"
      wx:for-item="itemName"
      wx:key="_id"
    >
      <post-item is="post-item" data="{{itemName}}" class="post-item-wrapper" />
    </block>
    <view wx:if="{{pageMainData.length === 0}}" class="item-placeholder"
      >无数据</view
    >
  </scroll-view>
  <button
    class="add-poster-btn"
    bindtap="onAddPosterTap"
    hover-class="add-poster-btn-hover"
    style="bottom:{{addPosterBtnBottom}}"
  >
    +
  </button>
</view>
...
```

这里用到了 [列表渲染](https://developers.weixin.qq.com/miniprogram/dev/reference/wxml/list.html) 和 [条件渲染](https://developers.weixin.qq.com/miniprogram/dev/reference/wxml/conditional.html)，还不清楚的可以点击进去学习一下。

可以看到，相比之前的代码，我添加一个 header，同时 `main-area` 的内部也新增了一个 `scroll-view`（用于展示 Feed 流） 和一个 `button`（用于编辑新迷你微博）。header 的功能很简单：左侧区域是一个 `picker`，可以选择查看的动态类型（目前有 _关注动态_ 和 _所有动态_ 两种）；右侧区域是一个按钮，点击后可以跳转到搜索页面，这两个功能我们先放一下，先继续看 `main-area` 的新增内容。

`main-area` 里的 `scroll-view` 是一个可监听滚动事件的列表，其中监听事件的实现：

```js
data: {
  ...
  addPosterBtnBottom: "190rpx",
  mainHeaderMaxHeight: "80rpx",
  mainAreaHeight: "calc(100vh - 200rpx)",
  mainAreaMarginTop: "80rpx",
},
onMainPageScroll: function(e) {
  if (e.detail.deltaY < 0) {
    this.setData({
      addPosterBtnBottom: "-190rpx",
      mainHeaderMaxHeight: "0",
      mainAreaHeight: "calc(100vh - 120rpx)",
      mainAreaMarginTop: "0rpx"
    })
  } else {
    this.setData({
      addPosterBtnBottom: "190rpx",
      mainHeaderMaxHeight: "80rpx",
      mainAreaHeight: "calc(100vh - 200rpx)",
      mainAreaMarginTop: "80rpx"
    })
  }
},
...
```

结合 wxml 可以知道，当页面向下滑动 （deltaY < 0） 时，header 和 `button` 会 “突然消失”，反之它们则会 “突然出现”。为了视觉上有更好地过渡，我们可以在 WXSS 中使用 `transition` ：

```css
...
.main-area {
  position: relative;
  flex-grow: 1;
  overflow: auto;
  z-index: 1;
  transition: height 0.3s, margin-top 0.3s;
}
.main-header {
  position: fixed;
  width: 100%;
  height: 80rpx;
  background: #fff;
  top: 0;
  left: 0;
  display: flex;
  justify-content: space-around;
  align-items: center;
  z-index: 100;
  border-bottom: 3rpx solid #aaa;
  transition: max-height 0.3s;
  overflow: hidden;
}
.add-poster-btn {
  position: fixed;
  right: 60rpx;
  box-shadow: 5rpx 5rpx 10rpx #aaa;
  display: flex;
  justify-content: center;
  align-items: center;
  color: #333;
  padding-bottom: 10rpx;
  text-align: center;
  border-radius: 50%;
  font-size: 60rpx;
  width: 100rpx;
  height: 100rpx;
  transition: bottom 0.3s;
  background: #fff;
  z-index: 1;
}
...
```

### 3. Feed 流

#### 3.1 post-item

前面提到，`scroll-view` 的内容是 Feed 流，那么首先就要想到使用 [列表渲染](https://developers.weixin.qq.com/miniprogram/dev/reference/wxml/list.html)。而且，为了方便在个人主页复用，列表渲染中的每一个 item 都要抽象出来。这时就要使用小程序中的 [Custom-Component](https://developers.weixin.qq.com/miniprogram/dev/framework/custom-component/) 功能了。

新建一个名为 `post-item` 的 `Component`，其中 wxml 的实现（路径：`pages/circle/component/post-item/post-item.js`）：

```html
<view
  class="post-item"
  hover-class="post-item-hover"
  bindlongpress="onItemLongTap"
  bindtap="onItemTap"
>
  <view class="post-title">
    <view class="author" hover-class="author-hover" catchtap="onAuthorTap"
      >{{data.author}}</view
    >
    <view class="date">{{data.formatDate}}</view>
  </view>
  <view class="msg-wrapper">
    <text class="msg">{{data.msg}}</text>
  </view>
  <view class="image-outer" wx:if="{{data.photoId !== ''}}" catchtap="onImgTap">
    <image-wrapper is="image-wrapper" src="{{data.photoId}}" />
  </view>
</view>
```

可见，一个 `poster-item` 最主要有以下信息：

- 作者名
- 发送时间
- 文本内容
- 图片内容

其中，图片内容因为是可选的，所以使用了 [条件渲染](https://developers.weixin.qq.com/miniprogram/dev/reference/wxml/conditional.html)，这会在没有图片信息时不让图片显示区域占用屏幕空间。另外，图片内容主要是由 `image-wrapper` 组成，它也是一个 `Custom-Component`，主要功能是：

- 强制长宽 1:1 裁剪显示图片
- 点击查看大图
- 未加载完成时显示 加载中

具体代码这里就不展示了，比较简单，读者可以在 `component/image-wrapper` 里找到。

回过头看 `main-area` 的其他新增部分，细心的读者会发现有这么一句：

```html
<view wx:if="{{pageMainData.length === 0}}" class="item-placeholder"
  >无数据</view
>
```

这会在 Feed 流暂时没有获取到数据时给用户一个提示。

#### 3.2 collections: poster、poster_users

展示 Feed 流的部分已经编写完毕，现在就差实际数据了。根据上一小节 `poster-item` 的主要信息，我们可以初步推断出一条迷你微博在 [云数据库](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/basis/capabilities.html#%E6%95%B0%E6%8D%AE%E5%BA%93) 的 collection `poster` 里是这样存储的：

```json
{
  "username": "Tester",
  "date": "2019-07-22 12:00:00",
  "text": "Ceshiwenben",
  "photo": "xxx"
}
```

先来看 `username`。由于社交平台一般不会限制用户的昵称，所以如果每条迷你微博都存储昵称，那将来每次用户修改一次昵称，就要遍历数据库把所有迷你微博项都改一遍，相当耗费时间，所以我们不如存储一个 `userId`，并另外把 id 和 昵称 的对应关系存在另一个叫 `poster_users` 的 collection 里。

```json
{
  "userId": "xxx",
  "name": "Tester",
  ...（其他用户信息）
}
```

`userId` 从哪里拿呢？当然是通过之前已经授权的获取用户信息接口拿到了，详细操作之后会说到。

接下来是 `date`，这里最好是服务器时间（因为客户端传过来的时间可能会有误差），而云开发文档里也有提供相应的接口：[serverDate](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/reference-client-api/database/db.serverDate.html#db.serverDate)。这个数据可以直接被 `new Date()` 使用，可以理解为一个 UTC 时间。

`text` 即文本信息，直接存储即可。

`photo` 则表示附图数据，但是限于小程序 `image` 元素的实现，想要显示一张图片，要么提供该图片的 url，要么提供该图片在 [云存储](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/basis/capabilities.html#%E5%AD%98%E5%82%A8) 的 id，所以这里最佳的实践是：先把图片上传到云存储里，然后把回调里的文件 id 作为数据存储。

综上所述，最后 `poster` 每一项的数据结构如下：

```json
{
  "authorId": "xxx",
  "date": "utc-format-date",
  "text": "Ceshiwenben",
  "photoId": "yyy"
}
```

确定数据结构后，我们就可以开始往 collection 添加数据了。但是，在此之前，我们还缺少一个重要步骤。

#### 3.3 用户信息录入 与 云数据库

没错，我们还没有在 `poster_users` 里添加一条新用户的信息。这个步骤一般在 `pages/circle/circle` 页面首次加载时判断即可：

```js
getUserId: function(cb) {
  let that = this
  var value = this.data.userId || wx.getStorageSync("userId")
  if (value) {
    if (cb) {
      cb(value)
    }
    return value
  }
  wx.getSetting({
    success(res) {
      if (res.authSetting["scope.userInfo"]) {
        wx.getUserInfo({
          withCredentials: true,
          success: function(userData) {
            wx.setStorageSync("userId", userData.signature)
            that.setData({
              userId: userData.signature
            })
            db.collection("poster_users")
              .where({
                userId: userData.signature
              })
              .get()
              .then(searchResult => {
                if (searchResult.data.length === 0) {
                  wx.showToast({
                    title: "新用户录入中"
                  })
                  db.collection("poster_users")
                    .add({
                      data: {
                        userId: userData.signature,
                        date: db.serverDate(),
                        name: userData.userInfo.nickName,
                        gender: userData.userInfo.gender
                      }
                    })
                    .then(res => {
                      console.log(res)
                      if (res.errMsg === "collection.add:ok") {
                        wx.showToast({
                          title: "录入完成"
                        })
                        if (cb) cb()
                      }
                    })
                    .catch(err => {
                      wx.showToast({
                        title: "录入失败，请稍后重试",
                        image: "/images/error.png"
                      })
                      wx.navigateTo({
                        url: "/pages/index/index"
                      })
                    })
                } else {
                  if (cb) cb()
                }
              })
          }
        })
      } else {
        wx.showToast({
          title: "登陆失效，请重新授权登陆",
          image: "/images/error.png"
        })
        wx.navigateTo({
          url: "/pages/index/index"
        })
      }
    }
  })
}
```

代码实现比较复杂，整体思路是这样的：

1. 判断是否已存储了 `userId`，如果有直接返回并调用回调函数，如果没有继续 2
2. 通过 `wx.getSetting` 获取当前设置信息
3. 如果返回里有 `res.authSetting["scope.userInfo"]` 说明已经授权读取用户信息，继续 3，没有授权的话就跳转回首页重新授权
4. 调用 `wx.getUserInfo` 获取用户信息，成功后提取出 `signature`（这是每个微信用户的唯一签名），并调用 `wx.setStorageSync` 将其缓存
5. 调用 `db.collection().where().get()` ，判断返回的数据是否是空数组，如果不是说明该用户已经录入（注意 `where()` 中的筛选条件），如果是说明该用户是新用户，继续 5
6. 提示新用户录入中，同时调用 `db.collection().add()` 来添加用户信息，最后通过回调判断是否录入成功，并提示用户

不知不觉我们就使用了云开发中的 云数据库 功能，紧接着我们就要开始使用 云存储 和 云函数了！

#### 3.4 addPoster 与 云存储

发送新的迷你微博，需要一个编辑新迷你微博的界面，路径我定为 `pages/circle/add-poster/add-poster`：

```html
<view class="app-poster-container">
  <view class="body">
    <view class="text-area-wrapper">
      <textarea bindinput="bindTextInput" placeholder="在此填写" value="{{text}}" auto-focus="true" />
      <view class="text-area-footer">
        <text>{{remainLen}}/140</text>
      </view>
    </view>
    <view bindtap="onImageTap" class="image-area">
      <view class="image-outer">
        <image-wrapper is="image-wrapper" src="{{imageSrc}}" placeholder="选择图片上传" />
      </view>
    </view>
  </view>
  <view class="footer">
    <button class="footer-btn" bindtap="onSendTap">发送</button>
  </view>
</view>
```

wxml 的代码很好理解：`textarea` 显示编辑文本，`image-wrapper` 显示需要上传的图片，最下面是一个发送的 `button`。其中，图片编辑区域的 `bindtap` 事件实现：

```js
onImageTap: function() {
  let that = this
  wx.chooseImage({
    count: 1,
    success: function(res) {
      const tempFilePaths = res.tempFilePaths
      that.setData({
        imageSrc: tempFilePaths[0]
      })
    }
  })
}
```

直接通过 `wx.chooseImage` 官方 API 获取本地图片的临时路径即可。而当发送按钮点击后，会有如下代码被执行：

```js
onSendTap: function() {
  if (this.data.text === "" && this.data.imageSrc === "") {
    wx.showModal({
      title: "错误",
      content: "不能发送空内容",
      showCancel: false,
      confirmText: "好的"
    })
    return
  }
  const that = this
  wx.showLoading({
    title: "发送中",
    mask: true
  })
  const imageSrc = this.data.imageSrc
  if (imageSrc !== "") {
    const finalPath = imageSrc.replace("//", "/").replace(":", "")
    wx.cloud
      .uploadFile({
        cloudPath: finalPath,
        filePath: imageSrc // 文件路径
      })
      .then(res => {
        that.sendToDb(res.fileID)
      })
      .catch(error => {
        that.onSendFail()
      })
  } else {
    that.sendToDb()
  }
},
sendToDb: function(fileId = "") {
  const that = this
  const posterData = {
    authorId: that.data.userId,
    msg: that.data.text,
    photoId: fileId,
    date: db.serverDate()
  }
  db.collection("poster")
    .add({
      data: {
        ...posterData
      }
    })
    .then(res => {
      wx.showToast({
        title: "发送成功"
      })
      wx.navigateBack({
        delta: 1
      })
    })
    .catch(error => {
      that.onSendFail()
    })
    .finally(wx.hideLoading())
}
```

1. 首先判断文本和图片内容是否都为空，如果是则不执行发送，如果不是继续 2
2. 提示发送中，上传图片到云存储，注意需要将图片中的临时 url 的一些特殊字符组合替换一下，原因见 [文件名命名限制](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/guide/storage/naming.html)
3. 上传成功后，调用 `db.collection().add()`，发送成功后退回上一页（即首页），如果失败则执行 `onSendFail` 函数，后者见源码，逻辑较简单这里不赘述

于是，我们就这样创建了第一条迷你微博。接下来就让它在 Feed 流中显示吧！

#### 3.5 云函数 getMainPageData

这个函数的主要作用如前所述，就是通过处理云数据库中的数据，将最终数据返回给客户端，后者将数据可视化给用户。我们先做一个初步版本，因为现在 `poster_users` 中只有一条数据，所以仅先展示自己的迷你微博。`getMainPageData` 云函数代码如下：

```js
// 云函数入口文件
const cloud = require("wx-server-sdk")
cloud.init()
const db = cloud.database()

// 云函数入口函数
exports.main = async (event, context, cb) => {
  // 通过 event 获取入参
  const userId = event.userId
  let followingResult
  let users
  // idNameMap 负责存储 userId 和 name 的映射关系
  let idNameMap = {}
  let followingIds = []
  // 获取用户信息
  followingResult = await db
      .collection("poster_users")
      .where({
        userId: userId
      })
      .get()
    users = followingResult.data
    followingIds = users.map(u => {
      return u.userId
    })
  users.map(u => {
    idNameMap[u.userId] = u.name
  })
  // 获取动态
  const postResult = await db
    .collection("poster")
    .orderBy("date", "desc")
    .where({
      // 通过高级筛选功能筛选出符合条件的 userId
      authorId: db.command.in(followingIds)
    })
    .get()
  const postData = postResult.data
  // 向返回的数据添加 存储用户昵称的 author 属性、存储格式化后的时间的 formatDate 属性
  postData.map(p => {
    p.author = idNameMap[p.authorId]
    p.formatDate = new Date(p.date).toLocaleDateString("zh-Hans", options)
  })
  return postData
}
```

最后在 `pages/circle/circle.js` 里补充云调用：

```js
getMainPageData: function(userId) {
  const that = this
  wx.cloud
    .callFunction({
      name: "getMainPageData",
      data: {
        userId: userId,
        isEveryOne: that.data.groupArrayIndex === 0 ? false : true
      }
    })
    .then(res => {
      that.setData({
        pageMainData: res.result,
        pageMainLoaded: true
      })
    })
    .catch(err => {
      wx.showToast({
        title: "获取动态失败",
        image: "/images/error.png"
      })
      wx.hideLoading()
    })
}
```

即可展示 Feed 流数据给用户。

之后，`getMainPageData` 还会根据使用场景的不同，新增了查询所有用户动态、查询关注用户动态的功能，但是原理是一样的，看源码可以轻易理解，后续就不再说明。

### 4. 关注系统

上一节中我们一口气把云开发中的大部分主要功能：云数据库、云存储、云函数、云调用都用了一遍，接下来其他功能的实现也基本都依赖它们。

#### 4.1 poster_user_follows

首先我们需要建一个新的 collection `poster_user_follows`，其中的每一项数据的数据结构如下：

```json
{
  "followerId": "xxx",
  "followingId": "xxx"
}
```

很简单，`followerId` 表示关注人，`followingId` 表示被关注人。

#### 4.2 user-data 页面

关注或者取消关注需要进入他人的个人主页操作，我们在 `pages/circle/user-data/user-data.wxml` 中放一个 `user-info` 的自定义组件，然后新建该组件编辑：

```html
<view class="user-info">
  <view class="info-item" hover-class="info-item-hover">用户名: {{userName}}</view>
  <view class="info-item" hover-class="info-item-hover" bindtap="onPosterCountTap">动态数: {{posterCount}}</view>
  <view class="info-item" hover-class="info-item-hover" bindtap="onFollowingCountTap">关注数: {{followingCount}}</view>
  <view class="info-item" hover-class="info-item-hover" bindtap="onFollowerCountTap">粉丝数: {{followerCount}}</view>
  <view class="info-item" hover-class="info-item-hover" wx:if="{{originId && originId !== '' && originId !== userId}}"><button bindtap="onFollowTap">{{followText}}</button></view>
</view>
```

这里注意条件渲染的 `button`：如果当前访问个人主页的用户 id （originId） 和 被访问的用户 id （userId）的值是相等的话，这个按钮就不会被渲染（自己不能关注/取消关注自己）。

我们重点看下 `onFollowTap` 的实现：

```js
onFollowTap: function() {
  const that = this
  // 判断当前关注状态
  if (this.data.isFollow) {
    wx.showLoading({
      title: "操作中",
      mask: true
    })
    wx.cloud
      .callFunction({
        name: "cancelFollowing",
        data: {
          followerId: this.properties.originId,
          followingId: this.properties.userId
        }
      })
      .then(res => {
        wx.showToast({
          title: "取消关注成功"
        })
        that.setData({
          isFollow: false,
          followText: "关注"
        })
      })
      .catch(error => {
        wx.showToast({
          title: "取消关注失败",
          image: "/images/error.png"
        })
      })
      .finally(wx.hideLoading())
  } else if (this.data.isFollow !== undefined) {
    wx.showLoading({
      title: "操作中",
      mask: true
    })
    const data = {
      followerId: this.properties.originId,
      followingId: this.properties.userId
    }
    db.collection("poster_user_follows")
      .add({
        data: {
          ...data
        }
      })
      .then(res => {
        wx.showToast({
          title: "关注成功"
        })
        that.setData({
          isFollow: true,
          followText: "取消关注"
        })
      })
      .catch(error => {
        wx.showToast({
          title: "关注失败",
          image: "/images/error.png"
        })
      })
      .finally(wx.hideLoading())
    }
  }
}
```

这里读者可能会有疑问：为什么关注的时候直接调用 `db.collection().add()` 即可，而取消关注却要调用云函数呢？这里涉及到云数据库的设计问题：删除多个数据的操作，或者说删除使用 `where` 筛选的数据，只能在服务端执行。如果确实想在客户端删除，则在查询用户关系时，将唯一标识数据的 `_id` 用 `setData` 存下来，之后再使用 `db.collection().doc(_id).delete()` 删除即可。这两种实现方式读者可自行选择。当然，还有一种实现是不实际删除数据，只是加个 `isDelete` 字段标记一下。

查询用户关系的实现很简单，云函数的实现方式如下：

```js
// 云函数入口文件
const cloud = require('wx-server-sdk')
cloud.init()
const db = cloud.database()

// 云函数入口函数
exports.main = async(event, context) => {
  const followingResult = await db.collection("poster_user_follows")
    .where({
      followingId: event.followingId,
      followerId: event.followerId
    }).get()
  return followingResult
}
```

客户端只要检查返回的数据长度是否大于 0 即可。

另外附上 `user-data` 页面其他数据的获取云函数实现：

```js
// 云函数入口文件
const cloud = require("wx-server-sdk")
cloud.init()
const db = cloud.database()

async function getPosterCount(userId) {
  return {
    value: (await db.collection("poster").where({
      authorId: userId
    }).count()).total,
    key: "posterCount"
  }
}

async function getFollowingCount(userId) {
  return {
    value: (await db.collection("poster_user_follows").where({
      followerId: userId
    }).count()).total,
    key: "followingCount"
  }
}

async function getFollowerCount(userId) {
  return {
    value: (await db.collection("poster_user_follows").where({
      followingId: userId
    }).count()).total,
    key: "followerCount"
  }
}


async function getUserName(userId) {
  return {
    value: (await db.collection("poster_users").where({
      userId: userId
    }).get()).data[0].name,
    key: "userName"
  }
}

// 云函数入口函数
exports.main = async (event, context) => {
  const userId = event.userId
  const tasks = []
  tasks.push(getPosterCount(userId))
  tasks.push(getFollowerCount(userId))
  tasks.push(getFollowingCount(userId))
  tasks.push(getUserName(userId))
  const allData = await Promise.all(tasks)
  const finalData = {}
  allData.map(d => {
    finalData[d.key] = d.value
  })
  return finalData
}
```

很好理解，客户端获取返回后直接使用即可。

### 5. 搜索页面

这部分其实很好实现。关键的搜索函数实现如下：

```js
// 云函数入口文件
const cloud = require('wx-server-sdk')
cloud.init()
const db = cloud.database()

const MAX_LIMIT = 100
async function getDbData(dbName, whereObj) {
  const totalCountsData = await db.collection(dbName).where(whereObj).count()
  const total = totalCountsData.total
  const batch = Math.ceil(total / 100)
  const tasks = []
  for (let i = 0; i < batch; i++) {
    const promise = db
      .collection(dbName)
      .where(whereObj)
      .skip(i * MAX_LIMIT)
      .limit(MAX_LIMIT)
      .get()
    tasks.push(promise)
  }
  const rrr = await Promise.all(tasks)
  if (rrr.length !== 0) {
    return rrr.reduce((acc, cur) => {
      return {
        data: acc.data.concat(cur.data),
        errMsg: acc.errMsg
      }
    })
  } else {
    return {
      data: [],
      errMsg: "empty"
    }
  }
}

// 云函数入口函数
exports.main = async (event, context) => {
  const text = event.text
  const data = await getDbData("poster_users", {
    name: {
      $regex: text
    }
  })
  return data
}
```

这里参考了官网所推荐的分页检索数据库数据的实现（因为搜索结果可能有很多），筛选条件则是正则模糊匹配关键字。

搜索页面的源码路径是 `pages/circle/search-user/search-user`，实现了点击搜索结果项跳转到对应项的用户的 `user-data` 页面，建议直接阅读源码理解。

### 6. 其他扩展

#### 6.1 poster_likes 与 点赞

由于转发、评论、点赞的原理基本相同，所以这里只介绍点赞功能如何编写，另外两个功能读者可以自行实现。

毫无疑问我们需要新建一个 collection `poster_likes`，其中每一项的数据结构如下：

```json
{
  "posterId": "xxx",
  "likeId": "xxx"
}
```

这里的 `posterId` 就是 `poster` collection 里每条记录的 `_id` 值，`likeId` 就是 `poster_users` 里的 `userId` 了。

然后我们扩展一下 `poster-item` 的实现：

```html
<view class="post-item" hover-class="post-item-hover" bindlongpress="onItemLongTap" bindtap="onItemTap">
  ...
  <view class="interact-area">
    <view class="interact-item">
      <button class="interact-btn" catchtap="onLikeTap" style="color:{{liked ? '#55aaff' : '#000'}}">赞 {{likeCount}}</button>
    </view>
  </view>
</view>
```

即，新增一个 `interact-area`，其中 `onLikeTap` 实现如下：

```js
onLikeTap: function() {
  if (!this.properties.originId) return
  const that = this
  if (this.data.liked) {
    wx.showLoading({
      title: "操作中",
      mask: true
    })
    wx.cloud
      .callFunction({
        name: "cancelLiked",
        data: {
          posterId: this.properties.data._id,
          likeId: this.properties.originId
        }
      })
      .then(res => {
        wx.showToast({
          title: "取消成功"
        })
        that.refreshLike()
        that.triggerEvent('likeEvent');
      })
      .catch(error => {
        wx.showToast({
          title: "取消失败",
          image: "/images/error.png"
        })
      })
      .finally(wx.hideLoading())
  } else {
    wx.showLoading({
      title: "操作中",
      mask: true
    })
    db.collection("poster_likes").add({
        data: {
          posterId: this.properties.data._id,
          likeId: this.properties.originId
        }
      }).then(res => {
        wx.showToast({
          title: "已赞"
        })
        that.refreshLike()
        that.triggerEvent('likeEvent');
      })
      .catch(error => {
        wx.showToast({
          title: "赞失败",
          image: "/images/error.png"
        })
      })
      .finally(wx.hideLoading())
  }

}
```

细心的读者会发现这和关注功能原理几乎是一样的。

#### 6.2 数据刷新

我们可以使用很多方式让主页面刷新数据：

```js
onShow: function() {
  wx.showLoading({
    title: "加载中",
    mask: true
  })
  const that = this
  function cb(userId) {
    that.refreshMainPageData(userId)
    that.refreshMePageData(userId)
  }
  this.getUserId(cb)
}
```

第一种是利用 `onShow` 方法：它会在页面每次从后台转到前台展示时调用，这个时候我们就能刷新页面数据（包括 Feed 流和个人信息）。但是这个时候用户信息可能会丢失，所以我们需要在 `getUserId` 里判断，并将刷新数据的函数们整合起来，作为回调函数。

第二种是让用户手动刷新：

```js
onPageMainTap: function() {
  if (this.data.currentPage === "main") {
    this.refreshMainPageData()
  }
  this.setData({
    currentPage: "main"
  })
}
```

如图所示，当目前页面是 Feed 流时，如果再次点击 首页 Tab，就会强制刷新数据。

第三种是关联数据变更触发刷新，比如动态类型选择、删除了一条动态以后触发数据的刷新。这种可以直接看源码学习。

#### 6.3 首次加载等待

当用户第一次进入主页面时，我们如果想在 Feed 流和个人信息都加载好了再允许用户操作，应该如何实现？

如果是类似 Vue 或者 React 的框架，我们很容易就能想到属性监控，如 `watch`、`useEffect` 等等，但是小程序目前 `Page` 并没有提供属性监控功能，怎么办？

除了自己实现，还有一个方法就是利用 `Component` 的 `observers`，它和上面提到的属性监控功能差不多。虽然官网文档对其说明比较少，但摸索了一番还是能用来监控的。

首先我们来新建一个 `Component` 叫 `abstract-load`，具体实现如下：

```js
// pages/circle/component/abstract-load.js
Component({
  properties: {
    pageMainLoaded: {
      type: Boolean,
      value: false
    },
    pageMeLoaded: {
      type: Boolean,
      value: false
    }
  },
  observers: {
    "pageMainLoaded, pageMeLoaded": function (pageMainLoaded, pageMeLoaded) {
      if (pageMainLoaded && pageMeLoaded) {
        this.triggerEvent("allLoadEvent")
      }
    }
  }
})
```

然后在 `pages/circle/circle.wxml` 中添加一行：

```html
<abstract-load is="abstract-load" pageMainLoaded="{{pageMainLoaded}}" pageMeLoaded="{{pageMeLoaded}}" bind:allLoadEvent="onAllLoad" />
```

最后实现 `onAllLoad` 函数即可。

另外，像这种没有实际展示数据的 `Component`，建议在项目中都用 `abstract` 开头来命名。

#### 6.4 scroll-view 在 iOS 的 bug

如果读者使用 iOS 系统调试这个小程序，可能会发现 Feed 流比较短的时候，滚动 `scroll-view` header 和 `button` 会有鬼畜的上下抖动现象，这是因为 iOS 自己实现的 WebView 对于滚动视图有回弹的效果，而该效果也会触发滚动事件。

对于这个 bug，[官方人员也表示暂时无法修复](https://developers.weixin.qq.com/community/develop/doc/00064a122e01782f0d583fc0151c00)，只能先忍一忍了。

#### 6.5 关于消息 Tab

读者可能会疑惑我为什么没有讲解消息 Tab 以及消息提醒的实现。首先是因为源码没有这个实现，其次是我觉得目前云开发所提供的能力实现主动提醒比较麻烦（除了轮询想不到其他办法）。

希望未来云开发可以提供 ***数据库长连接监控*** 的功能，这样通过订阅者模式可以很轻松地获取到数据更新的状态，主动提醒也就更容易实现了。到那时我可能会再更新相关源码。

#### 6.6 关于云函数耗时

读者可能会发现我有一个叫 `benchmark` 的云函数，这个函数只是做了个查询数据库的操作，目的在于计算查询耗时。

诡异的是，我前天在调试的时候，发现查询一次需要1秒钟，而写这篇文章时却不到100ms。建议在一些需要多次操作数据库的函数配置里，把超时时间设置长一点吧。目前云函数的性能不太稳定。

### 7. 结语

那么关于迷你版微博开发实战介绍就到此为止了，更多资料可以直接下载源码查看哦。

# 源码链接
[https://github.com/KuthorX/KuthorX-Helper](https://github.com/KuthorX/KuthorX-Helper)
