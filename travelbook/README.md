# IDE
- [微信开发者工具](https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html?t=201822)
- [VSCode](https://code.visualstudio.com/Download)

小程序开发必然少不了微信开发者工具，再加上其对云开发的全面支持，再好不过的开发利器。但熟悉微信开发者工具的朋友们应该知道，它不支持[Emmet缩写语法](https://www.cnblogs.com/cnjava/p/3225174.html)，并且wxml的属性值默认用单引号表示(强迫症表示很难受)。
而VSCode很好的补足了微信开发者工具的不足之处，并且支持多元化[插件开发](https://juejin.im/post/5a08d1d6f265da430f31950e)，轻量好用。

所以这里推荐采用微信开发者工具+VSCode配合开发。微信开发者工具负责调试、模拟小程序运行情况，VSCode负责代码编辑工作。二者各司其职，会使开发更加的高效、便捷
# 总体架构
该项目基于小程序云开发，使用的模板是[云开发快速启动模板](https://cloud.tencent.com/developer/article/1345310)

由于是个全栈项目，前端使用小程序所支持的wxml + wxss + js开发模式，命名采用[BEM](https://juejin.im/post/5bb4678a5188255c980be9d2)命名规范。后台则是借助云数据库+云储存进行数据管理。
#### 项目总体结构
```javascript
|-travelbook  项目名
    |-cloudfunctions  云函数模块
        |-deleteItems 级联删除--云函数
        |-getTime     获取时间--云函数
    |-miniprogram  项目模块
        |-components  自定义组件
            |-accountCover  账本封面组件
            |-spendDetail   支出细节组件
        |-pages  页面
            |-accountBooks     总账本页
            |-accountCalendar  账本日历页
            |-accountDetail    支出细节页
            |-accountList      支出明细页
            |-accountPage      选定账本页
            |-editAccount      账本编辑页
            |-index            首页
        |-vant-weapp   有赞vant框架组件库
            |-···      系列组件...
        app.js         全局js
        app.json       全局json配置
        app.wxss       全局wxss
```        
# 逆向工程
在做该小程序之前，有必要进行项目的逆向工程，进一步解构每一个页面，从而深入了解这款小程序的交互细节。那么现在我假设自己为腾讯旅游的产品设计师，在绘制完界面原型后，撰写了相应的交互文档。当然解构过程中可能有些细节处理并没有那么仔细到位...

以下是我绘制的界面原型
![](https://puui.qpic.cn/vupload/0/20190618_1560841685577_ez53lrmq4ji.jpeg/0)
![](https://puui.qpic.cn/vupload/0/20190618_1560841745416_ilo7zykytkc.jpeg/0)
![](https://puui.qpic.cn/vupload/0/20190618_1560841810370_zxgzdholi6j.png/0)
接下来对每个页面的细节进行解构，并完成简单的wxml结构
![](https://puui.qpic.cn/vupload/0/20190618_1560841863494_r2jb1rjjxz.png/0)
```javascript
<!--switchList使用定位布局-->
<view bindtap="switchList" class="list"></view>

<!--newAccount使用flex布局-->
<view class="newAccount" bindtap="createNewAccount">
    <view class="desc">旅行中的每一笔开支都有独特的意义！</view>
    <image src="{{}}"></image>
    <view class="title">创建一个新账本</view>
</view>
```
![](https://puui.qpic.cn/vupload/0/20190618_1560841983381_9t2npq4e0lc.jpeg/0)
```javascript
<!--整体用flex + 百分比布局-->
<input type="text" class="accuntName" placeholder="旅行账本名称" bindinput="getInput" />
  
<van-panel title="选择封面" class="panel">
    <van-row class="imageBox">
        <!--使用wx:for遍历数据库账本图片信息-->
        <van-col span="8" class="imgCol" bindtap="selectThis">
            <image class="select" src="{{}}"></image>
        </van-col>
        
        <van-col span="8">
            <view class="addBox" bindtap="useMore">更多封面</view>
        </van-col>
    </van-row>
</van-panel>

<button type="primary" bindtap="save">保存</button>
<button type="warn" bindtap="delete">删除</button>
```
![](https://puui.qpic.cn/vupload/0/20190618_1560842059815_nfusyuuau7.png/0)
```javascript
<view class="accountDesc" bindtap="viewDetail">
    <!--使用wx:for遍历数据库账本信息-->
    <view class="accountName">
        <view>{{}}</view>
        <view class="accountTime">{{}}</view>
    </view>
    
    <!--绝对定位-->
    <image class="updateImg" catchtap="editAccount" src="{{}}"></image>
</view>
```
![](https://puui.qpic.cn/vupload/0/20190618_1560842184322_kfrsl4rpv6p.png/0)
```javascript
<!--switchList使用定位布局-->
<view bindtap="switchList" class="list"></view>

<view class="account__list-year">{{}}</view>
<view class="account__list-new account__list-public" bindtap="createNewAccount">
    <!--日期小圆点-->
    <view class="account__list-point"></view>
    <view class="account__list-time">{{}}</view>
    <image src="{{}}"></image>
    <view class="account__list-title">创建一个新账本</view>
</view>

<!--使用wx:for遍历数据库账本信息-->
<view class="account__list-item account__list-public" bindtap="viewDetail">
    <!--日期小圆点-->
    <view class="account__list-point"></view>
    <image src="{{}}" mode="aspectFill"></image>
    <view class="account__list-name">{{}}</view>
    <view class="account__list-time">{{}}</view>
    <image class="account__list-update" catchtap="editAccount" src="{{}}"></image>
 </view>
 ```
 ![](https://puui.qpic.cn/vupload/0/20190618_1560842250801_o1qdpeigrhr.jpeg/0)
 ```javascript
 <view class="account__spend">
    <image bindtap="getCalendar" class="account__spend-calendar" src="{{}}"></image>
    <view class="account__spend-text">
        <view class="account__spend-total">总花费(元)</view>
        <view class="account__spend-num">{{}}</view>
    </view>
    <image bindtap="accountAnalyze" class="account__spend-detail" src="{{}}"></image>
</view>

<view class="account__show-time">今天</view>
    <view class="account__show-detail">
        <view class="account__show-income account__show-public">
        <view class="account__show-title">收入(元)</view>
        <text class="account__show-in">+{{}}</text>
    </view>
    <view class="account__show-spend account__show-public">
        <view class="account__show-title">支出(元)</view>
        <text class="account__show-out">-{{}}</text>
    </view>
</view>

<!--使用wx:for遍历数据库账本信息-->
<view class="account__show-items-spend">
    <view>
        <image src="{{}}"></image>
    </view>
    <text>{{}}</text>
    <text class="account__show-items-money">{{}}</text>
</view>
```
![](https://puui.qpic.cn/vupload/0/20190618_1560842373409_km8tjmyscql.png/0)
```javascript
<!--日历使用极点日历的插件-->
<!--json中做配置-->
"usingComponents": {
    "calendar": "plugin://calendar/calendar"
}

<!--js改变样式-->
days_style.push({
  month: 'current',
  day: new Date().getDate(),
  color: 'white',
  background: '#e0a58e'
})

<!--wxml中引用-->
<calendar weeks-type="cn" cell-size="50" next="{{true}}" prev="{{true}}"
    show-more-days="{{true}}" calendar-style="demo6-calendar"
    header-style="calendar-header"board-style="calendar-board" active-type="rounded" 
    lunar="true" header-style="header"calendar-style="calendar"days-color="{{days_style}}">
</calendar>
```
![](https://puui.qpic.cn/vupload/0/20190618_1560842442806_i03eiats4d.png/0)
```javascript
<!--顶栏日期及收支结构-->
<view class="account__title">
    <text class="account__title-time">{{}}</text>
    <text class="account__title-spend">支出{{}}元 收入{{}}元</text>
</view>

<!--收支细节结构 使用flex弹性布局-->
<view class="account__detail">
    <image src="{{}}"></image>
    <view class="account__detail-name">{{}}</view>
    <view class="account__detail-money">{{}}</view>
</view>
```
![](https://puui.qpic.cn/vupload/0/20190618_1560842599626_eqbucxicxre.png/0)
```javascript
<!--使用vant框架的van-tabs组件-->
<!--并封装自定义组件复用收支页，自定义组件后面会详细说明-->
<van-tabs active="{{ active }}" bind:change="onChange">
  <van-tab title="支出">
    <spendDetail detail="{{detail}}" accountKey="{{accountKey}}"></spendDetail>
  </van-tab>
  <van-tab title="收入">
    <spendDetail detail="{{income}}" accountKey="{{accountKey}}"></spendDetail>
  </van-tab>
</van-tabs>
```
# 云开发
在做完逆向工程的解构，页面基础结构基本搭建完成。但页面依旧是静态的，需要数据来填充。所以第二步就是数据库的设计。而小程序的云控制台恰好提供了数据的操作功能，为数据驱动提供基石。
![](https://puui.qpic.cn/vupload/0/20190618_1560842693321_2iryp54j1mr.png/0)
## 云数据库设计
云数据库是一种NoSQL数据库。每一张表是一个集合。值得注意的是在设计数据库时，`_id` 和`_openid`这两个字段需要带上。`_id`是表的主键，而`_openid`是用户标识，每个用户都有不同的`_openid`，可区分不同用户。

以下是项目中的数据表设计
```javascript
cover_photos 账本封面表  用于存储创建账本时需要的封面信息
    - _id
    - _openid
    - cover_index 封面索引
    - cover_url   封面url
    - isSelected  封面是否选中
```
```javascript
accounts 账本表   用于存储用户创建的账本
    - _id
    - _openid
    - accountKey  账本唯一标识
    - coverUrl    账本封面
    - i           账本索引
    - inputValue  账本名字
    - now         账本创建时间
    - spend       账本总花费
```
```javascript
account_detail 支出类型表   用于存储消费类型
    - _id
    - _openid
    - detail       类型细节
    - pic_index    消费类型索引
    - pic_url      未点击时的图片
    - pic_url_act  点击后的图片
    - type         消费类型
```
```javascript
account_income 收入类型表   用于存储收入类型
    - _id
    - _openid
    - pic_index    收入类型索引
    - pic_url      未点击时的图片
    - pic_url_act  点击后的图片
    - type         收入类型
```
```javascript
spend_items   消费明细表
    - _id
    - _openid
    - accountKey   账本唯一标识
    - address      消费地点
    - desc         消费描述
    - fullDate     消费时间
    - money        消费金额
    - pic_type     消费类型
    - pic_url      消费类型图片
```    
## 云储存管理
这是个非常实用的板块。类似于[百度云盘](https://pan.baidu.com/)，它提供了文件存储、上传与下载功能。
![](https://puui.qpic.cn/vupload/0/20190618_1560843125594_leagaa09yfi.png/0)
除此之外，它还会将你所上传的资源自动进行压缩操作，并生成一个地址供你引用。该项目中的一些图片资源就是存在于此，然后在云数据库的字段中引用这些资源地址即可，十分方便，不必在本地存储，占用小程序内存。
![](https://puui.qpic.cn/vupload/0/20190618_1560843165699_eczla5i3ixl.png/0)
## 云函数设计
[云函数](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/guide/functions.html)简单来说就是在云后端(Node.js)运行的代码，本地看不到这些代码的执行过程，全封闭式只暴露接口供本地调用执行，本地只需等待云端代码执行完毕后返回结果。这也是[面向接口编程](https://www.cnblogs.com/bobodeboke/p/5733422.html)的思想体现。

项目中的云函数设计
