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
