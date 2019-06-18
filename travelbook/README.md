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
