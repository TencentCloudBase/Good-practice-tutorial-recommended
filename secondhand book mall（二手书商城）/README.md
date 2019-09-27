“拱手让书，智慧相传。本文将带大家使用云开发快速开发完整的校园二手书商城“


## 导语

> 很多大学有个普遍现象，毕业或者搬校区的时候，成堆成堆的书都被随便处理掉，作为过来人，每每想到都十分痛心可惜，而导致这种情况发生的原因，我认为主要还是归结学校原因，一方面没有提供靠谱便利的平台，另一方面，宣传不到位，基于此开发了这款小程序。下面挑了些开发过程中遇到的典型来讲解实现过程，感兴趣可以一览......


## 一：登录注册页

目前小程序有了详细的登录规范，参考官方示例，本程序的登录入口做了以下处理：

1. 在需要涉及用户信息的部分，进行Modal提示进入，比如：游客发布、购买等
2. 个人中心，未登录默认显示”点击登录“按钮

好了，先来看看登录页面效果图吧：

![登录示例图][1]


* 手机号获取（相关代码）：

``` html
<button class="phone" open-type="getPhoneNumber" bindgetphonenumber="getPhoneNumber">
                  <block wx:if="{{phone==''}}"> 请点击获取您的手机号</block>
                  <block wx:if="{{phone!==''}}"> {{phone}}</block>
                  <image wx:if="{{phone==''}}" class="right" src="/images/right.png" />
</button>
```
``` javascript
 //获取用户手机号
      getPhoneNumber: function(e) {
            let that = this;
            //判断用户是否授权确认
            if (!e.detail.errMsg || e.detail.errMsg != "getPhoneNumber:ok") {
                  wx.showToast({
                        title: '获取手机号失败',
                        icon: 'none'
                  })
                  return;
            }
            wx.showLoading({
                  title: '获取手机号中...',
            })
            wx.login({
                  success(re) {
                        wx.cloud.callFunction({
                              name: 'regist', // 对应云函数名
                              data: {
                                    $url: "phone", //云函数路由参数
                                    encryptedData: e.detail.encryptedData,
                                    iv: e.detail.iv,
                                    code: re.code
                              },
                              success: res => {
                                    wx.hideLoading();
                                    //获取成功，设置手机号码
                                    that.setData({
                                          phone: res.result.data.phoneNumber
                                    })
                              },
                        })
                  },
            })
      },
```

1. 此处仅展示前端部分核心代码，手机号获取涉及到解密过程，需要配合云函数实现，具体的请参考完整demo注册页代码
2. 目前该接口针对非个人开发者，且完成了认证的小程序开放（不包含海外主体）。

* 常用联系方式的校检：

``` javascript
if (!(/^\w+((.\w+)|(-\w+))@[A-Za-z0-9]+((.|-)[A-Za-z0-9]+).[A-Za-z0-9]+$/.test(email))) {
                  wx.showToast({
                        title: '请输入常用邮箱',
                        icon: 'none'
                  });
                  return false;
            }
```
同理相关正则：
``` javascript
//手机号
/^[1][3,4,5,6,7,8,9][0-9]{9}$/
//QQ号
/^\s*[.0-9]{5,11}\s*$/
//微信号
/^[a-zA-Z]([-_a-zA-Z0-9]{5,19})+$/
```
> 目前常用手机号，似乎就差10和12字段的没有了。

## 二：发布信息页


![发布页效果图][2]


* 步骤条实现

发布页有几个小地方值得留意：

1. 顶部的步骤条，随操作流程一直在变。
2. 步骤改变时，有个横向切换动画
3. 价格设置，使用了步进器

刚刚上面之所以说这几个点，因为他们都是同出一源--vant组件

此组件的使用教程可直接看对应官网

[https://youzan.github.io/vant-weapp/][3]

使用组件开发效率会高很多，避免重复工作，同时可以参考部分组件的写法，还是有很多值得学习的地方的。


* textarea小注意

步骤二中备注信息那里使用了层级最高的原生组件textarea，这里有个特别使用注意项：如果下面tabbar是自己写的而非使用的自带原生的tabbar，会出现穿透现象，如下图示例：

![textarea层级穿透示例][4]


我常用的解决办法，通过动态改变textarea的聚焦状况，当点击该区域时，设置聚焦显示真实textarea，当失焦之后，展示为view层，代码如下：

``` html

<view class="beibox">
      <view wx:if="{{!focus}}" bindtap="focus" >{{beizhu?beizhu:'请输入信息'}}</view>
      <textarea wx:if="{{focus}}" focus="{{focus}}" bindblur="loose" bindinput="beiInput" value="{{beizhu}}"></textarea>
</view>
```

``` javascript
      data: {
            beizhu:'',
            focus: false //默认不聚焦
      }
    //点击聚焦显示textarea隐藏view
      focus() {
            let that = this;
            that.setData({
                  focus: true
            })
      },
      //失焦隐藏textarea显示view
      loose() {
            let that = this;
            that.setData({
                  focus: false
            })
      },

```

## 三：首页

![首页图][5]


上面左图是首页的进入后的初始样式，右图是下滑之后的动态页面，关于页面的样式布局方面，使用flex可以轻松搞定，我们重点说下面这点：

* 监控屏幕滚动实现动态响应

在上图第二张示例图中，随着页面下滑，顶部分类栏也随之置顶，下方也出现了一个返回顶部按钮，实现原理：

> 监控屏幕下滑高度，当大于我们设定的某个值时，元素进行渲染

这里我们需要使用页面的一个事件处理函数：onPageScroll

``` javascript
//监测屏幕滚动
      onPageScroll: function(e) {
            this.setData({
                  scrollTop: (e.scrollTop) * （wx.getSystemInfoSync().pixelRatio)
            })
      },
```
1. 函数获取的是页面在垂直方向已滚动的距离（单位px），但我们页面布局使用了rpx计算，所以后面我们乘以设备像素比获取对应的rpx值
2. 在view视图层中通过wx:if或者hidden进行控制显隐，区别在于：wx:if每次隐藏都是销毁了，而hidden只是不呈现，但依旧渲染到页面，具体的使用效果，可查看视图调试处的效果。

下面给个完整的返回顶部示例

``` html
<view class="totop" bindtap="gotop" hidden="{{ scrollTop<500 }}">
       <image  lazy-load src="/images/top.png" />
</view>
```

``` javascript
      data: {
            scrollTop: 0 //初始滚动高度为0
      },
        //监测屏幕滚动
      onPageScroll: function(e) {
          this.setData({
              scrollTop: parseInt((e.scrollTop) * wx.getSystemInfoSync().pixelRatio)
          })
      },
      //返回顶部
      gotop() {
            wx.pageScrollTo({
                  scrollTop: 0
            })
      },
```

## 四：详情页面

![详情页面效果图][6]


1. 小程序布局只要掌握一个flex，基本上就够了，所以这里不过多阐述样式问题，到时候如果有疑问可查看完整demo，都有注释的。

2. 因为此小程序的使用对象及功用限制，所以和完整的商城相比少了一个购物车功能，支付购买在商品详情页即完成了，这里涉及到两个点，一是下单购买，二是购买之后的通知问题。

* 小程序内支付提现

不仅仅是支付包括提现，此程序都借助了tenpay这个模块，详细介绍：

[https://www.npmjs.com/package/tenpay][7]


在小程序中的实例使用，可以参考之前社区之前发布的文章：

[10行代码实现小程序支付功能！丨实战][8]


当然，之前文章是教大家如何实现支付，关于提现流程也一样，先去看看tenpay的商户付款到余额的说明，再看一下此程序的相关代码，读一遍准能懂。

* 发送通知

 1. 此程序通知分为两类：短信通知、邮件通知
 2. 使用场景：用户下单后，对卖家进行短信+邮件通知，下单后订单状态改变使用邮件通知。
 
 > 说一点题外话：小程序有一个自带的模板通知，在用户主动触发后7天内能推送模板信息，之前写这个程序的时候慎重考虑过，最后还是舍弃了，毕竟七天时间，不是每本书都那么畅销的。

邮件只需要有一个账户即可，短信通知却是要成本的，当然效果要比邮件好，配置起来的话，难度都一样，我们就以短信为例：

1. 首先去腾讯云申请短信API:

[https://cloud.tencent.com/product/sms][9]

![腾讯云短信申请][10]

按照提示操作，设置好短信签名，模板等。

2. 配置云函数

新建sms云函数，代码如下：


``` javascript
    const cloud = require('wx-server-sdk')
    const QcloudSms = require("qcloudsms_js")
    const envid = 'zf-shcud'; //云开发环境id
    const appid = 140000001 // 替换成您申请的云短信 AppID 以及 AppKey
    const appkey = "abcdefghijkl123445"
    const templateId = 1234 // 替换成您所申请模板 ID
    const smsSign = "腾讯云" // 替换成您所申请的签名
    cloud.init({
      env: envid,
    })
    // 云函数入口函数
    exports.main = async (event, context) => new Promise((resolve, reject) => {    
      /*单发短信示例为完整示例，更多功能请直接替换以下代码*/
      var qcloudsms = QcloudSms(appid, appkey);
      var ssender = qcloudsms.SmsSingleSender();
      var params = ["测试内容"];
      // 获取发送短信的手机号码
      var mobile = event.mobile
      // 获取手机号国家/地区码
      var nationcode = event.nationcode
      ssender.sendWithParam(nationcode, mobile, templateId, params, smsSign, "", "", (err, res, resData) => {
          /*设置请求回调处理, 这里只是演示，您需要自定义相应处理逻辑*/
          if (err) {
            console.log("err: ", err);
            reject({ err })
          } else {
            resolve({ res: res.req, resData })
          }
        }
      );
    })
```

> 提一个小点：在有多个云环境时候，如果涉及到查询云数据库等和云环境有直接干系的操作时候，最好在cloud.init({env: envid})这里声明一下环境，否则有小几率报错。

五、启动页设计

![启动页效果图][11]


  [1]: https://cqu.oss-cn-shenzhen.aliyuncs.com/img/tencent/book/login.png
  [2]: https://cqu.oss-cn-shenzhen.aliyuncs.com/img/tencent/book/publish.jpg
  [3]: https://youzan.github.io/vant-weapp/
  [4]: https://cqu.oss-cn-shenzhen.aliyuncs.com/img/tencent/book/tabbar.jpg
  [5]: https://cqu.oss-cn-shenzhen.aliyuncs.com/img/tencent/book/index.jpg
  [6]: https://cqu.oss-cn-shenzhen.aliyuncs.com/img/tencent/book/detail.png
  [7]: https://www.npmjs.com/package/tenpay
  [8]: https://mp.weixin.qq.com/s/BAyHBPEaKCKTBPXBJkI6Ng
  [9]: https://cloud.tencent.com/product/sms
  [10]: https://cqu.oss-cn-shenzhen.aliyuncs.com/img/tencent/book/duanxin.png
  [11]: https://cqu.oss-cn-shenzhen.aliyuncs.com/img/tencent/book/start.png

启动页也算本程序一个亮点，首次进入就是一张美美的图给人一种身心愉悦之感，下面我们就详细说说这个怎么做：

哪些元素？

1. 全屏背景图
2. 倒计时跳转

说这个之前，大家注意一下整个页面是全屏了的，所以这里我们要配置一下页面参数：

在此页面的.json中这么配置：

``` json
{
  "navigationStyle":"custom"
}
```

这就成功全屏了，接着我们来编写页面样式：

``` html
<view class="contain">
     <view class="go">
             <button  bindtap="go">跳过{{count}}s</button> 
     </view>
     <image class="bg" src="{{bgurl}}"></image>
</view>
```


``` stylus
.contain {
      width: 100%;
      height: 100%;
      position: relative;
}
.bg {
      position: absolute;
      left: 0rpx;
      top: 0rpx;
      width: 100%;
      height: 100%;
      z-index: -1;
}
.go {
      position: absolute;
      right: 30rpx;
      top: 150rpx;  
      z-index: 9;
}
.go button {
      font-size: 28rpx;
      letter-spacing: 4rpx;
      border-radius: 30rpx;
      color: #000;
      background: rgba(255, 255, 255, 0.781);
       display: flex;
      justify-content: center;
      align-items: center;
      text-align: center;
      width: 160rpx;
      height: 60rpx;
}

```

样式快速搞定，再来说说js部分。

* 倒计时功能：

``` javascript
countDown: function() {
            let that = this;
            let total = 3;//倒计时总数3秒
            this.interval = setInterval(function() {
                  total > 0 && (total--, that.setData({
                        count: total
                  })), 0 === total && (that.setData({
                        count: total
                  }), wx.switchTab({
                        url: "/pages/index/index"
                  }), clearInterval(that.interval));
            }, 1e3);
      },
```

* 背景图

1. 实现有两种办法，第一是本地路径，第二是引用远程地址（可通过接口动态改变）

2. 第一种好处是直接使用本地图片，加载速度快，第二种可以随时更换启动图，两种办法都试过了，最终我建议还是采用第一种办法，使用本地图片，如果使用远程地址，首次进入会出现短时间白屏，体验不好，当然，你也可以想办法把图片压缩再压缩，那就不存在加载慢了，但分辨率又成了个问题，所以具体如何使用，还是根据产品需求。

## 总结

纸上得来终觉浅，绝知此事要躬行，以上总结的是开发此程序中我认为遇到的典型问题，实践过程中肯定会有更多有意思的问题的出现，“面向百度”编程是一个方面，但我更建议“面向官方文档”，很多问题其实官方文档中都有很详细的说明和代码示例，如果阅读文档颇感费力，我建议你该静下心来，先熟悉下html,css,javascript相关内容，到时候再回过头来看你会发现“原来如此”。

# 源码链接
https://github.com/xuhuai66/used-book-pro
