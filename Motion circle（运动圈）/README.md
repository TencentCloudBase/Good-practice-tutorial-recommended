# 乒乓圈小程序

和朋友合伙写了一个小程序，写了一个以共享乒乓信息和交流的平台———乒乓圈。我们使用了微信的云开发来完成数据和后台的作用。免去了租赁服务器。

我主要负责的是数据库的设计和云函数实现数据获取和触发器的功能和简单的两个页面。

# 正文
## 功能展示

![](https://user-gold-cdn.xitu.io/2019/6/28/16b9e1dae12641b1?w=409&h=718&f=gif&s=3051762)

## 页面分析
* ### 引导页

![](https://puui.qpic.cn/vupload/0/1567584710029_6709u8ca8y8.png/0)

当用户未授权则会弹出，点击下方指纹图片，则会弹出授权框，授权后，如果未注册则会注册完毕后进入首页 

![](/img/remote/1460000019646155?w=1092&h=2260)

* ### tabbar中的三个模块
 三个模块分别为 首页、圈友、个人 模块。
 
![](https://user-gold-cdn.xitu.io/2019/6/27/16b989eddcbef407?w=3600&h=2280&f=png&s=1770070)

* ### 首页的三个功能

1. 同城圈
    -- 同城圈可以看到共享的球馆，点击加号就可共享球馆
2. 签到
    -- 签到规则可以增加积分
3. 排行榜
    -- 可以看积分排行榜
    
    ![](https://puui.qpic.cn/vupload/0/1567584790734_pde35xfosll.png/0)
    
 **页面流程大致分为** 
```
- 引导页
    - 首页
        - 同城圈
        - 打卡
        - 榜单
    - 圈友页
        - 同城圈友
        - 留言列表
    - 个人页
        - 个人资料
```


## 数据库


从以上的功能出发我的数据库设计思路如此
对象有以下几个：
 * 个人
 * 球馆
 * 对话

对象就只有大致三个，但是为了数据操作的简便性我将个人的信息分成两个对象表，将留言中的对话又单独放出一张表，所以最后的表有为一下几个：
* 个人基础信息
* 个人详细信息(乒乓球相关)
* 球馆
* 对话
* 留言信息


**对象属性的类型选择** 

首先，小程序提供的数据库是基于mangoDB的面向对象数据库，区别于一般的关系数据库如：mysql等。二者之间的区别和我的理解会写在总结中。

信息是反映对象状态的一种

我认为数据库存储的属性大致可分为三种
- **基础信息数据**
    -- 是需要存储的基础数据，无需任何处理可直接输出的数据，例如：姓名等
- **功能性数据**
    -- 是可能需要一定处理转变，表示的数据，例如：会员等级(vip,svip)
- **标记型数据**
    -- 是一种特殊的标记，例如：唯一标识符(openId)等

但是很多信息都兼顾以上的几种，例如：学号(即是标记型，又是基础信息)

确认完对象基础属性后就要考虑对象之间的关系，例如人和对话，留言和对话信息。
关系种类有 一对一(1-1)，一对多(1-n)，多对多(m-n)。

在 **关系数据库** 中，**一对一**的关系只要在一条记录中添加一个属性即可，例如：个人信息和个人详情，在个人详情中添加个人的唯一表示符字段；
**一对多**的关系中需要在多数的记录中添加一个属性，或者单独建立一张表来存储关系，
例如：个人和物品，第一种在物品对象中添加一个所有者对象，或者建立一个所属关系表；
**多对多**的关系则只能通过单独一张关系表来完成，例如：学生和课程，需要单独一张选课表来表示关系。

在 **面向对象数据库**中**一对多**和**多对多**的关系可以通过对象中的一个数组字段来完成，例如：学生和课程，在学生对象中添加一个所选课程字段存储课程 ID ，在课程中添加选课学生字段存储学号，就完成了多对多的关系链接。


### 对象结构如下所示

#### 个人基础信息：
```
openId:{type:String}//openId主键
name:{type:String}//名字，默认为微信名
avatarUrl:{type:String}//头像，默认微信头像
context:{type:String,default:"这个人很懒什么都没留下"}//个人简介
//以下几项应为流水数据应但对放一张表，在此为图简便放入基础信息表
intergal:{type:Number,default:0}//积分 用来排名和升级
level:{type:String,default:"新人"}//等级
sign:{type:Array,default:[]}//记录打卡签到的日期
month:{type:Number}//记录上次打卡签到的月份，用于每月清空签到表
```
#### 个人详情(乒乓球相关)
```
openId:{type:String}//openId主键
years:{type:String}//球龄
phone:{type:Array}//电话
bat:{type:String}//球拍
board:{type:String}//底板
context:{type:String}//正面胶皮
intergal:{type:Number,default:0}//反面胶皮
```
#### 球馆
```
id:{type:String}//主键，由数据库自动生成
address:{type:String}//地址
arena:{type:String}//所在区域，例如球馆名
persons:{type:Array}//球馆的活动者，需求更改数据库中的字段为circle
city:{type:String}//球馆所在的城市
img:{type:String}//球馆图片地址
latitude:{type:Number}//经度
longitude{type:Number}//纬度
table:{type:Number}//球桌数
time:{type:String}//开放时间
```
#### 对话
```
message:{type:Array,default:[]}//留言内容数组，存储留言的id
my_id:{type:String}//创建者的openId
other_id:{type:String}//接收者的openId
```
#### 留言信息
```
id:{type:String}//主键，由数据库自动生成
msg:{type:String}//留言内容
my_id:{type:String}//创建者的openId
other_id:{type:String}//接收者的openId
time:{type:Date}//时间戳
```

## 云函数读取数据库和部分前端实现

### 1. 引导页

![](https://puui.qpic.cn/vupload/0/1567584852852_7d60ai2r8bt.gif/0)

当第一次登陆进区就是如上所示，登陆进去后通过 openId 进行云函数获取数据库中个人信息，如果没有则默认进行注册流程。
默认昵称为微信昵称（可在个人页更改），头像为微信头像（暂不提供更改），余下都为默认值。
引导页 js
```
const QQMapWX = require('../../libs/qqmap-wx-jssdk.js');// 连接腾讯地图
const qqmapsdk = new QQMapWX({
  key: 'HMGBZ-U5XCX-TUX4Y-ZPUH3-7RRX5-BZBCW'
});
const app = getApp()
Page({
  data: {
    login: false
  },
  getUserInfo(e) {
    if (e.detail.userInfo && !this.data.login) {
      console.log('登录中')
      let the_first = false;
      // 掉用获取用户信息函数，用openId作为唯一标识符
      wx.cloud.callFunction({
          name: "getPersonInfo",
        })
        .then(res => {
          // 判断是否为空，空则代表第一次进入
          if (res.result.data.length == 0) {
            the_first = true
          } else {
            // 已经注册过，获取到信息放入 app.globalData 全局数据中。 
            app.globalData.personInfo = res.result.data[0];
            console.log(app.globalData.personInfo);
            wx.cloud.callFunction({
              name: "getpingpang_info",
              success: res => {
                console.log('登录成功')
                // console.log(res.result.data)
                app.globalData.ping_personInfo = res.result.data[0]
                wx.setStorage({
                  key: 'login',
                  data: true
                })
                wx.switchTab({
                  url: '../home/home',
                })
              }
            })
          }
        }).then(() => {
          // 进入注册流程，
          return new Promise((resolve, reject) => {
            if (the_first) {
              // 获取用户的信息
              wx.getUserInfo({
                lang: "zh_CN",
                success: res => {
                  app.globalData.userInfo = res.userInfo;
                  resolve();
                },
              })
            }
          })
        })
        .then(() => {
          if (the_first) {
            // 用户注册所需昵称和头像
            const data = {
              name: app.globalData.userInfo.nickName,
              avatarUrl: app.globalData.userInfo.avatarUrl,
            };
            // 显示加载
            wx.showLoading({
              title: '授权登录中',
            })
            // 用户注册函数，除了昵称和头像，全置为最低或空
            wx.cloud.callFunction({
                name: "pingpang_init",
                data: data
              }).then(res => {
                // 数据库已经注册完成
                console.log("注册完成")
              })
              .then(() => {
                // 注册完成后获取一遍用户信息
                wx.cloud.callFunction({
                  name: "getPersonInfo"
                }).then(res => {
                  app.globalData.personInfo = res.result.data[0];
                  console.log(res.result.data[0])
                  // 隐藏加载
                  app.globalData.ping_personInfo = {
                    openId: app.globalData.personInfo.openId,
                    phone: '***********',
                    years: '0年',
                    bat: '右手横拍',
                    board: '新手用具',
                    infront_rubber: '新手用具',
                    behind_rubber: '新手用具'
                  }
                  wx.hideLoading();
                  // 提示注册完成
                  // wx.showModal({
                  //   title: '注册',
                  //   content: '注册完成',
                  // })
                  wx.setStorage({
                    key: 'login',
                    data: true
                  })
                  wx.switchTab({
                    url: '../home/home',
                  })
                })
              })
          }
        })
    }
  },
  onLoad() {
    wx.getStorage({
      key: 'login',
      success: (res) => {
        if (res.data) {
          this.setData({
            login: true
          })
          app.timeout = setTimeout(() => {
            wx.showLoading({
              title: 'lodaing',
            })
          }, 3000)
          app.neterror = setTimeout(() => {
            wx.hideLoading()
            wx.showModal({
              title: '伤心提示',
              content: '网络走丢了...',
              showCancel: false
            })
          }, 20000)
        }
      }
    })
  }
})
```

以上流程可分为以下几步。
#### 1. 进行 onLoad （页面加载完成） 生命周期 判断是否有缓存
首先调用 `wx.getStorage` 查询收缓存的登陆信息，如果获取成功，跳过引导页，将当前登陆状态的标识符(默认false)改为 true 。
通过 `setTimeout` 来控制提示信息和超时检测。

#### 2. 未缓存，进入登陆注册功能
如果获取缓存中登陆状态。先获取授权信息 `getUserInfo` 判断获取到的用户信息存在且为登录。

`the_first` 判断是否是第一次进入要进行注册流程(保险作用).

调用**云函数 `getPersonhInfo`** ——获取用户信息，如果获取成功，结果集不为空，就将信息存储到全局状态 `app.globalData` 中.接着调用**云函数 `getpingpang_info`** 获取个人详细信息一样放入全局状态中，然后写入缓存信息
`wx.setStorage({key: 'login',data: true})`以便下次不用检验，最后通过 `wx.switchTab({url: '../home/home',})` 来跳转到首页。

如果上一步未获取成功，判断为第一次登入，进入注册流程 先获取用户的昵称后头像，调用**云函数 `pingpang_init`**后台进行注册，并将初始值放入全局状态中，跳转到首页。

####  **所需云函数**
#### 1. getPersonhInfo
```
// 云函数入口文件
const cloud = require('wx-server-sdk')

cloud.init()

const db = cloud.database();
const personinfo = db.collection("pingpang_personinfo")
const _ = db.command;
// 云函数入口函数
exports.main = async(event, context) => {
  let {
    openIdarr,
    openId,
    all,
    city
  } = event;
  if (all) {//获取所有人信息
    return await personinfo.get()
  } else if (city) {//获取所给城市的所有用户信息
    return await personinfo.where({
      city
    }).get()
  } else if (openIdarr) {//获取openId在所给数组中的所有用户信息
    console.log(openIdarr)
    return await personinfo.where({
      openId: _.in(openIdarr)
    }).get()
  } else {//获取所给openId或自身的用户信息
    return await personinfo.where({
      openId: openId || event.userInfo.openId
    }).get()
  }
}
```
这个云函数是获取用户信息，
首先解构用户传来的参数来判断需要的数据，openIdarr--通过openId数组获取，openId--通过openId获取，city--通过用户所在城市获取，all--获取所有用户，以上四种都没有则获取当前用户的信息。

#### 注：event.userInfo.openId 只有用户程序直接调用云函数的时候，云函数才可以获取到，当云函数调用云函数时，被调用的的云函数无法获取 userInfo 这个对象属性。
#### 2. getpingpang_info
```
// 云函数入口文件
const cloud = require('wx-server-sdk')

cloud.init()
const db = cloud.database();
// 云函数入口函数
exports.main = async(event, context) => {
  return await db.collection("pingpang_info").where({
    openId: event.openId || event.userInfo.openId
  }).get()
}
```
这个云函数只是简单的通过两种方式(给与openId或默认自身)来获取获取用户详细信息
#### 3. pingpang_init
```
// 云函数入口文件
const cloud = require('wx-server-sdk')

cloud.init()

// 云函数入口函数
exports.main = async (event, context) => {
  const fun1 = await cloud.callFunction({
    name: "addPersonInfo",
    data: {
      openId: event.userInfo.openId,
      name: event.name || "未获取到名字",
      avatarUrl: event.avatarUrl,
      city: event.city,//所在城市
      level: "新人",
      intergal: 0,
      context: "",
      activitiew: [],
      circle: []
    }
  })
  const fun2 = await cloud.callFunction({
    name: "addpingpang_info",
    data: {
      openId: event.userInfo.openId,
      phone: '***********',
      years: '0年',
      bat: '右手横拍',
      board: '新手用具',
      infront_rubber: '新手用具',
      behind_rubber: '新手用具'
    }
  })
  return { fun1, fun2 }
}
```
这个函数是初始化函数，功能是向数据库添加新用户的初始数据。

### 2.签到功能
签到功能的页面并非我写的，所以我只能提供思路和云函数。
签到的存储是在个人信息的一个字段sign中，以数组的形式存储，当点击签到时，先判断此次签到的月份与上次签到的月份(person的month字段)是否相同，不同则将sign数据置为空并且将month字段更新为当前月份，接着存储签到的日期的的day，

![](https://user-gold-cdn.xitu.io/2019/6/29/16ba2805c62bcab1?w=409&h=718&f=gif&s=241607)
#### 云函数
```
// 云函数入口文件
const cloud = require('wx-server-sdk')

cloud.init()
const db = cloud.database();
const personInfo = db.collection("pingpang_personinfo")

// 云函数入口函数
exports.main = async(event, context) => {
  let data = personInfo.where({
    openId: event.openId || event.userInfo.openId
  })
  let info = await data.get()//先获取
  let sign = info.data[0].sign || [] //放入新数组
  //判断是否到了新的月份
  if (info.data[0].month != new Date().getMonth()) {
    sign = [];
    var month = new Date().getMonth()
  }
  //替换数组
  return await cloud.callFunction({
    name: "setPersonInfo",
    data: {
      personInfo: {
      //event.date是为了方便写管理调试用的一次性放日期数组
      sign: sign.concat(event.date || [new Date().getDate()]),
        month:month
      }
    }
  })
}
```

### 3.排行榜单

榜单十分简单，有多种做法：


#### 1. 第一种是将同城所有人查询出来按照积分排序，并区前一定数量的用户来输出排行榜
* 优点：无需其他的资源来存储，不占用空间，修改排行榜的时候无需多余的处理
* 缺点：无法承载大量的用户，当用户增多到一定数量后，单次查询时间会变得很慢，查询并发数量会有问题，因为查询的都是同一张表

所以这种方法只适用于用户量较少的情况下。


#### 2. 第二种是以城市排行榜为对象，创建一张表，表中存储的对象的属性大致如下所示
* 优点：减少了查询后大量数据的处理，单人查询一次只需要处理相应数量的数据，不需要遍历一遍所有数据
* 缺点：需要额外的存储空间，如果存储的是用户 openId 那查询速度依然较慢，如果存储的是用户对象，那么查询速度只需要查询单张表的时间，修改排行榜的时候又需要单独处理数组字段，较为麻烦。

这种方式使用用户量较大但是分散的情况，可以普遍使用。

```
city:{
    type:String
},
//存储排行榜，存储一定数量的用户openId，或者是用户对象
list:{
    type:Array,
    default:[]
}，
minIntergal:{
    type:Number,
    default:0
}
```


#### 3. 第三种则是以每个城市为一张表，存储积分达到排行榜对象
* 优点：有点是解决了处理数量的问题，并发问题也解决了，单个城市的人处理一张表，并发数量会下降。
* 缺点：大量占用空间


这种做法适用于用户数量极大的时候。


####  **总体方案**：从以上方法来说最好的方法是，在大量的用户的城市，做单独一张表来存储，剩余小型城市则存储在剩余的总表中，唯一的缺点就是判断处理的麻烦，当一个城市用户变多时，需要在数据库中添加一张新表，这需要手动来解决，变更后台的处理判断，可以使用**策略模式**来解决。

![](https://user-gold-cdn.xitu.io/2019/6/29/16ba330882b8c05d?w=409&h=718&f=gif&s=152887)

### 4. 留言功能
留言功能,是这个小程序的主要功能之一，目的是为了向兴趣相同的乒乓爱好者有一个初始的交流平台。
创建留言需要在圈友(同城的)中找到相应的用户，然后点击头像，弹出详情，接着点击留言按钮，会跳转到留言对话页。


留言有两种情况，一种是之前有过留言，存在留言对象，另一种则是第一次对话，之前不存在留言对象。
第一种，只需要查询到存在就可向里面添加留言信息。第二种则需要先创建在进行添加。

第一种没有任何问题，直接对对话对象的留言数组中进行添加，第二种则需要创建一个对话对象。

**具体流程**：首先在留言页查询到所有对话对象，这是走第一种情况，可以跳转到直接添加留言，第二种则是在圈友页中对象的详情页点击留言按钮，这会先查询对话兑现，不存在则会跳转到空白对话页，否则跳转到之前的留言对象。
**这种方法不是很好**

**缺点如下**：
* 如果点击留言但是不留言，会创建一个空白的对话对象，用户存在误触按钮的情况，这会存在很多空白留言，这是一大缺陷。
* 因为这是前端来控制的所以存在一定的延迟，要进行多次异步的操作，造成延迟。

**推荐方案** ：在点击留言时查询之前是否存在对话兑现，存在即读取，不存在就跳转到空白页，如果发送了留言，则创建对象。这样可已解决以上的缺点。但是还是存在一个问题就是未使用 **socket** 无法达成实时通信。
![](https://user-gold-cdn.xitu.io/2019/6/30/16ba6de054ba50c9?w=540&h=1124&f=gif&s=2377109)

#### 云函数
1. 获取对话对象
```
// 云函数入口文件
const cloud = require('wx-server-sdk')

cloud.init()
const db = cloud.database();
const dialoguedb = db.collection("pingpang_dialogue")
const _ = db.command;
// 云函数入口函数
exports.main = async (event, context) => {
  let {
    my_id,
    other_id
  } = event;
  if (!my_id) my_id = event.userInfo.openId
  // let data1 = await dialoguedb.where({
  //   my_id,
  //   other_id
  // }).get()
  if (!other_id) return await dialoguedb.where({
    my_id
  }).get()
  return await dialoguedb.where({
    my_id,
    other_id
  }).get()

}

  // console.log(data1)
  // console.log('\n',data2)
  // return data2;
```

云函数的大致功能为：
首先，结构传递的参数**my_id(当前用户的id)** 和 **other_id(留言对象的id)**。接着判断 **my_id** 是否存在，不存在就给当前用户的 **openId** ，最后判断 **ohter_id** 如果不存在，则查询前用户所有的对话。


2. 添加留言内容
```
// 云函数入口文件
const cloud = require('wx-server-sdk')

cloud.init();
const db = cloud.database();
const messagedb = db.collection("pingpang_message");
const dialoguedb = db.collection("pingpang_dialogue");
const _ = db.command;

// 云函数入口函数
exports.main = async(event, context) => {
  let {
    message,
    my_id,
    other_id,
    msg
  } = event;
  console.log(other_id)
  if (!my_id) my_id = event.userInfo.openId;
  // console.log(other_id)
  let message_id = await messagedb.add({
    data: message || {
      my_id,
      other_id,
      msg,
      time: new Date()
    }
  })
  // console.log(message_id)
  const res = await cloud.callFunction({
    name:"getpingpang_dialogue",
    data:{
      my_id:my_id,
      other_id:other_id
    }
  })
  let myTo = res.result.data[0];
  await dialoguedb.doc(myTo._id).update({
    data: {
      message: myTo.message.concat([message_id._id])
    }
  })

  const res1 = await cloud.callFunction({
    name: "getpingpang_dialogue",
    data: {
      my_id: other_id,
      other_id: my_id
    }
  })
  let otherTo = res1.result.data[0];
  await dialoguedb.doc(otherTo._id).update({
    data: {
      message: otherTo.message.concat([message_id._id])
    }
  })

}
```
上述的云函数功能大致为：
首先，结构参数，**message(留言对象，包含之后的几个参数)**，**my_id**，**other_id**和**msg(留言内容)**。接着判断 **my_id**是否存在，不存在就用当前用户的 **openId** 。然后是向留言表添加一条新数据 `messagedb.add` 最后获取对话对象并向对话中的留言数组中添加留言内容。

### 5. 个人模块
个人模块没有什么复杂的逻辑，就是数据渲染页面，不过页面结构是我写的，可以聊一聊页面了。

![](https://user-gold-cdn.xitu.io/2019/6/30/16ba763b1f6017aa?w=409&h=718&f=gif&s=957010)

#### 个人页
个人页面中没有什么比较花里胡哨的样式操作，只有简单基础的 css 和 html ，所以就在此简单结构一下。

大概要讲的就是点击切换成输入框的所需要讲的，还有下面的选择栏变变成**组件**

页面(部分)
```
//个人简介
<view class="infocard" bindtap='typeInfo'>
  <input type="text" wx:if="{{changecontext}}" placeholder='' bindblur='setcontext' focus='true' value="{{personInfo.context}}" maxlength='18'></input>
        <view class="context" wx:else bindtap='changecontext'>个性签名:{{personInfo.context}}</view>
  </view>
  
  
  //个人资料框
  <view class="project collections" bindtap="ToPage" data-name="pingpang_info">
      <image class="image" src="https://636f-coldday-67x7r-1259123272.tcb.qcloud.la/person.svg?sign=73135fcd2247e0a00ca78c131fa0d7d6&t=1559030458" />
      <view class="title">个人资料</view>
      <text class='cuIcon-right righticon text-grey'></text>
    </view>
```
js(部分)
```
data:{
    changecontext: false
}
changecontext() {
    this.setData({
      changecontext: true
    })
  },
 setcontext(event) {
    this.setData({
      changecontext: false
    })
    if (event.detail.value != "") {
      this.setData({
        "personInfo.context": event.detail.value,
        context: event.detail.value
      })
    }
else {
      this.setData({
        "personInfo.context": "这家伙打完球后不留任何足迹",
        context: "这家伙打完球后不留任何足迹"
      })
    }
  },
ToPage(event) {
    wx.navigateTo({
      url: `../${event.currentTarget.dataset.name}/${event.currentTarget.dataset.name}`,
      fail: () => {
        wx.showModal({
          title: '(ಥ_ಥ)',
          content: '敬请期待！',
          showCancel: false
        })
      }
    })
  },
```

文本和输入框的切换，是通过 `wx:if` 来控制显示，让两个大小近似的块占用相同的地方，当点击文本时，数据源(data)中的 changecontext 变量变成 ture 页面重新渲染，将输入框显示 value 为数据源中的个人简介，文本则隐藏；当输入框失去焦点时，将输入框中的value值写入数据源中，然后changecontext变为false，页面重新渲染，就改完了个人简介。


修改后提交数据的方案有三种

1. 在修改完后直接提交
2. 在页面隐藏或关闭后提交
3. 在页面隐藏或关闭后，判断是否修改过内容，是则提交

第一种和第三种都可以普遍使用。推荐第一种方式，因为大多数用户不会过于频繁的去修改这些东西，但是页面基本都是每次登陆都会访问多次的。频率和并发都是第一种好。



#### 个人详情
个人详情就是普通的页面，没有复杂的云函数，只有一个获取，一个提交修改，两个函数都不复杂。

详情页中球拍和球龄是使用了小程序自带组件 **picker** 其余则是使用了自定义组件 **info-section**

页面
```
<view class="container">
 <view class="cu-form-group">
		<view class="title">球龄</view>
		<picker bindchange="PickerAgeChange" value="{{indexAge}}" range="{{pickerAge}}">
			<view id='picker' class='picker'>
				{{indexAge?pickerAge[indexAge]:personInfo.years}}<text class='cuIcon-title' style='opacity:0'></text>
			</view>
		</picker>
	</view>
  <section title="电话" info="{{personInfo.phone}}" infoname="phone" bind:changend="getinfo" type='number'/>
  <!-- <section title="球龄" info="{{personInfo.years}}" infoname="years" bind:changend="getinfo" type='number'/> -->
  <!-- <section title="持拍" info="{{personInfo.bat}}" infoname="bat" bind:changend="getinfo" /> -->
  <section title="使用底板" info="{{personInfo.board}}" infoname="board" bind:changend="getinfo" />
  <section title="正手胶皮" info="{{personInfo.infront_rubber}}" infoname="infront_rubber" bind:changend="getinfo" />
  <section title="反手胶皮" info="{{personInfo.behind_rubber}}" infoname="behind_rubber" bind:changend="getinfo" isbottom="true" />
  <view class="cu-form-group">
		<view class="title">持拍</view>
		<picker bindchange="PickerChange" value="{{index}}" range="{{picker}}">
			<view id='picker' class='picker'>
				{{index?picker[index]:personInfo.bat || '点击选择'}}
			</view>
		</picker>
	</view>
  <button  class='button' bindtap="submit">点击提交</button>
</view>  
```

js
```
const app = getApp();
Page({

  /**
   * 页面的初始数据
   */
  data: {
    personInfo: {},
    picker: ['右手横拍', '右手直拍', '左手横拍', '左手直拍']
  },
  PickerChange(e) {
    let personInfo = this.data.personInfo;
    this.setData({
      index: e.detail.value
    })
    personInfo.bat = this.data.picker[this.data.index];
    this.setData({personInfo})
  },
  PickerAgeChange(e) {
    let personInfo = this.data.personInfo;
    this.setData({
      indexAge: e.detail.value
    })
    personInfo.years = this.data.pickerAge[this.data.indexAge];
    this.setData({personInfo})
  },
  getinfo() {
    this.setData({
      personInfo: app.globalData.ping_personInfo,
    })
  },
  submit() {
    let personInfo = this.data.personInfo;
    if(personInfo.phone.length != 11){
      wx.showModal({
        title: '提示',
        content: '无效电话号码',
        showCancel:false
      })
      personInfo.phone = '';
      this.setData({
        personInfo
      })
      return
    }
      const that = this;
    console.log("开始提交")
    wx.showLoading({
      title: '提交中',
    })
    let info = this.data.personInfo
    wx.cloud.callFunction({
      name: "setpingpang_info",
      data: info
    }).then(res => {
      wx.hideLoading();
      wx.showToast({
        title: "提交成功",
        duration: 1000,
      })
      console.log(res, "修改成功")
      wx.navigateBack({

      })
    })
  },
  /**
   * 生命周期函数--监听页面加载
   */
  onLoad: function (options) {
    this.setData({
      personInfo: app.globalData.ping_personInfo
    })
    let pickerAge = []
    for (let i = 0; i < 51; i++) {
      pickerAge.push(i + '年')
    }
    this.setData({ pickerAge })
  }
})
```

##### 自定义组件 info-section
页面
```
<view class="cu-form-group">
  <view class="title">{{title}}</view>
  <input type='{{type}}' wx:if="{{changeinfo}}" bindblur='changend' value='{{info}}' placeholder="请输入信息" focus='true'></input>
  <view class='info' wx:else>
    <input value='{{info?info:"新手用具"}}' disabled='true'></input>
  </view>
  <view class='icon-con' bindtap='changeinfo'>
    <image src="https://636f-coldday-67x7r-1259123272.tcb.qcloud.la/change-1.png?sign=c8936111328dcb2ee416201369716380&t=1559030699" class='icon'></image>
  </view>
</view>
```
<font size= 3>js</font>
```
// components/info-section/section.js
Component({
  /**
   * 组件的属性列表
   */
  properties: {
    title: {
      type: String,
      value: "属性名"
    },
    info: {
      type: String,
      value: "属性值"
    },
    infoname: {
      type: String,
      value: ""
    },
    isbottom: {
      type: Boolean,
      value: false
    },
    type:{
      type:String,
      value:'text'
    }
  },

  /**
   * 组件的初始数据
   */
  data: {
    changeinfo: false,
  },

  /**
   * 组件的方法列表
   */
  methods: {
    changeinfo() {
      this.setData({
        changeinfo: true
      })
      this.triggerEvent("changeinfo");
    },
    changend(event) {
      this.setData({
        changeinfo: false
      })
      getApp().globalData.ping_personInfo[this.properties.infoname] = event.detail.value
      //抛出事件以便于父组件响应
      this.triggerEvent("changend")
    }
  }
})
```

父子组件的通讯一定要注意在子组件中抛出事件，触发父组件的事件来达成。


# 总结
## 开发总结

**良好沟通的重要性**

在和朋友一起开发小程序的过程中注意到了以下的问题， **沟通** 是最重要的，在我们开发的过程中，因为没有良好的沟通，导致，前后端的功能开发对接不完美。部分功能分配不好，有些功能可以同过前端或后端单独解决，缺因为没有沟通完善，导致双方都做了或者双方都没做的情况发生，虽然有每个人都有自己的事，大多数时间都是单独开发的原因在。但是这些问题应当在代码开发流程就应当做的，这是我了解的一个问题。

## 个人思考
### 程序的结构
程序的结构大致分为前端页面、后端服务器和数据库三个组成部分。在小程序这种 MVVM 结构中前端占有了很重要的一部分。

![](https://puui.qpic.cn/vupload/0/1567584980484_w2mf890bcel.png/0)

前后端和数据库的比例大致为 n:1:1 的关系，所以当用户量大的程序，多数操作应当放在前端中处理，这是现在 mvvm 称为主流的原因，后台主要统筹管理总体数据或者对重要的流水数据处理，并且需要提供大量的 api 供前端获取数据，
这样能大量缓解数据库的压力。

### 关系型数据库和面向对象数据库的对比
关系型数据库是传统的数据库，现在使用的主要是mysql 和 microsoft sql server。面向对象数据库是新兴数据库，现在使用的是 mangoDB等。

关系型数据库中，最独特的也是最重要的是 **规划范式** 在关系型数据库中范式等级越高，数据的整体性越低，那么冗余度会逐渐下降。
一个学生用户可能会被分成多张表来存储相关信息。而关系型数据库中主要的也是两张表之间的关系(联系)，这个关系通常也必须使用一张表来存储。

在面向对象数据库中，与传统关系型数据库最大的区别数，它是以一个对象来存储的，对象的属性则是自己定义的，它的属性可以存储一个对象(函数，数组)。这就极大的增加了可操作性，我们可以把关系作为对象的一个属性来存储，例如：学生和课程的关系，二者之间是多对多的关系，本来在关系型数据中需要建立一张选课表来存储，现在只需要在课程对象中添加一个选课字段存储选课学生的 id 数组，而在学生对象中添加一个所选课程字段，二者之间的关系就链接起来了。面向对象数据库中，对象的属性通常可以聚集在一起，一个对象类就是一张表，这样会造成每张表中拥有大量的数据每次操作会造成的并发问题，所以每个对象类最好将属性分割，让数据访问更加平均，减少每个对象表的同时访问次数。


## 感想
在和他人一起，写小程序的时候出现种种问题，甚至有时候效率还没有一个人单独写的高，但是我发现和他人一起写会更有动力，每个人的想法在碰撞，能快速的提高自己的编程水平和与他人的沟通能力。

## 源码链接
[https://github.com/colddayer/TTcircle](https://github.com/colddayer/TTcircle)
