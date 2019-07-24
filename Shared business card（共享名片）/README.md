# 一、前言

从一个较早的小程序开发者到第一批使用小程序·云开发的开发者，这期间一直在关注关于小程序各方面的更新，同时也用小程序·云开发做了几款产品，其中包括上次分享的[随手记Lite小程序](https://mp.weixin.qq.com/s?__biz=Mzg3NTA1NjcyNQ==&mid=2247483701&idx=1&sn=0176e67b7647f6632fe0ce1721beb961&chksm=cec61a0ff9b1931963c221a5fdb3c58ef7a22cf0fc4c58e120672943c9c5d29f63ec65b7189d&token=1242218487&lang=zh_CN&scene=21#wechat_redirect)，比较上次，这次分享的技术点相对更加全面和实用一些。

> 涉及的技术点有：

- 数据上传、数据更新、分页读取、数据删除，AI智能名片识别读取。
- 单图上传、多图上传，图片URL获取，带参小成码生成。
- 下发模板消息，云调用使用。

# 二、主要功能

- 创建电子名片：**信息存储，图片上传，名片读取（AI智能名片识别）**
- 转发电子名片：**专属名片海报（带参小程序码生成）**
- 电子名片被访问：**下发模板消息（云调用）**

# 三、功能实现

## 3.1、准备工作

1、注册微信小程序账号：

> 方式一：直接注册（[https://mp.weixin.qq.com/wxopen/waregister?action=step1](https://mp.weixin.qq.com/wxopen/waregister?action=step1)） 
> 方式二：已经有微信公众号（已认证）朋友可以直接【登录公众号】 -> 【小程序管理】 -> 【添加】->【快速注册并认证小程序】，注册完成后，找到小程序的 AppID和 AppSecret

![](https://ask.qcloudimg.com/http-save/4744530/7ju2s9l158.webp)

2、下载微信开发者工具、创建项目 ，打开开发者工具，键入项目目录、项目名称、刚才的 AppID，此时项目创建成功，然后点击开发者工具上方的【云开发】开通云开发。

## 3.2功能实现一：【创建电子名片】

#### 信息存储，图片上传，名片读取（AI智能名片识别）

### 1.功能简要描述

对于一个名片的小程序，第一步肯定是创建电子名片，除此之外，可以用传统信息录入的方式创建名片，同时也支持纸质名片的识别读取，快速创建名片，这里本地需要导入 `mapping.js`框架，接下来以纸质名片识别为例。

### 2.核心代码

```
  // 上传名片后获取零时链接
  getTempFileURL() {
    wx.cloud.getTempFileURL({
      fileList: [{
        fileID: this.data.fileID,
      }],
    }).then(res => {
      console.log('获取成功', res);
      if (res.fileList.length) {
        this.setData({
          coverImage: res.fileList[0].tempFileURL
        }, () => {
          this.parseNameCard();
        });
      } else {
        Toast('获取图片地址失败');
      }
    }).catch(err => {
      Toast('获取图片地址失败');
    });
  },
  // 读取名片
  parseNameCard() {
    wx.cloud.callFunction({
      name: 'parseCard',
      data: {
        url: this.data.coverImage
      }
    }).then(res => {
      if (res.result.data.length == 0) {
        Toast('解析失败，请上传【纸质名片】或【手动创建】');
        return;
      }
      let data = this.transformMapping(res.result.data);
      wx.setStorageSync("parseCardData", data)
      Toast('解析成功');
    }).catch(err => {
      console.error('解析失败，请上传【纸质名片】或【手动创建】', err);
      Toast('解析失败，请上传【纸质名片】或【手动创建】');
    });
  },

  // 名片数据解析
  transformMapping(data) {
    let record = {};
    let returnData = [];
    data.map((item) => {
      let name = null;
      if (mapping.hasOwnProperty(item.item)) {
        name = mapping[item.item];
        // 写入英文名
        item.name = name;
      }
      return item;
    });
    // 过滤重复的字段
    data.forEach((item) => {
      if (!record.hasOwnProperty(item.item)) {
        returnData.push(item);
        record[item.item] = true;
      }
    });
    return returnData;
  },
```

## 3.3功能实现二：【转发电子名片】

#### 专属名片海报（带参小程序码生成）

### 1.功能简要描述：转发电子名片有两种方式。

> 1.以小程序的形式直接转发给好友或微信群。
> 2.生成专属名片海报分享到朋友圈长按进入对应的电子名片页面。名片海报上除了有对应用户的姓名之外，还有专属的名片小程序码，效果如下：

![](https://ask.qcloudimg.com/http-save/4744530/1tcjbf9m3q.webp)

### 2.核心代码

```
const cloud = require('wx-server-sdk')
const axios = require('axios')
var rp = require('request-promise');
cloud.init()

// 云函数入口函数，小程序端传过来页面和名片id
exports.main = async (event, context) => {
  console.log(event)
  try {
    const resultValue = await rp('https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=appid&secret=secret')
    const token = JSON.parse(resultValue).access_token;
    const response = await axios({
      method: 'post',
      url: 'https://api.weixin.qq.com/wxa/getwxacodeunlimit',
      responseType: 'stream',
      params: {
        access_token: token,
      },
      data: {
        page: event.page,
        width: 300,
        scene: event.id,
      },
    });

    return await cloud.uploadFile({
      cloudPath: 'xcxcodeimages/' + Date.now() + '.png',
      fileContent: response.data,
    });
  } catch (err) {
    console.log('>>>>>> ERROR:', err)
  }
}
```

## 3.4功能实现三：【电子名片被访问】

#### 下发模板消息（云调用）

### 1.功能简要描述

用户名片被访问的时候，用户者会收到【客户来访提醒】的模板消息，同时提醒用户完善名片信息。

### 2.核心代码

```
const cloud = require('wx-server-sdk')
cloud.init()
exports.main = async (event, context) => {
  try {
    const result = await cloud.openapi.templateMessage.send({
      touser: event.toUser,
      page: "pages/index/index",
      data: {
        keyword1: {
          value: event.visitDate
        },
        keyword2: {
          value: "刚刚有人深度访问了您的名片，经常完善名片信息，更容易被查找和访问。"
        },
      },
      templateId: 'templateId',
      formId: event.formId,
    })
    return result
  } catch (err) {
    throw err
  }
}
```

# 四、总结

> 和传统的小程序 + WEB后台开发模式比起来，云开发在精力和人力上真的是节省了很多，这能使开发者将大部分精力和时间放到功能的开发上。
> 云开发上线时间不算太长，但逐步有新的功能开放出来，比如云控制台数据的导入导出、云调用等，希望小程序·云开发开放出更多的接口和功能......

# 五、项目预览

![](https://ask.qcloudimg.com/http-save/4744530/7crisu2p1h.webp)
