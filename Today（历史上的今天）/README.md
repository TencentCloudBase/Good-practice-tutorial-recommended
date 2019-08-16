# 口袋工具y

## 前言
本项目是一个基于云开发的小程序。

本文选取项目中的一个页面 -- 历史上的今天 来做一个云开发的分享，会涉及云函数和云数据库。

由于是实战项目，关于小程序的基础知识请移步官方文档，本文不再赘述。

## 项目预览
- 微信搜索： `口袋工具y`

- 扫一扫：

![](https://puui.qpic.cn/vupload/0/20190729_1564366553054_j9u2q3dz4l.webp/0)

### 前期遇到的问题
- 数据来源：没有数据，寸步难行呀

### 如何解决数据来源
- 编写爬虫将需要的数据爬取并保存下来

- 找一些提供数据的平台，如阿凡达数据、聚合数据等等。

> 本项目选择第二种方式，并最终选择了聚合数据平台API。

### 项目开始
### 新建项目
- 新建项目，配置好名称、目录、AppID等信息，后端服务选择小程序·云开发，点击新建。

> 关于AppID: 请自行修改为你注册的小程序AppID。

![](https://puui.qpic.cn/vupload/0/20190729_1564366753027_1fgpk1k6p65.webp/0)

- 点击新建即可完成项目初始化，得到一个云开发模板：

![项目目录](https://puui.qpic.cn/vupload/0/20190729_1564366802415_fmnsf3hu0wq.webp/0)

目录结构：

```javascript
+-- cloudfunctions|[指定的环境]  // 存放云函数的目录
   +-- miniprogram                 // 小程序代码编写目录
   |-- README.md                   // 项目描述文件
   |-- project.config.json         // 项目配置文件
```   

### 新建云开发环境
- 点击左上角菜单项 `云开发`

![](https://puui.qpic.cn/vupload/0/20190729_1564366965553_l1ghuivu2vb.webp/0)

- 点击创建资源环境，环境名称及环境ID请自行设置：

![](https://puui.qpic.cn/vupload/0/20190729_1564367016585_u61awte3nbs.webp/0)

- 点击确定即可完成创建

### 编写云函数
#### 1. 新建云函数

> 在目录 `cloudfunctions` 上右键
新建云函数，填入新建云函数的名称（如todayInHistory）
回车或失去焦点即会自动创建并上传。

#### 2. 安装依赖
云函数目前执行环境仅支持node，所以需要使用js来编写云函数的逻辑。
在控制台中进入该云函数的目录，执行

```javascript
npm i -S axios
```

> 本项目使用axios来执行请求的发送，可以使用其他如request-promise等等的库来替换

### 3. 编写云函数
- 新建 `config.js` 文件，添加代码如下：

```javascript
exports.key = YOUR_JUHE_KEY // 在聚合数据平台申请的key
exports.baseUrl = 'http://v.juhe.cn/todayOnhistory/queryEvent.php'
```

- 打开 ·index.js· 文件，编写代码：

```javascript
// 云函数入口文件
const cloud = require('wx-server-sdk')
const axios = require('axios')

cloud.init()
const db = cloud.database()

// 聚合数据
const { baseUrl, key } = require('./config')

// 云函数入口函数
exports.main = async(event, context) => {
  const {
    month,
    day
  } = event
 
  const resp = await axios.get(baseUrl, {
    params: {
      key,
      date: `${month}/${day}`
    }
  }).then(res => {
    return res.data
  })

  return resp.result
}
```

### 编写页面
#### 1. 新建页面
在开发小程序的过程中，新建一个页面是很常见的操作，有两个非常方便的方式：

- 在 `app.json` 文件中，在pages项添加我们需要的页面路径，直接保存即可。如：

```javascript
"pages": [
  "pages/today-in-history/index"
]
```

- 在 `pages` 目录下新建目录 `today-in-history` ，在新建的目录上 `右键` -> `新建page` ， 填入名称如`index` , 回车即可完成页面下四个文件的创建

#### 2. 编写 `index.wxml`

```javascript
<view class="container">
  <view class="header full-width">
    <view>{{year}}年{{month}}月{{day}}日</view>
  </view>
  <view class="content full-width">
    <view class="list-view">
      <block wx:for="{{list}}" wx:key="index">
          <view class="item-title">{{item.title}}</view>
          <view class="item-date">{{item.date}}</view>
      </block>
    </view>
  </view>
</view>
```

#### 3. 编写 `index.js`


```javascript
// pages/today-in-history/index.js
Page({

  /**
   * 页面的初始数据
   */
  data: {
    year: 1990,
    month: 1,
    day: 1,
    list: []
  },

  /**
   * 生命周期函数--监听页面加载
   */
  onLoad: function() {
    const now = new Date();
    const year = now.getFullYear();
    const month = now.getMonth() + 1;
    const day = now.getDate();
    this.setData({
      year,
      month,
      day
    });
    this.doGetList();
  },

  /**
   * 执行数据获取
   */
  doGetList: function() {
    const {
      month,
      day
    } = this.data;
    wx.cloud.callFunction({
        name: 'todayInHistory',
        data: {
          month,
          day
        }
      }).then(res => {
        let list = res.result.reverse();
        this.setData({
          list
        });
      })
      .catch(console.error)
  }
})
```

#### 4. 编写 `index.wxss`


```javascript
/* pages/today-in-history/index.wxss */
.container {
  padding-bottom: 20rpx;
  background-color: #E8D3A9;
}

.header {
  display: flex;
  justify-content: space-around;
  align-items: center;
  height: 80rpx;
  color: #FFF;
}

.content {
  flex: 1;
}

.list-view {
  height: 100%;
  display: flex;
  flex-direction: column;
  padding: 0 20rpx;
}

.list-item {
  display: flex;
  flex-direction: column;
  border-radius: 10rpx;
  padding: 16rpx 0;
  box-sizing: border-box;
  margin-top: 20rpx;
  background-color: #fff;
  text-align: center;
  box-shadow: 1px 1px 5px 1px rgb(207, 207, 207);
}

.item-title {
  font-size: 36rpx;
  padding: 10rpx 16rpx;
  color: #262626;
  line-height: 48rpx;
}
```

#### 5. 效果预览
到这里我们完成了 `历史上的今天` 的列表页，效果如下：

![](https://puui.qpic.cn/vupload/0/20190729_1564367748553_3nhms9eb29m.webp/0)

### 添加日期选择器
#### 1. 引入 vantweapp
项目中使用 wantweapp 的部分组件

- 安装

```javascript
# npm
  npm i vant-weapp -S --production

  # yarn
  yarn add vant-weapp --production
```

- 构建npm

点击开发者工具菜单项 `工具` -> `构建npm`
程序将自动构建已安装的依赖

#### 2. 在app.json引入组件
```javascript
 "usingComponents": {
    "van-datetime-picker": "/miniprogram_npm/vant-weapp/datetime-picker/index",
    "van-popup": "/miniprogram_npm/vant-weapp/popup/index",
    "van-toast": "/miniprogram_npm/vant-weapp/toast/index"
  }
```

#### 3. 修改 index.wxml
添加下面的代码
```javascript
<view class="full-width">
  <van-popup show="{{ show }}" position="bottom">
    <van-datetime-picker
      type="date"
      value="{{ currentDate }}"
      bind:cancel="onCancel"
      bind:confirm="onConfirm"
    />
  </van-popup>
</view>
<van-toast id="van-toast" />
```

#### 4. 修改 index.js
- 引入 Toast
```javascript
import Toast from '../../miniprogram_npm/vant-weapp/toast/toast';
```

- data 添加 属性
```javascript
data: {
 year: 1990,
  month: 1,
  day: 1,
  list: [],
  show: false,
  currentDate: Date.now()
}
```

- 添加 监听方法
```javascript
/**
 * 监听日期选择
 */
onChangeDate: function() {
  this.setData({
    show: true
  });
},

/**
 * 监听取消
 */
onCancel: function() {
  this.setData({
    show: false
  });
},

/**
 * 监听确定
 */
onConfirm: function(event) {
  const date = new Date(event.detail);
  const year = date.getFullYear();
  const month = date.getMonth() + 1;
  const day = date.getDate();
  this.setData({
    year,
    month,
    day,
    show: false
  });
  this.doGetList();
}
```

- 最后修改 doGetList ，添加loading
```javascript
/**
 * 执行数据获取
 */
doGetList: function() {
  const {
    month,
    day
  } = this.data;
  Toast.loading({
    mask: true,
    message: '加载中...'
  });
  wx.cloud.callFunction({
      name: 'todayInHistory',
      data: {
        month,
        day
      }
    }).then(res => {
      let list = res.result.reverse();
      this.setData({
        list
      });

      Toast.clear();
    })
    .catch(console.error)
}
```

#### 5. 效果如下

![列表](https://puui.qpic.cn/vupload/0/20190729_1564368127422_gswtb8x9x0m.webp/0)

![切换日期](https://puui.qpic.cn/vupload/0/20190729_1564368161911_rnirpu05wv.webp/0)

### 补充
- 由于聚合数据平台API非会员调用次数有限（100次/天），明显是不太够用的。因此，我们可以考虑在请求到数据时，将数据存在云数据库中，其实也就实现了一个类似爬虫的功能啦。流程如下：：梵

![](https://puui.qpic.cn/vupload/0/20190729_1564368258560_x2lf3wfaf7s.webp/0)

代码实现：

- 修改 `cloudfunctions/todayInHistory/index.js`
```javascript
// ... 省略其他无需改动的代码
exports.main = async(event, context) => {
  const {
    month,
    day
  } = event

  const ret = await db.collection('todayInHistory').where({
    date: `${month}/${day}`
  }).get()

  if (ret.data.length > 0) {
    return ret.data[0].result
  }

  const resp = await axios.get(baseUrl, {
    params: {
      key,
      date: `${month}/${day}`
    }
  }).then(res => {
    return res.data
  })
  
  await db.collection('todayInHistory').add({
    data: {
      date: `${month}/${day}`,
      result: resp.result
    }
  })

  return resp.result
}
···
 ```

### 结语
目前只开发了两个小功能 `历史上的今天` 和 `周公解梦` ，后续会继续开发新的功能，希望可以做成一个小工具集合，这也是 `口袋工具` 这个名称的由来。

感谢各位读者的阅读，由于本人水平有限，文章中如有错误或不妥之处，请不吝赐教！


如果你喜欢这篇文章或是这个项目，不妨进去点个Star支持下 today。


# 源码链接
[https://github.com/GoKu-gaga/today](https://github.com/GoKu-gaga/today)
