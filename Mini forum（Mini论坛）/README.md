---
title: 【小程序+云开发】实战：一天搭建小型论坛
categories:
  - 软件开发
tags:
  - 小程序
abbrlink: 43024
date: 2018-11-25 15:10:40
---
笔者最近涉猎了小程序相关的知识，于是利用周末时间开发了一款类似于同事的小程序，**深度体验**了小程序云开发模式提供的**云函数、数据库、存储**三大能力。关于云开发，可参考文档：[小程序·云开发](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/basis/getting-started.html)。</br>

个人感觉云开发带来的最大好处是**鉴权流程的简化和对后端的弱化**，所以像笔者这种从未接触过小程序开发的人也能够在周末两天时间内开发出一个功能完备、体验闭环的勉强能用的产品。</br>

最后，本文并不是搬运官方文档，也不会详细介绍开发工具和云开发后台的使用，所以建议结合上面给出文档链接一起消化本文。
<!--more-->

# 功能分析

![](https://s1.ax1x.com/2018/11/25/FkduQg.gif)

该小程序功能目前较为简单（发布帖子、浏览帖子、发布评论），可用下图表示，无需赘述：

![](https://s1.ax1x.com/2018/11/25/FkdVFP.png)

由架构图可知，云开发的数据库（存帖子、存评论）、存储（图片）、云函数（读、写、更新数据库等）都将涉及，**很好地达到了练手的目的**。

# 发布帖子

如果帖子不带图片，直接写数据库即可，如果带图片则需要先存入图片到云开发提供的存储中，拿到返回的fileId（可理解为图片的url）再一并写入数据库，核心代码：

```js
    for (let i = 0; i < img_url.length; i++) {
      var str = img_url[i];
      var obj = str.lastIndexOf("/");
      var fileName = str.substr(obj + 1)
      console.log(fileName)
      wx.cloud.uploadFile({
        cloudPath: 'post_images/' + fileName,//必须指定文件名，否则返回的文件id不对
        filePath: img_url[i], // 小程序临时文件路径
        success: res => {
          // get resource ID: 
          console.log(res)
          //把上传成功的图片的地址放入数组中
          img_url_ok.push(res.fileID)
          //如果全部传完，则可以将图片路径保存到数据库
          if (img_url_ok.length == img_url.length) {
            console.log(img_url_ok)
            that.publish(img_url_ok)
          }
        },
        fail: err => {
          // handle error
          console.log('fail: ' + err.errMsg)
        }
      })
    }  
```

通过`img_url_ok.length == img_url.length`我们确定所有图片已经上传完成并返回了对应的id，然后执行写入数据库的操作：

```js

  /**
   * 执行发布时图片已经上传完成，写入数据库的是图片的fileId
   */
  publish: function(img_url_ok) {
    wx.cloud.init()
    wx.cloud.callFunction({
      name: 'publish_post',
      data: {
        openid: app.globalData.openId,// 这个云端其实能直接拿到
        author_name: app.globalData.userInfo.nickName,
        content: this.data.content,
        image_url: img_url_ok,
        publish_time: "",
        update_time: ""//目前让服务器自己生成这两个时间
      },
      success: function (res) {
        // 强制刷新，这个传参很粗暴
        var pages = getCurrentPages();             //  获取页面栈
        var prevPage = pages[pages.length - 2];    // 上一个页面
        prevPage.setData({
          update: true
        })
        wx.hideLoading()
        wx.navigateBack({
          delta: 1
        })
      },
      fail: console.error
    })
  },
```

通过`wx.cloud.callFunction`我们调用了一个**云函数**（通过`name`指定函数名），并将帖子内容`content`和图片`image_url`以及其他信息（发布者昵称、id等）一并传到云端。然后再看看这个云函数：

```js
exports.main = async (event, context) => {
  try {
    return await db.collection('post_collection').add({
      // data 字段表示需新增的 JSON 数据
      data: {
        // 发布时小程序传入
        //author_id: event.openid,不要自己传，用sdk自带的
        author_id: event.userInfo.openId,
        author_name: event.author_name,
        content: event.content,
        image_url: event.image_url,
        // 服务器时间和本地时间会造成什么影响，需要评估
        publish_time: new Date(),
        // update_time: event.update_time,// 最近一次更新时间，发布或者评论触发更新,目前用服务器端时间
        update_time: new Date(),
        // 默认值，一些目前还没开发，所以没设置
        // comment_count: 0,//评论数，直接读数据库，避免两个数据表示同一含义
        watch_count: 3,//浏览数
        // star_count: 0,//TODO：收藏人数
      }
    })
  } catch (e) {
    console.error(e)
  }
}

```

可以看到，云函数写入了一条数据库记录，我们的参数通过`event`这个变量带了进来。

# 获取帖子列表

所谓获取帖子列表其实就是读上一节写入的数据库，但是我们并不需要全部信息（例如图片url），并且要求按照时间排序，如果熟悉数据库的话，会发现这又是一条查询语句罢了：

```js
exports.main = async (event, context) => {
  return {
    postlist: await db.collection('post_collection').field({// 指定需要返回的字段
      _id: true,
      author_name: true,
      content: true,
      title: true,
      watch_count: true
    }).orderBy('update_time', 'desc').get(),//指定排序依据

  }
}
```

# 浏览帖子内容

浏览帖子内容及给定一个帖子的id，由帖子列表点击时带入：

```js
  onItemClick: function (e) {
    console.log(e.currentTarget.dataset.postid)
    wx.navigateTo({
      url: '../postdetail/postdetail?postid=' + e.currentTarget.dataset.postid,
    })
  },
```

然后在云函数中根据这个id拿到全部数据：

```js
exports.main = async (event, context) => {
  
  return {
    postdetail: await db.collection('post_collection').where({
      _id: event.postid
    }).get(),
  }
}
```

拿到全部数据后，再根据图片id去加载贴子的图片：

```
    // 获取内容
    wx.cloud.callFunction({
      // 云函数名称 
      name: 'get_post_detail',
      data: {
        postid: options.postid
      },
      success: function (res) {
        var postdetail = res.result.postdetail.data[0];
        that.setData({
          detail: postdetail,
          contentLoaded: true
        })
        that.downloadImages(postdetail.image_url)
      },
      fail: console.error
    })

```

这里`that.downloadImages(postdetail.image_url)`即加载图片：

```js
  /**
   * 从数据库获取图片的fileId，然后去云存储下载，最后加载出来
   */
  downloadImages: function(image_urls){
    var that = this
    if(image_urls.length == 0){
      return
    } else {
      var urls = []
      for(let i = 0; i < image_urls.length; i++) {
        wx.cloud.downloadFile({
          fileID: image_urls[i],
          success: res => {
            // get temp file path
            console.log(res.tempFilePath)
            urls.push(res.tempFilePath)
            if (urls.length == image_urls.length) {
              console.log(urls)
              that.setData({
                imageUrls: urls,
                imagesLoaded: true
              })
            }
          },
          fail: err => {
            // handle error
          }
        })
      }
    }
  },
```

# 发表评论

发表评论和发布帖子逻辑类似，只是写入的数据不同，不做赘述。

# 总结

前面说过，云开发弱化了后端（简化鉴权本质也是弱化后端），这样带来的好处就是提高了开发效率，因为前后端联调向来都是一件耗时间的事情，而且小程序本身主打的就
是小型应用，实在没有必要引入过多的开发人员。但云开发也不是万能的，例如我一开始想做RSS阅读器，那么后端就需要聚合信息，目前云开发还做不了。

个人感觉只要是信息类的小程序，如新闻类、视频类，云开发目前都很乏力，因为数据库的支持还过于简陋（也可能是我太菜，没发现很好的解决办法，欢迎拍砖）。但如果是本文提及的这种用户自己也会产生信息的小程序，那么云开发则会有开发效率上的优势。

最后就是云开发目前提供的2G数据库和5G存储，对于一些用户量较多的小程序是否足够也是个问题，目前也没见有付费版。</br>

总的类说，初次接触小程序开发，还是发现有不少值得借鉴学习之处。

## 源码链接

[https://github.com/vimerzhao/RssHub](https://github.com/vimerzhao/RssHub)
