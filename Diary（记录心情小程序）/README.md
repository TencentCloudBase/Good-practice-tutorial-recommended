> 闲暇之余，听媳妇嘀咕说要给她做一个能表达她每日心情的小程序，只能她在上面发东西。既然媳妇发话了，就花点心思做一个吧，因为没有UI图，所有布局全靠自己瞎编，下面结合图片和代码跟大家讲解下实现过程，内容略长，感兴趣的可以一览。

> 下面将以图片、代码的形式和大家讲解这个小demo的实现过程：

##  首页

### 首页效果图

![](https://puui.qpic.cn/vupload/0/20190813_1565663101023_pqmc8r3a7hp./0)

### 首页讲解

- 音乐(下面仅展示音乐相关代码)

```html
<div class="bg_music" @tap="audioPlay">
    <image src="../../static/images/music_icon.png" class="musicImg" :class="isPlay?'music_icon':''"/>
    <image src="../../static/images/music_play.png" class="music_play" :class="isPlay?'pauseImg':'playImg'"/>
</div>
<audio id="myAudio" :src="audioUrl" autoplay loop></audio>
```</br>

```javascript
data () {
  return {
    isPlay: true,
    audioCtx: ''
  }
},
onLoad () {
  const that = this
  that.audioCtx = wx.createAudioContext('myAudio')
  that.getMusicUrl()
},
methods: {
  getMusicUrl () {
    const that = this
    const db = wx.cloud.database()
    const music = db.collection('music')
    music.get().then(res => {
      that.audioUrl = res.data[0].musicUrl
      that.audioCtx.loop = true
      that.audioCtx.play()
    })
  },
  audioPlay () {
    const that = this
    if (that.isPlay) {
      that.audioCtx.pause()
      that.isPlay = !that.isPlay
      tools.showToast('您已暂停音乐播放~')
    } else {
      that.audioCtx.play()
      that.isPlay = !that.isPlay
      tools.showToast('背景音乐已开启~')
    }
  }
}
```</br>

```css
.bg_music
  position fixed
  right 0
  top 20rpx
  width 100rpx
  z-index 99
  display flex
  justify-content flex-start
  align-items flex-start
  .musicImg
    width 60rpx
    height 60rpx
  .music_icon
    animation musicRotate 3s linear infinite
  .music_play
    width 28rpx
    height 60rpx
    margin-left -10rpx
    transform-origin top
    -webkit-transform rotate(20deg)
  .playImg
    animation musicStop 1s linear forwards
  .pauseImg
    animation musicStart 1s linear forwards
#myAudio
  display none
```

1、通过wx.createInnerAudioContext()获取实例 ，安卓机上音乐能正常播放，IOS上不行，具体原因感兴趣的可以去深究一下；

2、由于前面邀请函小程序相关文章发出后，问的最多的问题依然是音乐无法播放这块，所以这个demo中就再给大家讲解了下实现的原理。

------

- 日历

> 这里日历使用了小程序插件，之所以在首页放一个日历是为了页面不显的太单调。下面讲解下插件是如何使用的：

 1、登录微信公众平台>设置>第三方设置>添加插件>搜索相关插件的名字（使用appId搜索更好）>点击某个插件右侧的查看详情，进入插件详情页添加插件，一般都能立马添加通过；

![](https://puui.qpic.cn/vupload/0/20190813_1565663507920_18d14d1vb3t./0)

2、插件详情里面一般都有使用文档，或git地址，插件的具体属性事件都会在文档里有介绍；

3、下面讲解下如何在项目中使用插件：

### 1、找到src根目录下的app.json文件，添加如下内容：

```javascript
// "cloud": true,
"plugins": {
  "calendar": {
    "version": "1.1.3",
    "provider": "wx92c68dae5a8bb046"
  }
}
```

### 2、在需要引用该插件的页面的.json文件中加入如下内容：

```javascript
{
  // "navigationBarTitleText": "媳妇的心情日记",
  // "enablePullDownRefresh": true,
  "usingComponents": {
    "calendar": "plugin://calendar/calendar"
  }
}
```

### 3、在页面中直接使用如下（具体属性方法的意思根据对应插件有所不同）：

```html
<calendar
    :class="showCalendar?'':'hide_right'"
    class="right"
    weeks-type="en"
    cell-size="20"
    :header="showHeader"
    show-more-days=true
    calendar-style="demo4-calendar"
    board-style="demo4-board"
    :days-color="demo4_days_style"
    @dayClick="dayClick"
/>
```
------

- 天气和地址

1、这里我借助的是[高德微信小程序SDK](https://lbs.amap.com/api/wx/summary/);

2、首先获取使用相关api需要的key值，如下：

![](https://puui.qpic.cn/vupload/0/20190813_1565663671496_w9g464o15h./0)

3、下载对应SDK（.js文件）并引入到项目中；

4、通过相关api获取天气和地址:

```javascript
getWeather () {
  const that = this
  let myAmapFun = new amapFile.AMapWX({key: '你申请的key'})
  myAmapFun.getWeather({
    success (res) {
      // 成功回调
      that.address = res.liveData.city
      that.weather = res.liveData.weather + ' '
      that.temperature = res.liveData.temperature + '℃'
      that.winddirection = res.liveData.winddirection + '风' + res.liveData.windpower + '级'
    },
    fail (info) {
      // 失败回调
      console.log(info)
    }
  })
},
```
------

- 发表日记

> 这里涉及到发表文字图片内容，在个人小程序提交审核后很大可能是不会被通过的，虽然第一次提交我的个人小程序通过审核了，后面几次审核均未通过，虽然我这里只限制了我和媳妇两个人能发日记，其他人压根看不到右下角的发布加号，但是审核人员会查代码，代码中一旦被他们发现有类似发表相关的内容或字样就会导致审核不通过，好在已经通过了一次，媳妇能正常写点东西，也算基本符合要求，遗憾的是后面实现点赞相关的功能都没有更新到线上。

![](https://puui.qpic.cn/vupload/0/20190813_1565663782616_60uuqubj2ms./0)

1、通过唯一的openId来判断是否显示首页右下角的发布加号；

2、后面会具体讲解页面里上传图片到云开发及存储到数据库相关功能

------

- 点赞功能

1、这里点赞功能借助的小程序云开发的云函数来实现的，结合代码：

```html
<ul class="list">
    <li class="item" v-for="(item, index) in diaryList" :key="item._id" @tap="toDetail(item)">
        <image class="like" src="../../static/images/like_active.png" v-if="likeList[index] === '2'" @tap.stop="toLike(item._id, '1', item.like)"/>
        <image class="like" src="../../static/images/like.png" v-if="likeList[index] === '1'" @tap.stop="toLike(item._id, '2', item.like)"/>
        <image class="img" :src="item.url" mode="aspectFill"/>
        <p class="desc">{{item.desc}}</p>
        <div class="name-weather">
            <span class="name">{{item.name}}</span>
            <span class="weather">{{item.weather}}</span>
        </div>
        <p class="time-address">
            <span class="time">{{item.time}}</span>
            <!-- <span class="address">{{item.address}}</span> -->
        </p>
    </li>
</ul>
<div class="dialog" v-if="showDialog">
    <div class="box">
        <h3>提示</h3>
        <p>是否授权使用点赞功能？</p>
        <div class="bottom">
            <button class="cancel" @tap="hideDialog">取消</button>
            <button class="confirm" lang="zh_CN" open-type="getUserInfo" @getuserinfo="login">确认</button>
        </div>
    </div>
</div>
```</br>

```javascript
// 获取日记列表
getDiaryList () {
  const that = this
  wx.cloud.callFunction({
    name: 'diaryList',
    data: {}
  }).then(res => {
    that.getSrcFlag = false
    that.diaryList = res.result.data.reverse()
    that.likeList = []
    that.diaryList.forEach((item, index) => {
      item.like.forEach(itemSecond => {
        if (itemSecond.openId === that.openId) {
          that.likeList.push(itemSecond.type)
        }
      })
      if (that.likeList.length < index + 1) {
        that.likeList.push('1')
      }
    })
    wx.hideNavigationBarLoading()
    wx.stopPullDownRefresh()
  })
},
// 点赞或取消点赞
toLike (id, type, arr) {
  const that = this
  that.tempObj = {
    id: id,
    type: type,
    like: arr
  }
  wx.getSetting({
    success (res) {
      if (res.authSetting['scope.userInfo']) {
        // 已经授权，可以直接调用 getUserInfo 获取头像昵称
        wx.getUserInfo({
          success: function (res) {
            that.userInfo = res.userInfo
            wx.cloud.callFunction({
              name: 'like',
              data: {
                id: id,
                type: type,
                like: arr,
                name: that.userInfo.nickName
              }
            }).then(res => {
              if (type === '1') {
                tools.showToast('取消点赞成功')
              } else {
                tools.showToast('点赞成功~')
              }
              // getOpenId()方法里会执行一遍获取日记列表
              that.getOpenId()
            })
          }
        })
      } else {
        that.showDialog = true
      }
    }
  })
},
// 授权获取用户信息
login (e) {
  const that = this
  console.log(that.tempObj, e)
  if (e.target.errMsg === 'getUserInfo:ok') {
    wx.getUserInfo({
      success: function (res) {
        that.userInfo = res.userInfo
        wx.cloud.callFunction({
          name: 'like',
          data: {
            id: that.tempObj.id,
            type: that.tempObj.type,
            like: that.tempObj.like,
            name: that.userInfo.nickName
          }
        }).then(res => {
          if (that.tempObj.type === '1') {
            tools.showToast('取消点赞成功')
          } else {
            tools.showToast('点赞成功~')
          }
          // getOpenId()方法里会执行一遍获取日记列表
          that.getOpenId()
        })
      }
    })
  }
  that.showDialog = false
}
```

2、首页获取日记列表，在存储日记到数据库集合的时候我会在每条日记对象中添加一个like属性，like默认是一个空数组；

3、当用户点赞或取消点赞时，组件data中tempObj属性会临时存储三个参数：
   ①、对应日记的_id；
   ②、用户操作的类型是点赞（点赞是‘2’）或是取消点赞（取消点赞是‘1’）；
   ③、对应日记的like数组；

4、通过小程序api的wx.getSetting({})来判断用户是否已经授权。如果授权了获取用户信息，未授权则弹框引导用户点击确认按钮去手动授权；

5、授权成功后，拿到用户信息，我们开始调用点赞或取消点赞相关的云函数，如下：

```javascript
const cloud = require('wx-server-sdk')
cloud.init()
const db = cloud.database()
exports.main = async (event, context) => {
  try {
    // wxContext内包含用户的openId
    const wxContext = cloud.getWXContext()
    // 定义空数组
    let arr = []
    if (event.like && event.like.length > 0) {
      // 让定义的数组等于用户操作的当前日记下的like数组
      arr = event.like
      // 定义一个计数变量
      let count = 0
      // 循环遍历，当openId相同时替换like数组中的相同项，并存储对应的type
      arr.forEach((item, index) => {
        if (item.openId === wxContext.OPENID) {
          count++
          arr.splice(index, 1, {
            openId: wxContext.OPENID,
            type: event.type,
            name: event.name
          })
        }
      })
      // 当计数变量为0时，说明在这条日记中，like数组中未存储过此用户，直接push此用户并存储type
      if (count === 0) {
        arr.push({
          openId: wxContext.OPENID,
          type: event.type,
          name: event.name
        })
      }
    } else {
      // 如果此条日记like数组本身就为空，直接push当前用户并存储type
      arr.push({
        openId: wxContext.OPENID,
        type: event.type,
        name: event.name
      })
    }
    // 通过云开发操作数据库的相关api,即update通过_id来更新集合中某条数据
    return await db.collection('diary').doc(event.id).update({
      data: {
        like: arr
      }
    })
  } catch (e) {
    console.error(e)
  }
}
```

6、相关云函数操作说明都写在上面的注释里，有不清楚的欢迎留言，由于点赞功能未更新到线上（原因是因为审核不通过），想体验的同学也可以加本人微信号，提供体验权限。

------

## 发表心情

### 效果图

![](https://puui.qpic.cn/vupload/0/20190813_1565663979369_jeix851nxec./0)

### 讲解 

1、通过首页右下角的发布加号，进入发布心情页；

2、地址等相关信息是从首页通过路由带过来的；

3、下面重点讲解下关于上传图片到云存储并写入数据库的操作过程，内容如下：

```javascript
upload () {
  const that = this
  wx.chooseImage({
    count: 1,
    sizeType: ['compressed'], // 可以指定是原图还是压缩图，默认二者都有
    sourceType: ['album', 'camera'], // 可以指定来源是相册还是相机，默认二者都有
    success: function (res) {
      wx.showLoading({
        title: '上传中'
      })
      // 返回选定照片的本地文件路径列表，tempFilePath可以作为img标签的src属性显示图片
      let filePath = res.tempFilePaths[0]
      const name = Math.random() * 1000000
      const cloudPath = 'picture/' + name + filePath.match(/\.[^.]+?$/)[0]
      wx.cloud.uploadFile({
        cloudPath, // 云存储图片名字
        filePath // 临时路径
      }).then(res => {
        console.log(res)
        wx.hideLoading()
        that.imgUrl = res.fileID
      }).catch(e => {
        console.log('[上传图片] 失败：', e)
      })
    }
  })
},
save () {
  const that = this
  if (that.desc) {
    that.getSrcFlag = false
    const db = wx.cloud.database()
    const diary = db.collection('diary')
    if (that.imgUrl === '../../static/images/default.png') {
      that.imgUrl = '../../static/images/default.jpg'
    }
    diary.add({
      data: {
        desc: that.desc,
        time: tools.getNowFormatDate(),
        url: that.imgUrl,
        name: that.name,
        weather: that.weather,
        address: that.address,
        like: []
      }
    }).then(res => {
      wx.reLaunch({
        url: '/pages/index/main'
      })
    }).catch(e => {
      console.log(e)
    })
  } else {
    tools.showToast('写点什么吧~')
  }
}
```

4、这里的cloudPath可以自己定义，存储到云中是这样的：

![](https://puui.qpic.cn/vupload/0/20190813_1565664063839_gwtjv4viv0v./0)

5、我们通过组件data中的imgUrl临时存储手动上传的图片路径，最终通过保存按钮一起存储到云数据库，存如到数据库是这样的：

![](https://puui.qpic.cn/vupload/0/20190813_1565664109045_55upawe4xig./0)

------



## 日记详情页

### 详情页效果图

![](https://puui.qpic.cn/vupload/0/20190813_1565664156137_097x5upwjtk6./0)

### 讲解 

1、详情就不过多讲解，这里利用了一些小程序api，比方说动态改变头部标题，每次进入动态随机改变顶部标题背景，点赞数也是从首页带过来的；

------

## 访客页

### 效果图

1、授权前

![](https://puui.qpic.cn/vupload/0/20190813_1565664215248_qaqn4fn01nm./0)

2、授权后

![](https://puui.qpic.cn/vupload/0/20190813_1565664252933_a51qi0vb7sn./0)

------

## 总结

> 云开发虽然能用，但对于大型项目个人还是不推荐，从图片和数据加载这块的效果来看，传统服务端拿到的数据明显要快很多，既然有这么一个免费的工具，我想感兴趣的同学可以利用起来，玩点小demo，新花样。

# 源码链接
[https://gitee.com/roberthuang123/diary](https://gitee.com/roberthuang123/diary)
