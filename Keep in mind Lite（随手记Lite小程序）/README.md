**一、做一款轻便的备忘录小工具**
>有次在外地出差开会，台上的演讲者妙语连珠，分享的颇具启发性，我想把这些分享都记录下来，但一时找不到合适的记录地方，于是和身边大多数人一样，打开微信在 `文件传输助手` 或 `亲人的微信` 开始记录。但是这种形式的记录没有形成一个完整的记录体系，且时间久了很难再找到。 

>会后我在软件市场找了一圈，都没有找到特别合适的软件应用：一方面它们功能太繁琐复杂，远超我的需求；另一方面像这种不是特别高频使用的 `APP` ，说实话，我真不太愿意专门为此下载安装。 

>于是我想到小程序 —— `轻量` 、 `便捷` 、 `即开即用` ，用小程序开发这么一款这样做备忘录的小工具，非常合适。

**二、主要功能**
- 创建备忘录：内容快速记录，支持表情和图片，自动获取标题和时间，可选择记录位置
- 备忘录查询：历史记录按时间排序，允许记录回查，导航到记录位置
- 备忘录修改：允许重复编辑修改

**三、随手记Lite功能实现**
#### 3.1、准备工作
>**1、注册微信小程序账号：**

>方式一：直接注册（https://mp.weixin.qq.com/wxopen/waregister?action=step1）

>方式二：已经有微信公众号（已认证）朋友可以直接【登录公众号】 -> 【小程序管理】 -> 【添加】->【快速注册并认证小程序】

>注册完成后，找到小程序的 `ApID` 和 `AppSecret`

>![](https://puui.qpic.cn/vupload/0/20190725_1564021363630_9khzujvh5wg.jpeg/0)

>2、**下载微信开发者工具、创建项目** ，打开开发者工具，键入项目目录、项目名称、刚才的 AppID，此时项目创建成功，然后点击开发者工具上方的【云开发】开通云开发。小程序·云开发官方地址（<https://developers.weixin.qq.com/miniprogram/dev/wxcloud/basis/getting-started.html>）

#### 3.2功能实现一：

#### 创建备忘录（用户文本及图片内容的上传存储）

1）功能简要描述：对于随手记Lite来说，为用户提供文本及图片上传，快速记录，是最基本的功能

2）从用户端到云开发的信息处理流程图及描述

![](https://puui.qpic.cn/vupload/0/20190725_1564021586722_ucd4l54h10m.png/0)

>如果记录中包含图片，则先上传图片，将返回的 `fileID` 添加到数组，最后将数组和记录内容上传到云数据库；如果记录中不包含图片则直接将记录内容上传到云数据库。

3）功能核心代码
```javascript
// 用户填写信息后，提交事件
 fromSubmit: function(e) {
   let submitInfo = e.detail.value
   let categoryId = Number(submitInfo.categoryIndex)
   var title = submitInfo.title ? submitInfo.title : this.data.datas.title2
   var images = [];
   for (var i = 0; i < this.data.uploadImages.length; i++) {
     let randString = Math.floor(Math.random() * 1000000).toString() + '.png'
     wx.cloud.uploadFile({
     // 以昵称创建云存储文件夹路径
     cloudPath: wx.getStorageSync("wxUserInfo").nickName + '/' + Date.now() + '.png',
     filePath: this.data.uploadImages[i],
     success: res => {
       images = images.concat(res.fileID)
       // 图片全部上传完毕，上传记录数据
       if (images.length == this.data.uploadImages.length) {
       db.collection('allnote').add({
         data: {
          "titlename": title,
          "createtime": util.formatTime2(new Date()),
          "updatetime": null,
          "address": submitInfo.locationName ? submitInfo.locationName : "未选择位置",
          "latAndLong": this.data.datas.latAndLong,
          "content": submitInfo.content,
          "images": images,
          'nickName': wx.getStorageSync("wxUserInfo").nickName,
          'avatar': wx.getStorageSync("wxUserInfo").avatarUrl,
          'province': wx.getStorageSync("wxUserInfo").province,
        }
       })
       .then(res => {
         wx.hideLoading()
         wx.navigateBack({})
          })
        }
        },
       fail: console.error
    })
   }
 }
},
```
#### 3.2 功能实现二：记录查询功能

1）功能简要描述：已经创建备忘录的用户，可以按照时间顺序查看所有记录及任意记录的详细内容

2）从用户端到云开发的信息处理流程图及描述

![](https://puui.qpic.cn/vupload/0/20190725_1564021980542_z1l2wj40xke.png/0)

>小程序端用户上传自己的 `userId` 到云函数，云函数根据该 `userId` 到云数据库请求对应数据，数据返回到云函数后，云函数进行排序和时间格式截取处理，最后将数据返回到小程序端。

3）功能核心代码
```javascript
<!--云函数端-->
// 云函数入口函数
exports.main = async(event, context) => {
  return await new Promise(function(resolve, reject) {
    var notes = []
    var notesNew = []
    db.collection('allnote').where({
      _openid: event.userOpenid
    }).get().then(res => {
      console.log(res)
      notes = res.data
      if (res.data) {
        // 由于时间格式问题，orderBy('createtime', 'desc')排序无效，所以使用以下排序方式
        // 按照时间倒叙排列
        notes.sort(function(a, b) {
          return Date.parse(b.createtime) - Date.parse(a.createtime); 
        });
        // 显示的时间格式转换，截取秒
        for (var i = 0; i < res.data.length; i++) {
          let showTime = res.data[i].createtime.substring(0, res.data[i].createtime.length - 3)
          notes[i].showCreateTime = showTime
          // 数据解码
          notes[i].titlename = decodeURI(notes[i].titlename)
          // 将加入新格式的时间放入新的数组
          notesNew.push(notes[i])
        }
        // 返回新的数组
        resolve(notesNew)
      }
    }).catch(error => {
      reject(error)
    })
  })
}

<!--小程序端-->
  // 请求心得数据
  requestNoteData: function() {
    let _this = this;
      wx.cloud.callFunction({
      name: 'getMyNote',
      data: {
        userOpenid: wx.getStorageSync('userId'),
        skip:0
      }
    }).then(res => {
      console.log(res)
      _this.setData({
        notes: res.result
      })
    }).catch(err => {
      console.log(err)
    })
  },
```

#### 3.3 功能实现三：备忘录修改功能

1）功能简要描述：对于记录详细内容，用户除了可以查看时间、位置和内容之外，还可以进行内容的编辑和删除记录

2）从用户端到云开发的信息处理流程图及描述

![](https://puui.qpic.cn/vupload/0/20190725_1564022141349_nqempgfwbx.png/0)

>用户修改记录时，直接上传新的记录内容和记录id调用 `.update` 进行即可。

3）功能核心代码
```javascript
<!--记录详情页数据获取-->
onLoad: function(options) {
    // 根基记录id获取记录详情
    db.collection('allnote')
      .where({
        _id: options._id
      })
      .get()
      .then(res => {
        console.log(res)
        noteData = res.data[0]
        // 显示的时间格式转换，截取秒
        let showTime = res.data[0].createtime.substring(0, res.data[0].createtime.length - 3)
        noteData.createtime = showTime
        // 数据解码，如果上传的时候没有encodeURI，则不需要decodeURI
        noteData.titlename = decodeURI(noteData.titlename)
        noteData.content = decodeURI(noteData.content)
        _this.setData({
          noteData: noteData
        })
        wx.hideLoading()
      })
  },

  
<!--更新单条记录-->
  // 单条记录提交更新
  fromSubmit: function(e) {
    let _this = this;
    var submitInfo = e.detail.value
    db.collection('allnote').doc(_this.data.noteData._id).update({
        data: {
           titlename: submitInfo.name,
           content: submitInfo.content
        }
     })
       .then(res => {
           wx.navigateBack({
          })
      })
      .catch(console.error)
  },
```

**四、小结**
>其实一开始，我的随手记LIte小程序是计划用小程序+WEB后台开发实现的，从买域名(备案···)到后台开发环境搭建占到了这个项目的2/3的时间，且域名、服务器、CA证书都需要管理续费。 

>有一天我收到公众号推送小程序·云开发的消息，打开一看太惊喜了，用1天时间了解学习了下，第二天我便将后台切换到了云开发。 

>我可以将大部分精力都放在业务的实现上，可以说是零部署，零维护，也不需要操心域名和服务器相关的东西，而且云控制台的数据和文件都是可视化管理，用户登录寥寥几行代码就可实现，常规的数据读写都封装好了接口。

>期待小程序·云开发开放出更多的接口和功能···

**五、项目预览**

![](https://puui.qpic.cn/vupload/0/20190725_1564022284995_agncbb0qk7.jpeg/0)
