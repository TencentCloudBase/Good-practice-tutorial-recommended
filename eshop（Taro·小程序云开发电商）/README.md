# eshop
## Taro·小程序云开发电商教程示例
本 demo 适用于 Taro·小程序云开发
## 关于 Taro
[Taro](https://taro.jd.com/) 是由凹凸实验室打造的一套遵循 React 语法规范的多端统一开发解决方案。使用 Taro，我们可以只书写一套代码，再通
过 Taro 的编译工具，将源代码分别编译出可以在不同端（字节跳动小程序、微信小程序、百度智能小程序、支付宝小程序、QQ
小程序、跨应用、H5、App 端等）运行的代码。同时 Taro 还提供开箱即用的语法检测和自动补全等功能，有效地提升了开发体验和开发效率。</br>
[Taro](https://taro.jd.com/) 作为 **凹凸实验室** 的主要开源产品之一，在 github 上拥有 19,000+ star， Taro 在业界获得广泛的关注。
## 关于云开发
文章中的几个数据集的模拟数据，已上传至 `cloud/doc` 中，可根据需要自行导入。不过购物车，订单类的数据与 `openId` 相关联，所以更多的是可以参考其数据结构。
云函数的代码存放在 `cloud/functions` 中，代码是按模块进行分割，每个模块有一个入口文件和一些执行具体逻辑的文件。
## 关于云函数
在使用你自己的云函数环境时，需要将云函数初始化换成你小程序的云环境。有以下几个地方需要注意：
- `src/app.js`
- `cloud/functions` 下每个云函数里的 `index.js`
- `project.config.json` 里的appid需要换成你的小程序的 `appid`
```javascript
// src/app.js
async componentDidMount () {
  Taro.cloud.init({
    env: 'xxyyzz', // 换成你的云函数环境
    traceUser: true // 是否要捕捉每个用户的访问记录。设置为true，用户可在管理端看到用户访问记录
  })
  const userData = await getWxUserData()
  setGlobalData('userData', userData)
}
```
```javascript
// 例： cloud/functions/cart/index.js
...
app.init({
  envName: 'wushufang-h36jx', // 换成你的云函数环境
  mpAppId: 'wx22203329a468e8a1' // 换成你的小程序id
})
...
```
## 如果有问题怎么办？
> 您可以到 Taro 社区 [小程序·云开发](https://taro-club.jd.com/category/14/%E5%B0%8F%E7%A8%8B%E5%BA%8F-%E4%BA%91%E5%BC%80%E5%8F%91) 版块来讨论咯

# 源码链接
[https://github.com/honlyHuang/eshop](https://github.com/honlyHuang/eshop)
