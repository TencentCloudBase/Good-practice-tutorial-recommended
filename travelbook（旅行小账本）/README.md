# 旅行小账本
# IDE
- [微信开发者工具](https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html?t=201822)
- [VSCode](https://code.visualstudio.com/Download)</br>
小程序开发必然少不了微信开发者工具，再加上其对云开发的全面支持，再好不过的开发利器。但熟悉微信开发者工具的朋友们应该知道，它不支持[Emmet缩写语法](https://www.cnblogs.com/cnjava/p/3225174.html)，并且wxml的属性值默认用单引号表示(强迫症表示很难受)。
而VSCode很好的补足了微信开发者工具的不足之处，并且支持多元化[插件开发](https://juejin.im/post/5a08d1d6f265da430f31950e)，轻量好用。</br>
所以这里推荐采用微信开发者工具+VSCode配合开发。微信开发者工具负责调试、模拟小程序运行情况，VSCode负责代码编辑工作。二者各司其职，会使开发更加的高效、便捷
# 总体架构
该项目基于小程序云开发，使用的模板是[云开发快速启动模板](https://cloud.tencent.com/developer/article/1345310)</br>
由于是个全栈项目，前端使用小程序所支持的wxml + wxss + js开发模式，命名采用[BEM](https://juejin.im/post/5bb4678a5188255c980be9d2)命名规范。后台则是借助云数据库+云储存进行数据管理。
#### 项目总体结构
```javascript
|-travelbook  项目名
    |-cloudfunctions  云函数模块
        |-deleteItems 级联删除--云函数
        |-getTime     获取时间--云函数
    |-miniprogram  项目模块
        |-components  自定义组件
            |-accountCover  账本封面组件
            |-spendDetail   支出细节组件
        |-pages  页面
            |-accountBooks     总账本页
            |-accountCalendar  账本日历页
            |-accountDetail    支出细节页
            |-accountList      支出明细页
            |-accountPage      选定账本页
            |-editAccount      账本编辑页
            |-index            首页
        |-vant-weapp   有赞vant框架组件库
            |-···      系列组件...
        app.js         全局js
        app.json       全局json配置
        app.wxss       全局wxss
```        
# 逆向工程
在做该小程序之前，有必要进行项目的逆向工程，进一步解构每一个页面，从而深入了解这款小程序的交互细节。那么现在我假设自己为腾讯旅游的产品设计师，在绘制完界面原型后，撰写了相应的交互文档。当然解构过程中可能有些细节处理并没有那么仔细到位...</br>
以下是我绘制的界面原型
![](https://puui.qpic.cn/vupload/0/20190618_1560841685577_ez53lrmq4ji.jpeg/0)
![](https://puui.qpic.cn/vupload/0/20190618_1560841745416_ilo7zykytkc.jpeg/0)
![](https://puui.qpic.cn/vupload/0/20190618_1560841810370_zxgzdholi6j.png/0)
接下来对每个页面的细节进行解构，并完成简单的wxml结构
![](https://puui.qpic.cn/vupload/0/20190618_1560841863494_r2jb1rjjxz.png/0)
```javascript
<!--switchList使用定位布局-->
<view bindtap="switchList" class="list"></view>

<!--newAccount使用flex布局-->
<view class="newAccount" bindtap="createNewAccount">
    <view class="desc">旅行中的每一笔开支都有独特的意义！</view>
    <image src="{{}}"></image>
    <view class="title">创建一个新账本</view>
</view>
```
![](https://puui.qpic.cn/vupload/0/20190618_1560841983381_9t2npq4e0lc.jpeg/0)
```javascript
<!--整体用flex + 百分比布局-->
<input type="text" class="accuntName" placeholder="旅行账本名称" bindinput="getInput" />
  
<van-panel title="选择封面" class="panel">
    <van-row class="imageBox">
        <!--使用wx:for遍历数据库账本图片信息-->
        <van-col span="8" class="imgCol" bindtap="selectThis">
            <image class="select" src="{{}}"></image>
        </van-col>
        
        <van-col span="8">
            <view class="addBox" bindtap="useMore">更多封面</view>
        </van-col>
    </van-row>
</van-panel>

<button type="primary" bindtap="save">保存</button>
<button type="warn" bindtap="delete">删除</button>
```
![](https://puui.qpic.cn/vupload/0/20190618_1560842059815_nfusyuuau7.png/0)
```javascript
<view class="accountDesc" bindtap="viewDetail">
    <!--使用wx:for遍历数据库账本信息-->
    <view class="accountName">
        <view>{{}}</view>
        <view class="accountTime">{{}}</view>
    </view>
    
    <!--绝对定位-->
    <image class="updateImg" catchtap="editAccount" src="{{}}"></image>
</view>
```
![](https://puui.qpic.cn/vupload/0/20190618_1560842184322_kfrsl4rpv6p.png/0)
```javascript
<!--switchList使用定位布局-->
<view bindtap="switchList" class="list"></view>

<view class="account__list-year">{{}}</view>
<view class="account__list-new account__list-public" bindtap="createNewAccount">
    <!--日期小圆点-->
    <view class="account__list-point"></view>
    <view class="account__list-time">{{}}</view>
    <image src="{{}}"></image>
    <view class="account__list-title">创建一个新账本</view>
</view>

<!--使用wx:for遍历数据库账本信息-->
<view class="account__list-item account__list-public" bindtap="viewDetail">
    <!--日期小圆点-->
    <view class="account__list-point"></view>
    <image src="{{}}" mode="aspectFill"></image>
    <view class="account__list-name">{{}}</view>
    <view class="account__list-time">{{}}</view>
    <image class="account__list-update" catchtap="editAccount" src="{{}}"></image>
 </view>
 ```
 ![](https://puui.qpic.cn/vupload/0/20190618_1560842250801_o1qdpeigrhr.jpeg/0)
 ```javascript
 <view class="account__spend">
    <image bindtap="getCalendar" class="account__spend-calendar" src="{{}}"></image>
    <view class="account__spend-text">
        <view class="account__spend-total">总花费(元)</view>
        <view class="account__spend-num">{{}}</view>
    </view>
    <image bindtap="accountAnalyze" class="account__spend-detail" src="{{}}"></image>
</view>

<view class="account__show-time">今天</view>
    <view class="account__show-detail">
        <view class="account__show-income account__show-public">
        <view class="account__show-title">收入(元)</view>
        <text class="account__show-in">+{{}}</text>
    </view>
    <view class="account__show-spend account__show-public">
        <view class="account__show-title">支出(元)</view>
        <text class="account__show-out">-{{}}</text>
    </view>
</view>

<!--使用wx:for遍历数据库账本信息-->
<view class="account__show-items-spend">
    <view>
        <image src="{{}}"></image>
    </view>
    <text>{{}}</text>
    <text class="account__show-items-money">{{}}</text>
</view>
```
![](https://puui.qpic.cn/vupload/0/20190618_1560842373409_km8tjmyscql.png/0)
```javascript
<!--日历使用极点日历的插件-->
<!--json中做配置-->
"usingComponents": {
    "calendar": "plugin://calendar/calendar"
}

<!--js改变样式-->
days_style.push({
  month: 'current',
  day: new Date().getDate(),
  color: 'white',
  background: '#e0a58e'
})

<!--wxml中引用-->
<calendar weeks-type="cn" cell-size="50" next="{{true}}" prev="{{true}}"
    show-more-days="{{true}}" calendar-style="demo6-calendar"
    header-style="calendar-header"board-style="calendar-board" active-type="rounded" 
    lunar="true" header-style="header"calendar-style="calendar"days-color="{{days_style}}">
</calendar>
```
![](https://puui.qpic.cn/vupload/0/20190618_1560842442806_i03eiats4d.png/0)
```javascript
<!--顶栏日期及收支结构-->
<view class="account__title">
    <text class="account__title-time">{{}}</text>
    <text class="account__title-spend">支出{{}}元 收入{{}}元</text>
</view>

<!--收支细节结构 使用flex弹性布局-->
<view class="account__detail">
    <image src="{{}}"></image>
    <view class="account__detail-name">{{}}</view>
    <view class="account__detail-money">{{}}</view>
</view>
```
![](https://puui.qpic.cn/vupload/0/20190618_1560842599626_eqbucxicxre.png/0)
```javascript
<!--使用vant框架的van-tabs组件-->
<!--并封装自定义组件复用收支页，自定义组件后面会详细说明-->
<van-tabs active="{{ active }}" bind:change="onChange">
  <van-tab title="支出">
    <spendDetail detail="{{detail}}" accountKey="{{accountKey}}"></spendDetail>
  </van-tab>
  <van-tab title="收入">
    <spendDetail detail="{{income}}" accountKey="{{accountKey}}"></spendDetail>
  </van-tab>
</van-tabs>
```
# 云开发
在做完逆向工程的解构，页面基础结构基本搭建完成。但页面依旧是静态的，需要数据来填充。所以第二步就是数据库的设计。而小程序的云控制台恰好提供了数据的操作功能，为数据驱动提供基石。
![](https://puui.qpic.cn/vupload/0/20190618_1560842693321_2iryp54j1mr.png/0)
## 云数据库设计
云数据库是一种NoSQL数据库。每一张表是一个集合。值得注意的是在设计数据库时，`_id` 和`_openid`这两个字段需要带上。`_id`是表的主键，而`_openid`是用户标识，每个用户都有不同的`_openid`，可区分不同用户。</br>
以下是项目中的数据表设计
```javascript
cover_photos 账本封面表  用于存储创建账本时需要的封面信息
    - _id
    - _openid
    - cover_index 封面索引
    - cover_url   封面url
    - isSelected  封面是否选中
```
```javascript
accounts 账本表   用于存储用户创建的账本
    - _id
    - _openid
    - accountKey  账本唯一标识
    - coverUrl    账本封面
    - i           账本索引
    - inputValue  账本名字
    - now         账本创建时间
    - spend       账本总花费
```
```javascript
account_detail 支出类型表   用于存储消费类型
    - _id
    - _openid
    - detail       类型细节
    - pic_index    消费类型索引
    - pic_url      未点击时的图片
    - pic_url_act  点击后的图片
    - type         消费类型
```
```javascript
account_income 收入类型表   用于存储收入类型
    - _id
    - _openid
    - pic_index    收入类型索引
    - pic_url      未点击时的图片
    - pic_url_act  点击后的图片
    - type         收入类型
```
```javascript
spend_items   消费明细表
    - _id
    - _openid
    - accountKey   账本唯一标识
    - address      消费地点
    - desc         消费描述
    - fullDate     消费时间
    - money        消费金额
    - pic_type     消费类型
    - pic_url      消费类型图片
```    
## 云储存管理
这是个非常实用的板块。类似于[百度云盘](https://pan.baidu.com/)，它提供了文件存储、上传与下载功能。
![](https://puui.qpic.cn/vupload/0/20190618_1560843125594_leagaa09yfi.png/0)
除此之外，它还会将你所上传的资源自动进行压缩操作，并生成一个地址供你引用。该项目中的一些图片资源就是存在于此，然后在云数据库的字段中引用这些资源地址即可，十分方便，不必在本地存储，占用小程序内存。
![](https://puui.qpic.cn/vupload/0/20190618_1560843165699_eczla5i3ixl.png/0)
## 云函数设计
[云函数](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/guide/functions.html)简单来说就是在云后端(Node.js)运行的代码，本地看不到这些代码的执行过程，全封闭式只暴露接口供本地调用执行，本地只需等待云端代码执行完毕后返回结果。这也是[面向接口编程](https://www.cnblogs.com/bobodeboke/p/5733422.html)的思想体现。</br>
项目中的云函数设计
![](https://puui.qpic.cn/vupload/0/20190618_1560843311997_6uz9ccu5erg.png/0)
```javascript
// getTime  获取当前时间并格式化为 yyyy-mm-dd

// 云函数入口文件
const cloud = require('wx-server-sdk')

// 初始化云函数
cloud.init()

// 云函数入口函数
exports.main = async (event, context) => {
  var date = new Date()
  var seperator1 = "-"
  var year = date.getFullYear()
  var month = date.getMonth() + 1
  var strDate = date.getDate()
  if (month >= 1 && month <= 9) {
    month = "0" + month
  }
  if (strDate >= 0 && strDate <= 9) {
    strDate = "0" + strDate
  }
  // 格式化当前时间
  var currentdate = year + seperator1 + month + seperator1 + strDate
  return currentdate
}
```
```javascript
// deleteItems  批量删除，云数据库的批量删除只允许在云函数中执行

// 云函数入口文件
const cloud = require('wx-server-sdk')

// 初始化云函数
cloud.init()

// 连接云数据库
const db = cloud.database()
const _ = db.command


// 云函数入口函数
exports.main = async (event, context) => {
  try {
    return await db.collection('spend_items')
      .where({
        accountKey: event.accountKey
      })
      .remove()
  } catch (e) {
    console.error(e)
  }
}
```
## MVVM
界面有了，数据有了。万事俱备，只欠东风！所以下一步就是[MVVM](https://www.liaoxuefeng.com/wiki/1022910821149312/1108898947791072)的设计。小程序本质就是基于MVVM所设计的，在MVVM的世界里，数据是灵魂，一切都由数据来驱动。
## 账本页显示
账本页有两种显示的风格，左上角的按钮可以来回切换风格，下拉可刷新页面，显示accounts数据表中存储的账本信息。显示时有个小细节，需要根据创建的时间先后来显示，越晚创建的越先显示。
```javascript
// 页面数据设计, 在wxml中使用{{}}符号引用数据，数据就动态显示到了页面上
data: {
    isList: false, // 转换页面风格的标识 true为竖向风格 false为横向风格
    accounts: [],  // 存储查询的账本数据
    now: null,     // 存储当日时间
    year: null     // 存储年份
}

 // 转换显示风格
switchList() {
    // 设置页面风格样式
    let isList = !this.data.isList
    this.setData({
      isList
    })
    wx.setStorage({
      key: "isList",
      data: isList
    })
}

// 获取页面风格转换标识
var isList = wx.getStorageSync('isList')
    
// 查询账本
db.collection('accounts')
  .get({
    success: res => {
      this.setData({
        accounts: res.data.reverse(),  // 反转数组，优先显示创建早的账本
        isList
      })
      wx.hideLoading()
    }
  })

// 调用云函数接口 获取当前日期
wx.cloud.callFunction({
    // 云函数接口名就是创建的云函数名字，这里是'getTime'
    name: 'getTime',
    success: (res) => {
    let year = res.result.split('-')[0]
    this.setData({
      now: res.result,
      year
    })
    },
    fail: console.error
})
```
## 账本页增删改
![](https://puui.qpic.cn/vupload/0/20190618_1560843593146_wui2kg9i3s.gif/0)
账本页通过调用相应的云数据库API，可进行一系列的增删改操作。值得一提的是，修改时需要表单回显，删除时需要级联删除。因为一个账本中有许多收支情况，spend_items表就是进行收支记录，所以删除账本时需要级联删除对应的spend_items表中的收支信息。</br>
一些重要的逻辑
- 封面单选逻辑
```javascript
data: {
    images: [],      // 封面数组
    selectImg: null, // 选择其它封面
    isSelected: {},  // 选中的图片
    inputValue: '',  // 账本名字
    now: null,       // 当前时间
    account: {}      // 传入账本信息
}

  // 单选逻辑 通过构造{'0': isSelected}来实现
selectThis(e) {
    let index = e.currentTarget.dataset.index
    let coverUrl = e.currentTarget.dataset.coverurl
    let is = this.data.isSelected[index]
    let obj = {
        coverUrl
    }
    // obj[index] 属性动态改变
    obj[index] = !is
    obj.i = index
    this.setData({
        isSelected: obj
    })
}
```
- 表单回显逻辑
```javascript
// 页面加载时先通过对应的accountKey, 得到回显信息
let { i, id, value, url, accountKey } = options
photos.get({
    success: res => {
    this.setData({
      images: res.data,
      account: {
        id,
        value,
        url,
        i,
        accountKey
      },
      isSelected: obj
    })
    wx.hideLoading()
  }
})
// 修改
save() {
    let { id } = this.data.account
    let { i, coverUrl, value } = this.data.isSelected
    // 若没修改 则为之前的value
    let inputValue = this.data.inputValue || value
    
    db.collection('accounts')
      .doc(id)
      .update({
        data: {
            inputValue,
            coverUrl,
            i
        }
    })
}
```
- 级联删除逻辑
```javascript
db.collection('accounts')
    .doc(this.data.account.id)
    .remove()
    .then(() => {
      wx.hideLoading()
      wx.showToast({
        title: '删除成功'
      })
      setTimeout(() => {
        wx.reLaunch({
          url: '../accountBooks/accountBooks'
        })
      }, 400)
    })
  // 调用deleteItems云函数, 传入对应accountKey主键, 通过云函数批量删除
  wx.cloud.callFunction({
    name: 'deleteItems',
    data: {
      accountKey
    }
  })
  ```
  ## 账本页收支
![](https://camo.githubusercontent.com/75d424fb932f00f94c78df0f5683975d3d0419ad/68747470733a2f2f376136382d7a68682d636c6f75642d6237613161392d313235373839323938382e7463622e71636c6f75642e6c612f74726176656c626f6f6b2545362542432539342545372541342542416769662f2545382542342541362545362539432541432545362539342542362545362539342541462e6769663f7369676e3d323365613764363165643938643936363036316664666139393765303961303726743d31353432363130373537)</br>
因为收入与支出页面基本类似，所以使用自定义组件封装，可以复用。
```javascript
// 封装spendDetail组件
// 注册组件
properties: {
    detail: {
      type: Object
    },
    accountKey: {
      type: Number
    },
    isSpend: {
      type: Boolean
    }
}

// 引用组件
<van-tab title="支出">
    <spendDetail detail="{{detail}}" accountKey="{{accountKey}}" isSpend="{{isSpend}}"></spendDetail>
  </van-tab>
  <van-tab title="收入">
    <spendDetail detail="{{income}}" accountKey="{{accountKey}}" isSpend="{{isSpend}}"></spendDetail>
</van-tab>
```
收入与支出类型icon选择使用两个view来存放，通过选择不同类型，跳转不同的icon
```javascript
// js
data: {
    address: '',
    money: 0,
    desc: '',
    selectPicIndex: 0,
    selectIndex: 0
}
// 选择消费类别
selectSpend(e) {
  let { index } = e.currentTarget.dataset
  let { selectPicIndex } = this.data
  selectPicIndex = index
  this.setData({
    selectPicIndex
  })
},

// 选择消费类别中的细节
selectSpendDetail(e) {
  let { index } = e.currentTarget.dataset
  let { selectIndex } = this.data
  selectIndex = index
  this.setData({
    selectIndex
  })
}

// wxml
// 消费类型
<view class="expense">
  <block wx:for="{{detail}}" wx:key="index">
    <view class="expense__type" bindtap="selectSpend" data-index="{{index}}">
      <block wx:if="{{selectPicIndex == item.pic_index}}">
        <view class="expense__type-icon" style="background-color: #e64343">
          <image src="{{item.pic_url_act}}"></image>
        </view>
      </block>
      <block wx:else>
        <view class="expense__type-icon">
          <image src="{{item.pic_url}}"></image>
        </view>
      </block>
      <view class="expense__type-name">{{item.type}}</view>
    </view>
  </block>
</view>

// 消费子类型
<view class="detail">
  <block wx:for="{{detail[selectPicIndex].detail}}" wx:key="index">
    <view class="detail__type" bindtap="selectSpendDetail" data-index="{{index}}">
      <image class="detail__type-icon" src="{{item.detail_url}}"></image>
      <block wx:if="{{selectIndex == item.detail_index}}">
        <view class="detail__type-name" style="color: #f86319; border-bottom: 1rpx solid #f86319;">
          {{item.detail_type}}
        </view>
      </block>
      <block wx:else>
        <view class="detail__type-name" style="border-bottom: 1rpx solid #e4e2e2;">
          {{item.detail_type}}
        </view>
      </block>
    </view>
  </block>
</view>
```
## 账本页明细
![](https://puui.qpic.cn/vupload/0/20190618_1560844085773_tj4zundo26n.gif/0)</br>
因为收支明细中需要显示每一天的消费信息，所以需要将数据表中的数据通过时间来分类，分成若干个数组，页面从而使用wx:for来遍历这些数组。在显示之前，首先需要判断有无收支信息。
```javascript
// 通过时间分类算法  {} => [ [{时间1}], [{时间2}], [{时间3}] ]
arr.forEach(item => {
  if (!_this.isExist(item.fullDate, dateArr)) {
    dateArr.push([item])
  } else {
    dateArr.forEach(res => {
      if (res[0].fullDate == item.fullDate) {
        res.push(item)
      }
    })
  }
})

// 使用map 方法构造 [{}, {}, {}, ...] 类型数组
dateArr = dateArr.map((item) => {
  let spend = 0
  let income = 0
  item.forEach(res => {
    if (res.money > 0) {
      spend += res.money
    } else {
      income += (-res.money)
    }
  })
  return {
    item,
    spend,
    income
  }
})

// 判断自身是否存在数组中
isExist(item, arr) {
    for (let i = 0; i < arr.length; i++) {
      if (item == arr[i][0].fullDate)
        return true
    }
    return false
  }
  ```
  以上是小程序中比较复杂的逻辑实现。
# 源码链接
[https://github.com/FightingHao/travelbook](https://github.com/FightingHao/travelbook)
