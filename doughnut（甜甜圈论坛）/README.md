# 甜甜圈论坛
# 前言
>虽然不会后台开发，但是也想自己做项目，正好云开发出现了。开发者可以使用云开发开发微信小程序、小游戏，无需搭建服务器，即可使用云端能力。
社区作为一个交流的平台，可以通过发布自己、别人喜欢的文字、图片的方式进行交流分享。
刚学完云开发，正好可以用社区小程序项目练练手~

#【社区小程序】功能实现

>**首页【广场】**
- 显示用户发布的内容
- 管理员发布的一些教程

>**消息【发布】**
- 发布图文
- 水平图片的滑动显示

>**个人中心【我的】**
- 显示用户的登录信息
- 用户的收藏列表
- 发布历史
- 邀请好友
- 产品意见

![](https://puui.qpic.cn/vupload/0/20190724_1563958468638_2yxelmego95.gif/0)

# 一、首页【广场】
- 显示用户发布的内容
- 管理员发布的一些教程

## 实现的效果

![](https://puui.qpic.cn/vupload/0/20190724_1563958563706_3g8f5plne5x.png/0)

## 实现要点
### 1.WXML 不同类别数据的显示
>通过 `if-elif-else` 实现，在 `wxml` 文件中通过 `<block></block>` 渲染，因为它仅仅是一个包装元素，不会在页面中做任何渲染，只接受控制属性。也就是说可以通过属性来控制页面是否要渲染这部分的内容，可以减少页面渲染时间。

### 2.云开发数据的获取
>先开通云开发功能 ，参考官方文档，然后在创建项目的时候勾选上 `使用云开发模板`（看个人吧，我直接使用后点击项目中的 `login` ）就可以获取到用户的 `oppenid` ，之后就可以使用云数据库了。

![](https://puui.qpic.cn/vupload/0/20190724_1563958739452_uz7bpxstqdr.jpeg/0)

- 云开发登录：

![](https://puui.qpic.cn/vupload/0/20190724_1563958781526_55ph7d44fsl.jpeg/0)

- 云数据的获取

```javascript
/**
   * 生命周期函数--监听页面加载
   */
  onLoad: function(options) {
    console.log('onload');
    this.getData(this.data.page);
    },
  /**
   * 获取列表数据
   * 
   */
  getData: function(page) {
    var that = this;
    console.log(="page--->" + page);
    const db = wx.cloud.database();
    // 获取总数
    db.collection('topic').count({
      success: function(res) {
        that.data.totalCount = res.total;
      }
    })
    // 获取前十条
    try {
      db.collection('topic')
        .where({
          _openid: 'oSly***********vU1KwZE', // 填入当前用户 openid
        })
        .limit(that.data.pageSize) // 限制返回数量为 10 条
        .orderBy('date', 'desc')
        .get({
          success: function(res) {
            // res.data 是包含以上定义的两条记录的数组
            // console.log(res.data)
            that.data.topics = res.data;
            that.setData({
              topics: that.data.topics,
            })
            wx.hideNavigationBarLoading();//隐藏加载
            wx.stopPullDownRefresh();

          },
          fail: function(event) {
            wx.hideNavigationBarLoading();//隐藏加载
            wx.stopPullDownRefresh();
          }
        })
    } catch (e) {
      wx.hideNavigationBarLoading();//隐藏加载
      wx.stopPullDownRefresh();
      console.error(e);
    }
  },
```

- 云数据的添加：

```javascript
/**
   * 保存到发布集合中
   */
  saveDataToServer: function(event) {
    var that = this;
    const db = wx.cloud.database();
    const topic = db.collection('topic')
    db.collection('topic').add({
      // data 字段表示需新增的 JSON 数据
      data: {
        content: that.data.content,
        date: new Date(),
        images: that.data.images,
        user: that.data.user,
        isLike: that.data.isLike,
      },
      success: function(res) {
        // res 是一个对象，其中有 _id 字段标记刚创建的记录的 id
        // 清空，然后重定向到首页
        console.log("success---->" + res)
        // 保存到发布历史
        that.saveToHistoryServer();
        // 清空数据
        that.data.content = "";
        that.data.images = [];

        that.setData({
          textContent: '',
          images: [],
        })

        that.showTipAndSwitchTab();

      },
      complete: function(res) {
        console.log("complete---->" + res)
      }
    })
  },
```

### 3.数据列表的分页
>主要就是定义一个临时数组存放加载上来的数据，然后通过传递给对象，最后传递到布局中去。

```javascript
/**
   * 页面上拉触底事件的处理函数
   */
  onReachBottom: function() {
    var that = this;
    var temp = [];
    // 获取后面十条
    if(this.data.topics.length < this.data.totalCount){
      try {
        const db = wx.cloud.database();
        db.collection('topic')
          .skip(5)
          .limit(that.data.pageSize) // 限制返回数量为 5 条
          .orderBy('date', 'desc')    // 排序
          .get({
            success: function (res) {
              // res.data 是包含以上定义的两条记录的数组
              if (res.data.length > 0) {
                for(var i=0; i < res.data.length; i++){
                  var tempTopic = res.data[i];
                  console.log(tempTopic);
                  temp.push(tempTopic);
                }

                var totalTopic = {};
                totalTopic =  that.data.topics.concat(temp);

                console.log(totalTopic);
                that.setData({
                  topics: totalTopic,
                })
              } else {
                wx.showToast({
                  title: '没有更多数据了',
                })
              }

            },
            fail: function (event) {
              console.log("======" + event);
            }
          })      
      } catch (e) {
        console.error(e);
      }
    }else{
      wx.showToast({
        title: '没有更多数据了',
      })
    }

  },
```
# 二、消息【发布】

- 发布图文
- 水平图片的滑动显示（效果不是很好，可以改为九宫格实现）

#### 发布页面效果如下：

![](https://puui.qpic.cn/vupload/0/20190725_1564019404954_1riz9323fmf.jpeg/0)

#### 分析如何实现

- 导航栏的实现很简单就不说了，可参考我之前的文章
- 重点是中间的 ② 是内容区域
- 区域三是功能操作区

#### 内容区域的实现

- 第一个是文本区域
- 第二个是水平的图片展示区域

在图片的右上角有关闭按钮，这里使用的是 `icon` 组件。

主要的实现代码如下：

```javascript
<view class="content">
  <form bindsubmit="formSubmit">
    <view class="text-content">
      <view class='text-area'>
        <textarea name="input-content" type="text" placeholder="说点什么吧~" placeholder-class="holder" value="{{textContent}}" bindblur='getTextAreaContent'></textarea>
      </view>
    </view>
    <scroll-view class="image-group" scroll-x="true">
      <block wx:for='{{images}}' wx:for-index='idx'>
      <view>
        <image src='{{images[idx]}}' mode='aspectFill' bindtap="previewImg"></image>
        <icon type='clear' bindtap='removeImg'  data-index="{{idx}}" ></icon>
      </view>
      </block>
    </scroll-view>
    <view class='btn-func'>
      <button class="btn-img" bindtap='chooseImage'>选择图片</button>
      <button class="btn" formType='submit'  open-type="getUserInfo">发布圈圈</button>
      <!-- <image hidden=''></image> -->
    </view>
  </form>

</view>
```

布局样式如下：

```javascript
.content {
  height: 100%;
  width: 100%;
}

textarea {
  width: 700rpx;
  padding: 25rpx 0;
}

.text-content {
  background-color: #f3efef;
  padding: 0 25rpx;
}

.image-group {
  display: flex;
  white-space: nowrap;
  margin-top: 30px;
}

.image-group view{
  display: inline-block;
  flex-direction: row;
  width: 375rpx;
  height: 375rpx;
  margin-right: 20rpx;
  margin-left: 20rpx;
  background-color: #cfcccc;
}

.image-group view image{
  width: 100%;
  height: 100%;
  align-items: center;
}

.image-group view icon{
  display: inline-block;
  vertical-align: top;
  position: absolute
}
.
btn-func {
  display: flex;
  flex-direction: column;
  width: 100%;
  position: absolute;
  bottom: 0;
  margin: 0 auto;
  align-items: center;
}

.btn-img {
  width: 220px;
  height: 45px;
  line-height: 45px;
  margin-top: 20px;
  margin-bottom: 20px;
  background-color: rgb(113, 98, 250);
  color: #fff;
  border-radius: 50px;
}

.btn {
  width: 220px;
  height: 45px;
  line-height: 45px;
  background-color: #d50310;
  color: #fff;
  border-radius: 50px;
  margin-bottom: 20px;
}
```

页面布局之后就该从 `js` 中去处理数据了，在 `js` 中主要实现的功能有：

- 文本内容的获取
- 图片的选择
- 图片的阅览
- 图片的删除
- 将结果发布到云数据库中

#### 1.文本内容的获取

```javascript
/**
   * 获取填写的内容
   */
  getTextAreaContent: function(event) {
    this.data.content = event.detail.value;
  },
```

#### 2.图片的选择

```javascript
/**
   * 选择图片
   */
  chooseImage: function(event) {
    var that = this;
    wx.chooseImage({
      count: 6,
      success: function(res) {
        // tempFilePath可以作为img标签的src属性显示图片
        const tempFilePaths = res.tempFilePaths

        for (var i in tempFilePaths) {
          that.data.images = that.data.images.concat(tempFilePaths[i])
        }
        // 设置图片
        that.setData({
          images: that.data.images,
        })
      },
    })
  },
```
#### 3.图片的预览

```javascript
// 预览图片
  previewImg: function(e) {
    //获取当前图片的下标
    var index = e.currentTarget.dataset.index;

    wx.previewImage({
      //当前显示图片
      current: this.data.images[index],
      //所有图片
      urls: this.data.images
    })
  },
```

#### 4.图片的删除

```javascript
/**
   * 删除图片
   */
  removeImg: function(event) {
    var position = event.currentTarget.dataset.index;
    this.data.images.splice(position, 1);
    // 渲染图片
    this.setData({
      images: this.data.images,
    })
  },
```

#### 5.发布内容到数据库中
>数据发布到数据中，需要先开启云开发，然后在数据库中创建集合也就是表之后就是调用数据库的增删改查API即可。

```javascript
/**
   * 添加到发布集合中
   */
  saveToHistoryServer: function(event) {
    var that = this;
    const db = wx.cloud.database();
    db.collection('history').add({
      // data 字段表示需新增的 JSON 数据
      data: {
        content: that.data.content,
        date: new Date(),
        images: that.data.images,
        user: that.data.user,
        isLike: that.data.isLike,
      },
      success: function(res) {
        // res 是一个对象，其中有 _id 字段标记刚创建的记录的 id
        console.log(res)
      },
      fail: console.error
    })
  },
```

# 三、个人中心【我的】
- 【显示用户的登录信息】主要就是调用小程序接口，获取用户的微信公开信息进行展示
- 【用户的收藏列表】获取数据库中的收藏列表进行展示
- 【发布历史】在发布页面，当发布成功将数据存到发布历史表中，需要的时候获取该表的数据进行展示
- 【邀请好友】调用小程序的分享接口，直接分享给微信群，或者个人
- 【产品意见】一个类似于发布页的页面，实现思路和发布页实现是一样的。

## 实现的效果

![](https://puui.qpic.cn/vupload/0/20190725_1564019342057_zq2bs8tfm28.jpeg/0)

## 实现分析

#### 1.要实现的效果

- 在用户进入个人中心，直接弹出获取用户信息弹窗
- 显示圆形的用户头像

#### 2.授权弹窗

>官方获取用户信息文档调整 为优化用户体验，使用 `wx.getUserInfo` 接口直接弹出授权框的开发方式将逐步不再支持。从2018年4月30日开始，小程序与小游戏的体验版、开发版调用 `wx.getUserInfo` 接口，将无法弹出授权询问框，默认调用失败。正式版暂不受影响。

也就是以前的 `wx.getUserInfo` 不直接弹出授权窗口了，而且在新版中调用会直接返回 `fail` ，现在的做法呢就是通过点击一个 `button` 去实现用户授权功能。

文档中说明了有两种方式能够获取用户信息。
- 一个是利用 `<open-data>` 获取公开的用户信息：

```javascript
<open-data type="userNickName" lang="zh_CN"></open-data>
<open-data type="userAvatarUrl"></open-data>
<open-data type="userGender" lang="zh_CN"></open-data>
```

- 另一个是利用 `button` 组件将 `open-type` 指定为 `getUserInfo` 类型：

```javascript
<!-- 需要使用 button 来授权登录 -->
  <button wx:if="{{canIUse}}" open-type="getUserInfo" bindgetuserinfo="bindGetUserInfo">授权登录</button>
  <view wx:else>请升级微信版本</view>
Page({
  data: {
    canIUse: wx.canIUse('button.open-type.getUserInfo')
  },
  onLoad: function() {
    // 查看是否授权
    wx.getSetting({
      success (res){
        if (res.authSetting['scope.userInfo']) {
          // 已经授权，可以直接调用 getUserInfo 获取头像昵称
          wx.getUserInfo({
            success: function(res) {
              console.log(res.userInfo)
            }
          })
        }
      }
    })
  },
  bindGetUserInfo (e) {
  // 获取到用户信息
    console.log(e.detail.userInfo)
  }
})
```

#### 3.<open-data>中实现圆形头像

```javascript
<view class='amountBg'>
  <view class='img'>
    <open-data type="userAvatarUrl"></open-data>
  </view>
  <view class='account'>
    <view class='nick-name'>
      <open-data type="userNickName" lang="zh_CN"></open-data>
    </view>
    <view class='address'>
      <open-data type="userCountry" lang="zh_CN"></open-data>·
      <open-data type="userProvince" lang="zh_CN"></open-data>·
      <open-data type="userCity" lang="zh_CN"></open-data>
    </view>
  </view>
</view>
```

css 样式如下：

```javascript
.amountBg {  
  display: flex;
  flex-direction: row;
  height: 100px;
  background-color: #5495e6;
  align-items: center;
}

.img {  
  overflow: hidden;
  display: block;
  margin-left: 20px;
  width: 49px;
  height: 49px;
  border-radius: 50%;
}

.account {
  width: 70%;
  color: #fff;
  margin-left: 10px;
  align-items: center;
}

.nick-name{
  font-family: 'Mcrosoft Yahei';
  font-size: 16px;
}

.address{
  font-size: 13px;
}
.nav {
  width: 15px;
  color: #fff;
}
```

# 可能存在的一些问题
>- 其他用户发布的内容，有时候显示不出来？ 将数据库的权限设置为全部人可见。
>- 发布内容之后返回首页没有自动刷新？ 在广场首页 onShow 的时候获取数据库的数据进行展示。
>- clone 源码后运行不起来？ 需要在自己的云数据库中创建对应的表。

# 源码链接
[https://github.com/dongxi346/doughnut](https://github.com/dongxi346/doughnut)
