### 一、导语

如何省去企业上门（现场）搜集客户需求的环节，节约企业人力和时间成本，将客户的业务定制需求直接上传至云数据库？云开发为我们提供了这个便利！  

### 二、需求背景

作为一名XX公司IT萌萌新，这段时间对小程序开发一直有非常浓厚的兴趣，并且感慨于“云开发·不止于快”的境界。近期工作中，刚好碰见业务部门的一个需求，目的是节约上门跟客户收集业务定制资料的时间，以往是每变更一次，就需要上门一次，碰见地域较远的，费时费力，且往往要求几天内完成上线，时间非常紧迫。因此，结合一直以来对云开发的各种优势的了解，我说服公司领导通过小程序·云开发来实现。


下面是其中一项业务定制界面的展示：
（1）业务对业务流程有简单说明；
（2）相关业务介绍；
（3）不同客户输入个性化需求；
（4）云存储后台实现需求表单的收集。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191121104640541.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RDQl9DbG91ZEJhc2U=,size_16,color_FFFFFF,t_70#pic_center)
         得力于云开发提供的API和WeUI库的便利，本项目在我在极短的时间内就实现了比较理想的效果 。接下来，我就从本项目入手，讲讲我是如何依靠小程序·云开发将想法快速实现的，其实我也是刚入门没多久，只是想分享一下自身在学习小程序开发项目中的一些知识点和体会，代码可能略为粗糙，逻辑也有待优化，欢迎大家在评论区多多交流。

### 三、开发过程

#### 1、组件

主要使用了官方WeUI扩展能力的一些组件库来实现主要功能。

核心的WeUI库主要有 Msg、Picker、图片的Upload等（以快为目的，节省自己写CSS样式的时间，也方便0基础的同学上手，这里又体会到了小程序开发的便捷）。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191121104845916.png#pic_center)

#### 2、实现代码

本次云开发包括`云数据库`、`云存储`两大功能：

##### （1）**云数据库**

**云数据库**的主要就是搜集客户提交上来的表单信息，包括客户的联系方式和选择的业务类型等，并存储在云数据库中，方便业务经理搜集需求。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191121105001801.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RDQl9DbG91ZEJhc2U=,size_16,color_FFFFFF,t_70#pic_center)
我们来看简单的实现过程：

首先是表单，用到了 form 表单组件以及它的 bindsubmit 方法，在 wxml 中放置 form 表单：

```js
<form bindsubmit="formSubmit">
    <view class="form">
      <view class="section">
        <picker bindchange="bindPickerGsd" mode="selector" value="{{indexGsd}}" range="{{arrayGsd}}">
          <view class="picker">归属县市</view>
          <view class="picker-content" >{{arrayGsd[indexGsd]?arrayGsd[indexGsd]:"(必填项) 请下拉选择归属地"}}</view> 
        </picker>
      </view>    
      <!---中间部分详见代码--->
    </view>

    <view class="footer">
      <button class="dz-btn" formType="submit" loading="{{formStatus.submitting}}" disabled="{{formStatus.submitting}}" bindtap="openSuccess">提交</button>
    </view>
  </form>
```

表单中除了普通的文本输入，增加有下拉列表的实现（毕竟客户有时候是比较懒的）。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019112110511124.png#pic_center)
来看一下具体代码：

```js
bindPickerGsd: function (e) {    
  console.log('归属地已选择，携带值为', e.detail.value)
  console.log('归属地选择:', this.data.arrayGsd[e.detail.value])    
  this.setData({
   	 indexGsd: e.detail.value     
   })   
   this.data.formData.home_county = this.data.arrayGsd[e.detail.value]
},
```

最后表单上传到云数据库：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191121105219113.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RDQl9DbG91ZEJhc2U=,size_16,color_FFFFFF,t_70#pic_center)

```js
  // 表单提交
  formSubmit: function (e) {
    var minlength = '';
    var maxlength = '';
    console.log("表单内容",e)
    var that = this;
    var formData = e.detail.value;
    var result = this.wxValidate.formCheckAll(formData);
    
    console.log("表单提交formData", formData);
    console.log("表单提交result", result)
    wx.showLoading({
      title: '发布中...',
    })
    const db = wx.cloud.database()
    db.collection('groupdata').add({
      data: {
        time: getApp().getNowFormatDate(),
        home_county: this.data.formData.home_county,
        group_name: formData.group_name,
        contact_name: formData.contact_name,
        msisdn: formData.msisdn,
        product_name: this.data.formData.product_name,
        word: formData.word,
      },
      success: res => {
        wx.hideLoading()
        console.log('发布成功', res)

      },
      fail: err => {
        wx.hideLoading()
        wx.showToast({
          icon: 'none',
          title: '网络不给力....'
        })
        console.error('发布失败', err)
      }
    })
  },
```

##### （2）云存储

因为业务的定制需要填单客户所在单位的授权证明材料，因此需要提单人（使用人）上传证明文件，因此增加了使用云存储的功能。

核心代码： 

```js
    promiseArr.push(new Promise((reslove,reject)=>{
    	wx.cloud.uploadFile({
    		cloudPath: "groupdata/" + group_name + "/" + app.getNowFormatDate() +suffix,
    		filePath:filePath
    	}).then(res=>{
    		console.log("授权文件上传成功")          
    		})
    		reslove()
    		}).catch(err=>{
    		console.log("授权文件上传失败",err)
    })

 	因为涉及到不同页面的数据传递，即将表单页面的group_name作为云存储的文件夹用于存储该客户在表单中上传的图片，因此还需要用到getCurrentPages()来进行页面间的数据传递 

    var pages = getCurrentPages();
    var prePage = pages[pages.length - 2];//pages.length就是该集合长度 -2就是上一个活动的页面，也即是跳过来的页面
    var group_name = prePage.data.formData.group_name.value//取上页data里的group_name数据用于标识授权文件所存储文件夹的名称
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191121105353566.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RDQl9DbG91ZEJhc2U=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191121105423686.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RDQl9DbG91ZEJhc2U=,size_16,color_FFFFFF,t_70#pic_center)

##### 3、待进一步优化 

基于时间关系，本次版本仅对需求进行了简单实现，作为公司一个可靠的项目，还需要关注"客户隐私”、“数据安全”，以及更人性化的服务。比如：

**（1）提单人确认和认证过程**

可靠性：增加验证码验证（防止他人冒名登记），以及公司受理业务有个客户本人提交凭证。

**（2）订阅消息**  

受理成功后，可以给客户进行处理结果的反馈，增强感知。

**（3）人工客服**  

进行在线咨询等。

### 四、总结

在本次项目开发中，我深刻体会到了云开发的“快”，特别是云数据库的增删查改功能非常方便。云开发提供的种种便利，让我在有新创意的时候，可以迅速采用小程序云开发快速实现，省时省力，还能免费使用腾讯云服务器，推荐大家尝试！

## 源码地址

<https://github.com/fengwolf3/GroupData>

