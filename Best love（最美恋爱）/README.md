![](https://puui.qpic.cn/vupload/0/20190807_1565159329722_xl8xl1g3q5b.png/0)

# 一、前言

   像每一滴酒回不了最初的葡萄，我回不到年少。爱情亦是如此，这就是写一篇小程序的初衷，用来记录我和她最美的恋爱。什么是最美恋爱？就是繁忙之余的一封书信，一起奋斗的目标，精彩的瞬间，旅游的足迹，和那无数的纪念日。

言归正传吧，先看看小程序给你的第一印象。（截图的是体验版本，上线版本有些功能是没有上的哦）

![](https://upload-images.jianshu.io/upload_images/4252197-f72b87df57571e3b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/4252197-88c0729c04610300.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/4252197-5899628618562b45.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/4252197-09aadf9f0d9dfca7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**页面比较简约，她很喜欢。**

# 二、说说代码

## 1.框架

小程序前端用的是taro框架写的，后台用的云开发（简直是个人开发者的福音）。
贴一下总体架构图：

![](https://upload-images.jianshu.io/upload_images/4252197-552bb6542d2b67b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


其他的架构，页面等等都很常见，我具体来说说云函数的调用吧，主要是对数据库的操作：

![](https://upload-images.jianshu.io/upload_images/4252197-43e6773fbbb1cb2f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

云函数的入口（运用TcbRouter实现不同方法的调用）：

![](https://upload-images.jianshu.io/upload_images/4252197-51e1cc0f7f70d42e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

小程序端是这样调用的：

![](https://upload-images.jianshu.io/upload_images/4252197-8a018a5b2eab4989.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

调用方法的参数：

```
  let param = {
      method:'get',
      collection:'mail',
      id:auth.user._id,
      bindId:auth.user.bindId,
      start:this.start,
      limit:PAGE.LIMIT
    };

    let res = await commonApi.list(param);
```

# 三、说说功能

主要来说说邮箱这个功能吧，毕竟现在写信的越来越少了，这里很大程度的还原了写信的过程，可以挑选信封，挑选邮票，然后寄出你的思恋。
我已经收到这么多信了 你们呢？

![](https://upload-images.jianshu.io/upload_images/4252197-aa5f6688fb9471d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/4252197-8d490dae4b574c73.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

还有个留言板功能，她说和微信聊天有什么区别（区别就在于没有websocket），这里就不赘述啦。



# 四、结语
七夕已至，快和亲爱的人绑定最美恋爱关系吧！在这里，你们就是导演，记录美好爱情。
特别说明：此小程序，是我亲手为女朋友写的，感谢她提供需求支持，七夕快乐。

❤️ 小彩蛋 ❤️

使用小程序·云开发按以下四步操作即可查看惊喜彩蛋~

在cloudfuntions文件夹下，点击右键新建一个云函数love，并部署上传，
```javascript
exports.main = async (event, context) => {
  return {
    timelimit: 10000,
  }
}
```
使用开发者工具新增一个小程序页面love，在love.wxml里输入以下代码

```html
<view>你问我爱你多少年？</view>
<image bindtap="loveFunction" src="{{loveurl}}"></image>
<view wx:if="{{timelimit}}">
  <view><text>我爱你{{timelimit}}年</text></view>
  <view><text>云开发祝所有程序员七夕脱单</text></view>  
</view>
```
在love.js里输入：
```javascript
  data: {
    timelimit:'',
    loveurl:"https://tcb-1251009918.cos.ap-guangzhou.myqcloud.com/love.png",
  },

  loveFunction() {
    wx.cloud.callFunction({
      name: 'love',
      data: {
      },
      success: res => {
        this.setData({
          timelimit: JSON.stringify(res.result.timelimit)
        })
      }
    })
  },
```
在love.wxss里输入：
```css
view{text-align: center;margin: 20px 0;font-weight: bold;}
text{color: #FFC0CB;}
image{width: 100rpx;height: 100rpx}
button{background-color: #FFC0CB;color:#fff;width: 300rpx}
```

# 源码链接
[https://github.com/uloveits/loveing](https://github.com/uloveits/loveing)
