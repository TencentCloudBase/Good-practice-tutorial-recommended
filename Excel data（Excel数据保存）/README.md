# 小程序导出数据到excel表，借助云开发后台实现excel数据的保存

>我们在开发小程序的过程中，可能会有这样的需求：如何将云数据库里的数据批量导出到excel表里？
>这个需求可以用强大的云开发轻松实现！
>这里需要用到云函数，云存储和云数据库。可以说通过这一个例子，把小程序云开发相关的知识都用到了。下面就来介绍如何实现

# 实现思路
- 1，创建云函数
- 2，在云函数里读取云数据库里的数据
- 3，安装node-xlsx类库（node类库）
- 4，把云数据库里读取到的数据存到excel里
- 5，把excel存到云存储里并返回对应的云文件地址
- 6，通过云文件地址下载excel文件

# 一、创建excel云函数
关于如何创建云开发小程序，这里我就不再做具体讲解。不知道怎么创建云开发小程序的同学，可以去翻看腾讯云云开发公众号内菜单【技术交流-视频教程】中的教学视频。

#### 创建云函数时有两点需要注意的，给大家说下
- 1、一定要把app.js里的环境id换成你自己的
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS85LzcvMTZkMGFlYzQ0Njg2NjJlZQ?x-oss-process=image/format,png#pic_center)
- 2，你的云函数目录要选择你对应的云开发环境（通常这里默认选中的）
不过你这里的云开发环境要和你app.js里的保持一致
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS85LzcvMTZkMGFlY2U2OWY1NjY4OA?x-oss-process=image/format,png#pic_center)

# 二、读取云数据库里的数据
我们第一步创建好云函数以后，可以先在云函数里读取我们的云数据库里的数据。
- 1、先看下我们云数据库里的数据
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS85LzcvMTZkMGFmMTY0ZjQxNWRkZg?x-oss-process=image/format,png#pic_center)
- 2、编写云函数，读取云数据库里的数据（一定要记得部署云函数）
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS85LzcvMTZkMGIyN2EzYzU2ZmFiNA?x-oss-process=image/format,png#pic_center)
- 3、成功读取到数据
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS85LzcvMTZkMGFmYjg3MTIxM2VlYw?x-oss-process=image/format,png#pic_center)

把读取user数据表的完整代码给大家贴出来。
```
// 云函数入口文件
const cloud = require('wx-server-sdk')
cloud.init({
  env: "test-vsbkm"
})
// 云函数入口函数
exports.main = async(event, context) => {
  return await cloud.database().collection('users').get();
}
```

# 三、安装生成excel文件的类库 node-xlsx
通过上面第二步可以看到我们已经成功的拿到需要保存到excel的源数据，我们接下来要做的就是把数据保存到excel
- 1、安装node-xlsx类库
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS85LzcvMTZkMGFmZWZhYTBhNDA2YQ?x-oss-process=image/format,png#pic_center)
这一步需要我们事先安装node,因为我们要用到npm命令，通过命令行
```
npm install node-xlsx
```
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS85LzcvMTZkMGIwMDgyMjE0YWE1OQ?x-oss-process=image/format,png#pic_center)

可以看出我们安装完成以后，多了一个package-lock.json的文件
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS85LzcvMTZkMGIwMTI0MWQ2YmU4ZQ?x-oss-process=image/format,png#pic_center)


# 四、编写把数据保存到excel的代码，
下图是我们的核心代码：
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS85LzcvMTZkMGIyNmNhOTJiN2YwNQ?x-oss-process=image/format,png#pic_center)
这里的数据是我们查询的users表的数据，然后通过下面代码遍历数组，然后存入excel。这里需要注意我们的id,name,weixin要和users表里的对应。
```
   for (let key in userdata) {
      let arr = [];
      arr.push(userdata[key].id);
      arr.push(userdata[key].name);
      arr.push(userdata[key].weixin);
      alldata.push(arr)
    }
```
还有下面这段代码，是把excel保存到云存储用的
```
    //4，把excel文件保存到云存储里
    return await cloud.uploadFile({
      cloudPath: dataCVS,
      fileContent: buffer, //excel二进制文件
    })
```
下面把完整的excel里的index.js代码贴给大家,记得把云开发环境id换成你自己的。
```
const cloud = require('wx-server-sdk')
//这里最好也初始化一下你的云开发环境
cloud.init({
  env: "test-vsbkm"
})
//操作excel用的类库
const xlsx = require('node-xlsx');

// 云函数入口函数
exports.main = async(event, context) => {
  try {
    let {userdata} = event
    
    //1,定义excel表格名
    let dataCVS = 'test.xlsx'
    //2，定义存储数据的
    let alldata = [];
    let row = ['id', '姓名', '微信号']; //表属性
    alldata.push(row);

    for (let key in userdata) {
      let arr = [];
      arr.push(userdata[key].id);
      arr.push(userdata[key].name);
      arr.push(userdata[key].weixin);
      alldata.push(arr)
    }
    //3，把数据保存到excel里
    var buffer = await xlsx.build([{
      name: "mySheetName",
      data: alldata
    }]);
    //4，把excel文件保存到云存储里
    return await cloud.uploadFile({
      cloudPath: dataCVS,
      fileContent: buffer, //excel二进制文件
    })

  } catch (e) {
    console.error(e)
    return e
  }
}

```

# 五、把excel存到云存储里并返回对应的云文件地址
经过上面的步骤，我们已经成功的把数据存到excel里，并把excel文件存到云存储里。可以看下效果。
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS85LzcvMTZkMGIyZTA0MWQ0YzdjYg?x-oss-process=image/format,png#pic_center)
接着，就可以通过上图的下载地址下载excel文件了。
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS85LzcvMTZkMGIyZjNlOWFkODU1Nw?x-oss-process=image/format,png#pic_center)
其实到这里就差不多实现了基本的把数据保存到excel里的功能了，但是为了避免每次导出数据都需要去云开发后台下载excel的麻烦，接下来介绍如何动态获取下载地址。
# 六、获取云文件地址下载excel文件
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS85LzcvMTZkMGIzNjEzMWE3ZTg5MQ?x-oss-process=image/format,png#pic_center)
通过上图我们可以看出，我们获取下载链接需要用到一个fileID,而这个fileID在我们保存excel到云存储时，有返回，如下图。我们把fileID传给我们获取下载链接的方法即可。
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS85LzcvMTZkMGIzNzZmZTBmZTZkNw?x-oss-process=image/format,png#pic_center)
- 1、我们获取到了下载链接，接下来就要把下载链接显示到页面
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS85LzcvMTZkMGIzYWRhYjIyM2U4Nw?x-oss-process=image/format,png#pic_center)
- 2、代码显示到页面以后，我们就要复制这个链接，方便用户粘贴到浏览器或者微信去下载。
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS85LzcvMTZkMGI0MDhjMTIzMTJjNA?x-oss-process=image/format,png#pic_center)

下面是完整代码：
```
Page({
  onLoad: function(options) {
    let that = this;
    //读取users表数据
    wx.cloud.callFunction({
      name: "getUsers",
      success(res) {
        console.log("读取成功", res.result.data)
        that.savaExcel(res.result.data)
      },
      fail(res) {
        console.log("读取失败", res)
      }
    })
  },

  //把数据保存到excel里，并把excel保存到云存储
  savaExcel(userdata) {
    let that = this
    wx.cloud.callFunction({
      name: "excel",
      data: {
        userdata: userdata
      },
      success(res) {
        console.log("保存成功", res)
        that.getFileUrl(res.result.fileID)
      },
      fail(res) {
        console.log("保存失败", res)
      }
    })
  },

  //获取云存储文件下载地址，这个地址有效期一天
  getFileUrl(fileID) {
    let that = this;
    wx.cloud.getTempFileURL({
      fileList: [fileID],
      success: res => {
        // get temp file URL
        console.log("文件下载链接", res.fileList[0].tempFileURL)
        that.setData({
          fileUrl: res.fileList[0].tempFileURL
        })
      },
      fail: err => {
        // handle error
      }
    })
  },
  //复制excel文件下载链接
  copyFileUrl() {
    let that=this
    wx.setClipboardData({
      data: that.data.fileUrl,
      success(res) {
        wx.getClipboardData({
          success(res) {
            console.log("复制成功",res.data) // data
          }
        })
      }
    })
  }
})
```

梳理下上面代码的逻辑：

- 1、先通过getUsers云函数去云数据库获取数据。
- 2、把获取到的数据通过excel云函数把数据保存到excel，然后把excel保存的云存储。
- 3、获取云存储里的文件下载链接。
- 4、复制下载链接，到浏览器里下载excel文件。

到这里我们就完整的实现了把数据保存到excel的功能了。

文章有点长，知识点有点多，但是大家理解上述内容后，就可以对小程序云开发的云函数、云数据库、云存储有一个较为完整的了解过程。

---

如果你想要了解更多关于云开发CloudBase相关的技术故事/技术实战经验，请扫码关注【腾讯云云开发】公众号～![在这里插入图片描述](https://img-blog.csdnimg.cn/20190910094810571.png#pic_center)
