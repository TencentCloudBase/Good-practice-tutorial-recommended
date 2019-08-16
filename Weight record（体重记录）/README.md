前一段看到朋友圈里总是有人用txt记录体重，就特别想写一个记录体重的小程序， 现在小程序的云开发有云函数、数据库，真的挺好用，很适合个人开发者，服务器域名什么都不用管，云开发让你完全不用操心这些东西。

**先看看页面效果图吧：**

![](https://puui.qpic.cn/vupload/0/20190801_1564627993758_gx1ucl1mgcf./0)
![](https://puui.qpic.cn/vupload/0/20190801_1564628011137_e5zkj4g46a./0)
![](https://puui.qpic.cn/vupload/0/20190801_1564628020742_j4sifexrbrm./0)
![](https://puui.qpic.cn/vupload/0/20190801_1564628029557_tdee5267t7o./0)
![](https://puui.qpic.cn/vupload/0/20190801_1564628038618_rhamcbies8./0)
![](https://puui.qpic.cn/vupload/0/20190801_1564628047697_rw9ildx018j./0)
![](https://puui.qpic.cn/vupload/0/20190801_1564628080320_nxcdbw1d1y./0)

记录的几个点：

1.全局变量 globalData

2.npm 的使用

3.云函数

4.数据库操作

5.async 的使用

6.分享的配置

7.antV使用

8.tabBar地址跳转

9.切换页面刷新

### 1.全局变量 globalData

首次进入后，要存储openId给其他页面使用，使用globalData共享。

```javascript
<!--app.js 设置 globalData.openid --> 
App({
  onLaunch: function () {

    this.globalData = {}

    wx.cloud.init({})

    wx.cloud.callFunction({
      name: 'login',
      data: {},
      success: res => {
        this.globalData.openid = res.result.openid
        wx.switchTab({
          url: '/pages/add/add',
          fail: function(e) {}
        })
      }, 
      fail: err => { 
     
      }
    })

  }
})

<!--其他页面引用-->
const app = getApp()  // 获得实例
app.globalData.openid // 直接引用即可
```

### 2.npm 的使用

1.进入小程序源码` miniprogram` 目录，创建 `package.json` 文件（使用 `npm init` 一路回车）

2.`npm i --save` 我们要安装的 `npm` 包

3.设置微信开发者工具 构建 `npm`

4.`package.json` 增加 `"miniprogram": "dist"` 打包目录字段，如果不设置的话上传和预览不成功，提示文件包过大。

```javascript
cd miniprogram
npm init 
npm i @antv/f2-canvas --save   // 我用到了f2，可以换成其他包
```

**设置微信开发者工具**

![](https://puui.qpic.cn/vupload/0/20190801_1564628449211_mosinmptvtm./0)

构建 `npm`

![](https://puui.qpic.cn/vupload/0/20190801_1564628496293_9yqvham5g05./0)

最后，务必添加 `miniprogram` 字段

```javascript
{
  "name": "21Day",
  "version": "1.1.0",
  "miniprogram": "dist",
  "description": "一个21天体重记录的app",
  "license": "MIT",
  "dependencies": {
    "@antv/f2-canvas": "~1.0.5",
    "@antv/wx-f2": "~1.1.4"
  },
  "devDependencies": {}
}
```

### 3.云函数

官方解释 `云函数即在云端（服务器端）运行的函数` ，服务端是 `node.js` ，都是 `JavaScript` 。官方有数据库的操作，但是**更新的操作强制要求使用云函数**, 另外，如果云函数中使用了 `npm` 包，记得在所在云函数文件夹右键**上传并部署**，不然运行失败。

![](https://puui.qpic.cn/vupload/0/20190801_1564628676367_kakijcuofj./0)

上一个例子，更新体重的云函数

```javascript
// 云函数
const cloud = require('wx-server-sdk')
const moment = require('moment')

cloud.init(
  { traceUser: true }
)

const db = cloud.database()
const wxContext = cloud.getWXContext()

exports.main = async (event, context) => {
  // event 入参参数
  delete event.userInfo
  try {
    return await db.collection('list').where({
      _openid:wxContext.OPENID,
      date:moment().format('YYYY-MM-DD')
    })
    .update({
      data: {
      	...event
      },
    })
  } catch(e) {
    console.error(e)
  }
}
```

小程序端调用

```javascript
wx.cloud.callFunction({
     name: 'add',
     data: {
      ...Param
     },
     success: res => {
        wx.showToast({
          title: '新增记录成功',
        })
     },
     fail: err => { 
        wx.showToast({
          icon: 'none',
          title: '新增记录失败'
        })
     }
   })
```

### 4.数据库操作

其实是接入的 `MongoDB` ，封装了一部分 `api` 出来，详细的就看[官方文档](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/reference-client-api/database/)吧，有区分服务端和小程序段。

```javascript
const db = wx.cloud.database()

// 查询数据
db.collection('list').where({
    _openid: app.globalData.openid,
    date: moment().subtract(1, 'days').format('YYYY-MM-DD'),
}).get({
    success: function (res) {
        // do someThing
    }
})
```

### 5.async 的使用

![](https://puui.qpic.cn/vupload/0/20190801_1564628955577_1t7h763knxz./0)

官方文档提示不支持 `async`，需要引入 `regeneratorRuntime` 这个包，先 `npm i regenerator` 。
然后把 `node_modules` 文件夹下的 `regenerator-runtime` 的 `runtime-module.js` 和 `runtime.js` 两个文件拷贝到lib目录下，在页面上引入即可。

```javascript
<!--事例-->
const regeneratorRuntime = require('../../lib/runtime.js')
onLoad: async function (options) {

    // 获取当天数据
    await this.step1()

    // 时间类型设置
    let nowHour = moment().hour(),timeType
    nowHour > 12 ? timeType = 'evening' : timeType = 'morning'
    this.setData({timeType})
  }
```

### 6.分享的配置

分享很简单，有区分右上角的直接分享和点击按钮分享

```javascript
onShareAppMessage: function (res) {
        
      // 右上角分享
      let ShareOption = {
        title: '21天体重减肥记录',
        path: '/pages/index/index',
      } 
      
      // 按钮分享
      if(res.from == "button"){
        ShareOption = {
            title: '来呀 看看我的减肥记录呀',
            path: '/pages/detail/detail?item=' + app.globalData.openid,
          } 
      }
      
      return ShareOption
  }
```

分享后，他人点击页面，跳转到对应 `pages` 地址，从 `onLoad` 的 `options `中拿入参请求数即可

```javascript
onLoad: function (options) {
    const db = wx.cloud.database()
    let This = this
    let resault = {}
    db.collection('list').where({
      _openid: options.item
    }).get({
      success: function (res) {
        resault = res.data
        This.setData({
          resault:resault
        })

      }
    })
  },
```

### 7.antV使用

上边第二小节有提到 `antV` 的安装，就不再赘述，直接说一下再页面中引用。

说下使用，需要设置一个全局变量储存图表的实例，然后在钩子函数内容使用 `changeData` 方法修改数据。

 `index.json` 中引入包名

```javascript
{
  "usingComponents": {
  	"ff-canvas": "@antv/f2-canvas"
  }
}
```

```javascript
// 引入F2
import F2 from '@antv/wx-f2';

// 设置实例全局变量（务必）
let chart = null;
function initChart(canvas, width, height, F2) { // 使用 F2 绘制图表
  let data = [
    // { timestamp: '1951 年', step: 38 },
  ];

  chart = new F2.Chart({
    el: canvas,
    width,
    height
  });

  chart.source(data, {
    step: {
      tickCount: 5
    },
    timestamp: {
      tickCount: 8
    },

  });


  chart.axis('timestamp', {
    label(text, index, total) {
      const textCfg = {};
      if (index === 0) {
        textCfg.textAlign = 'left';
      }
      if (index === total - 1) {
        textCfg.textAlign = 'right';
      }
      return textCfg;
    }
  });

  chart.axis('step', {
    label(text) {
      return {
        text: text / 1000 + 'k步'
      };
    }
  });

  chart.tooltip({
    showItemMarker: false,
    onShow(ev) {
      const { items } = ev;
      items[0].name = null;
      items[0].name = items[0].title;
      items[0].value = items[0].value + '步';
    }
  });
  chart.area().position('timestamp*step').shape('smooth').color('l(0) 0:#F2C587 0.5:#ED7973 1:#8659AF');
  chart.line().position('timestamp*step').shape('smooth').color('l(0) 0:#F2C587 0.5:#ED7973 1:#8659AF');
  chart.render();
  return chart;
}

// 生命周期函数
onLoad(){
    // 使用changeData赋值
    chart.changeData(stepInfoList)
}
```

### 8.tabBar地址跳转

如果要跳转的地址不在 `app.json` 的 `tabBar` 内可以使用 `wx.navigateTo` ，如果在死活跳不过去，要使用` wx.switchTab` 方法跳转。

```javascript
wx.switchTab({
  url: '/pages/add/add',
  fail: function(e) {}
})

wx.navigateTo({
  url: '../deployFunctions/deployFunctions',
})
```

### 9.切换页面刷新

切换几个tabBar的时候，需要刷新数据。 在 `onShow` 方法中再调用一下 `onLoad` 方法就可以了。

```javascript
onShow: function () {
    this.onLoad()
}
```

# 原文链接
[https://juejin.im/post/5d359484f265da1bb5653716](https://juejin.im/post/5d359484f265da1bb5653716)
