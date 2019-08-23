用小程序·云开发将博客小程序常用功能“一网打尽”

> 本文介绍mini博客小程序的详情页的功能按钮如何实现，具体包括评论、点赞、收藏和海报功能，这里记录下整个实现过程和实际编码中的一些坑。

## 评论、点赞、收藏功能

## 实现思路

实现文章的一些操作功能，最主要的还是评论，这是作者和读者之间沟通的桥梁，评论功能的衍生无非是细化作者和读者之间的互动，或者增加文章的传播，所以在动手开发时需要思考下你期望实现哪些功能，并对应功能进行细化。

我一般的经验是，先在脑子里过一遍需要的功能和大致流程，然后在笔记稍微画下「最最基础的原型，相当于产品的角色」。

然后就开始直接开始搭建页面和简单的交互「使用假数据，优先完成页面」，在构造页面的时候其实也能够补充最初想法上一些流程上的缺陷，这样在设计后端和数据库结构的时候可以补上，整体下来也基本比较完善了。

回头看我的小程序的需求，首先肯定是操作，在文章底部需要有个操作栏，用于发送点评和其他一些操作，在参考了一些同类型的小程序之后，逐步实现自己的一套风格，样式截图如下：

![](http://image.bug2048.com/1557467850284.jpg)

在有了功能之后，点评的数据需要有地方展示「通常是文章底部」，然后就有了文章底部的评论列表，样式如下：

![](https://puui.qpic.cn/vupload/0/1566546398576_93lp95m72vm.jpeg/0)

既然有`点赞`和`收藏`的功能按钮，是否用户需要看下我点赞和收藏的文章列表呢，所以在「我的」中就有相应的列表，样式如下：

![image](http://image.bug2048.com/1557468695860.jpg)

到这里，最最基础的功能基本差不多，接下来就要看后端是否能支持这些页面了「主要就是数据的保存和展示了」

对于评论来说，肯定需要一个集合用于保存用户的评论，而对于用户的喜欢和收藏也需要一个集合来进行保存。

所以根据页面我们就可以设计`mini_comments`和`mini_posts_related`两个集合。前者用于保存评论数据，后者用户保存用户操作与文章之间的关联。

剩下的工作就是变现了，无非就是页面交互和数据的增删改查了。

## 细节点解析

#### 关于评论数量

目前在文章的集合中有个`totalComments`这个属性，当这篇文章每新增一个评论时，需要加1。

最初在写这个的时候，每次都是先查再更新，两段式，原代码如下：

```
let count=post.totalComments+1;
let result =await db.collection('mini_posts').doc(event.commentContent.postId).update({
    data: {
      totalComments: count
    }
  });
```
后来看文档发现，可以使用`db.command.inc`这个指令，无需再查一遍，直接可对原字段加1，还能保证原子性。代码如下：

```
const _ = db.command
let result = db.collection('mini_posts').doc(event.commentContent.postId).update({
    data: {
      totalComments: _.inc(1)
    }
  });
```

#### 关于新增子评论

需要实现在某个评论下进行回复。

在交互上，点击评论者的昵称或头像时，触发相应的点击事件，在事件中去记录相应的评论ID及必要数据，同时去设置焦点到评论框内：

```
 /**
  * 点击评论内容回复
  */
  focusComment: function (e) {
    let that = this;
    let name = e.currentTarget.dataset.name;
    let commentId = e.currentTarget.dataset.id;
    let openId = e.currentTarget.dataset.openid;

    that.setData({
      commentId: commentId,
      placeholder: "回复" + name + ":",
      focus: true,
      toName: name,
      toOpenId: openId
    });
  },
```
利用云开发新增子评论时可以使用`db.command.push`来进行操作「更新指令，对一个值为数组的字段，往数组尾部添加一个或多个值」，往子评论集合中新增：

```
/**
 * 新增子评论
 * @param {} event 
 */
async function addPostChildComment(event) {

  let task = db.collection('mini_posts').doc(event.postId).update({
    data: {
      totalComments: _.inc(1)
    }
  });
  await db.collection('mini_comments').doc(event.id).update({
    data: {
      childComment: _.push(event.comments)
    }
  })
  await task;
}
```

#### 关于判断是否已收藏

在文章第一次加载时，我们需要判断下该用户是否有对该文章有相关操作，如果有相应的收藏和点赞操作，在初始化时需要更新相应的功能图标,核心代码如下：

```
  /**
   * 获取收藏和喜欢的状态
   */
  getPostRelated: async function (blogId) {
    let where = {
      postId: blogId,
      openId: app.globalData.openid
    }
    let postRelated = await api.getPostRelated(where, 1);
    let that = this;
    for (var item of postRelated.data) {
      if (config.postRelatedType.COLLECTION === item.type) {
        that.setData({
          collection: { status: true, text: "已收藏", icon: "favorfill" }
        })
        continue;
      }
      if (config.postRelatedType.ZAN === item.type) {
        that.setData({
          zan: { status: true, text: "已赞", icon: "appreciatefill" }
        })
        continue;
      }
    }
  },
```

至于其他一些交互细节和代码细节，可以自行阅读源码去体会，如果有任何疑问或者有更好的实现方式，也可以与我沟通。


## 海报功能

## 交代些背景

其实在最早之前的小程序中已经实现了一次，具体可以参考[利用云开发优化博客小程序（三）——生成海报功能](https://www.bug2048.com/wechat20181015/),主要还是使用原生的`cavans`进行组装，原本想代码copy过来改改就行了，但总觉得原来的代码写的不是特别好。

于是想看看是否有现成的轮子可以利用，果然发现了`wxa-plugin-canvas`这款组件，通过非常简单的配置就可以生成精美的海报。

## 小程序使用npm

在总结生成海报功能之前还是有必要记录下小程序npm的使用，避免一些不必要的坑。

考虑到小程序本身的大小限制，使用npm的方式是最佳的。

原因是根据官方文档介绍，小程序 npm 包里只有构建文件生成目录会被算入小程序包的占用空间，上传小程序代码时也只会上传该目录的代码。这样大大减少了上传的代码体积。

下面简单介绍下小程序端如何使用npm的「其实根据官方文档按照步骤就可以了」。

以我目前小程序的路径为例，在`/miniprogram`新增文件夹`node_modules`,在命令行指向到`/miniprogram`目录下：

![image](http://image.bug2048.com/1557669284694.jpg)

通过命令进行安装：

```
npm install wxa-plugin-canvas --production  
```
安装成功后，即可在小程序开发工具中进行构建，构建前需要勾选`使用 npm 模块`

![image](http://image.bug2048.com/1557669545807.jpg)

然后点击开发者工具中的菜单栏：工具 --> 构建 npm即可：

![image](http://image.bug2048.com/1557669634230.jpg)

构建完成后会生成miniprogram_npm目录，到这里，项目端基本就调通了。

![](https://puui.qpic.cn/vupload/0/1566546579976_tgmxlijrrsl.jpeg/0)

## wxa-plugin-canvas

在构建完之后，就可以正常使用wxa-plugin-canvas这个自定义组件，使用方式还是比较简单的。

首先在你需要的页面引入该组件：

```
{
  "usingComponents": {"poster": "wxa-plugin-canvas/poster"}
}
```
然后就可以在`wsml`中使用了：

```
<poster id="poster" hide-loading="{{false}}" preload="{{false}}" config="{{posterConfig}}" bind:success="onPosterSuccess" bind:fail="onPosterFail"></poster>
```

由于我们在生成海报前，需要异步获取一些用于海报的数据，所以我们采用异步生成的海报方式。

需要引入该组件的`poster/poster.js`文件，然后在代码中调用即可：

```
import Poster from '../../utils/poster';
Page({
    /**
     * 异步生成海报
     */
    onCreatePoster() {
    	// setData配置数据
    	this.setData({ posterConfig: {...} }, () => {
        	Poster.create(); 
    	});
    }
})
```
## 核心代码解析

#### 海报需要的数据

先来看看分享海报的整体结构：

![image](http://image.bug2048.com/1557670306978.jpg)

首先需要确认海报的构成需要哪些数据，在调用组件前先获取好相应的数据。

在我设计的海报中主要包含三块内容，用户的信息(头像和昵称)，文章信息(首图,标题,简介)和最重要的文章的小程序码。

用户信息和文章信息其实比较简单，在小程序的详情页两者数据都有，但这里有两个问题点需要注意下。

第一个是域名问题，在画布中使用到的图片都需要配置域名，头像的域名和公众号文章首图的域名

```
https://mmbiz.qpic.cn
https://wx.qlogo.cn
```

![](https://puui.qpic.cn/vupload/0/1566546579976_tgmxlijrrsl.jpeg/0)

第二个是公众号首图的问题，公众号素材列表返回的图片url其实是`http`的，但小程序规定绑定的域名必须是`https`的，当时比较无奈，后来尝试改用https访问首图的url也可以，不幸中的万幸，所以在使用首图地址时进行替换下：

```
imageUrl = imageUrl.replace('http://', 'https://')
```

最后就是文章的小程序码了，需要利用小程序的`getUnlimited`的api，具体可以参考官方文档，目前已经提供了云调用的方式「无需获取access_token」,调用起来还是比较简单的。

原本打算在文章同步的时候「adminService」直接生成对应文章的小程序码，代码写完后本地调试可以，但上传至云端后测试发现一直报错，逛了轮胎才知道原来不支持，同时触发器也不支持云调用，所以这个计划泡汤了，我在代码中打了TODO。

![image](http://image.bug2048.com/1557671381977.jpg)

既然这样，那就在生成海报的时候进行生成，同时生成后直接上传至云存储，将对应的FileID保存至文章集合中，这样只用生成一次就可以一直使用了，具体代码如下：

```
/**
 * 新增文章二维码
 * @param {} event 
 */
async function addPostQrCode(event)
{
  let scene = 'timestamp=' + event.timestamp;
  let result = await cloud.openapi.wxacode.getUnlimited({
    scene: scene,
    page: 'pages/detail/detail'
  })

  if (result.errCode === 0) {
    let upload = await cloud.uploadFile({
      cloudPath: event.postId + '.png',
      fileContent: result.buffer,
    })

    await db.collection("mini_posts").doc(event.postId).update({
      data: {
        qrCode: upload.fileID
      }
    });

    let fileList = [upload.fileID]
    let resultUrl = await cloud.getTempFileURL({
      fileList,
    })
    return resultUrl.fileList
  }

  return []

}
```

但这里有个尴尬的地方是，生成小程序码的api中的`scene`参数最大长度是32，而文章id的长度已经是32了，无法根据文章id进行拼接跳转页面的路径了，所以这里暂时用了`mini_posts`集合中timestamp字段「理论上也是唯一的」。

所以在详情页中也需要兼容timestamp这个字段。

#### 海报图片展示

海报图片展示就比较简单了，使用个弹窗，将生成好的海报图片进行展示即可：

```
  /**
   * 生成海报成功-回调
   * @param {} e 
   */
  onPosterSuccess(e) {
    const { detail } = e;
    this.setData({
      posterImageUrl: detail,
      isShowPosterModal: true
    })
    console.info(detail)
  },
```

#### 保存海报图片

保存图片使用wx.saveImageToPhotosAlbum调用用户相册，这里主要需要兼容用户拒绝相册授权的一些列操作，具体代码如下：

```
  /**
  * 保存海报图片
  */
  savePosterImage: function () {
    let that = this
    wx.saveImageToPhotosAlbum({
      filePath: that.data.posterImageUrl,
      success(result) {
        console.log(result)
        wx.showModal({
          title: '提示',
          content: '二维码海报已存入手机相册，赶快分享到朋友圈吧',
          showCancel: false,
          success: function (res) {
            that.setData({
              isShowPosterModal: false,
              isShow: false
            })
          }
        })
      },
      fail: function (err) {
        console.log(err);
        if (err.errMsg === "saveImageToPhotosAlbum:fail auth deny") {
          console.log("再次发起授权");
          wx.showModal({
            title: '用户未授权',
            content: '如需保存海报图片到相册，需获取授权.是否在授权管理中选中“保存到相册”?',
            showCancel: true,
            success: function (res) {
              if (res.confirm) {
                console.log('用户点击确定')
                wx.openSetting({
                  success: function success(res) {
                    console.log('打开设置', res.authSetting);
                    wx.openSetting({
                      success(settingdata) {
                        console.log(settingdata)
                        if (settingdata.authSetting['scope.writePhotosAlbum']) {
                          console.log('获取保存到相册权限成功');
                        } else {
                          console.log('获取保存到相册权限失败');
                        }
                      }
                    })

                  }
                });
              }
            }
          })
        }
      }
    });
  },
```

## 体验总结



- 有好的开源组件可以充分利用，避免重复造轮子，有机会也可以学习下别人的实现方式。



- 多看看文档，其实小程序的文档真的挺详细的。




- 这里主要想分享实现一个功能实现的过程，有想法的时候如何一步步去成功实现。



- 小程序本身不难，相应的文档也很详细，但是组装的过程和逻辑的实现需要自身去思考和体会。多看看文档，其实小程序的文档真的挺详细的。



- 如果你的想法和流程都非常清晰，但还是没办法实现你的预期功能，那我建议你先放放，先把`html`,`css`,`javascript`熟悉下，再看几遍小程序的文档，也许你当时面临的问题就不再是问题了。

# 源码链接
[https://github.com/CavinCao/mini-blog](https://github.com/CavinCao/mini-blog)
