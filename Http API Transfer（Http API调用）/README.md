# 小程序云开发之httpApi调用。

小程序云开发之httpApi调用（返回“47001处理”）

## 技术栈

    采用 nodejs + express 搭建web服务器，采用 axios 请求第三方 httpApi
   
   [nodejs](http://nodejs.cn/)  
   [express](http://www.expressjs.com.cn/)  
   [axios](https://www.npmjs.com/package/axios)  

## 项目结构

通过应用生成器工具 **express-generator** 可以快速创建一个应用的骨架。

主要的核心文件 **routes/base.js(api设置)，util/rq.js(axios封装),views/base.pug(接口文档)**

```javascript
  
|---bin (框架生成，服务启动命令文件夹)
|---public (框架生成，静态资源存储)
|-------images
|-------javascripts
|-------stylesheets
|---routes (框架生成，路由配置/api)
|-------base.js // base相关接口及文档说明页
|---util (自行添加文件夹,放置公用js)
|-------result.js // 最终返回结果包装js
|-------rq.js // axios封装
|---views (框架生成，页面存放)
|-------error.pug
|-------base.pug (自行添加pug模板页面，用于base接口说明)
|-------index.pug
|-------layout.pug
|---app.js (框架生成，项目核心)

```

- **axios封装（util/rq.js）**


```javascript

// 模块引用
let axios = require("axios")
let qs = require("qs")

// 变量声明
const CONFKEY = "dev"
const BASECONF = {
    "dev":{
        baseUrl:'https://api.weixin.qq.com/',
    },
    "prod":{
        baseUrl:'https://api.weixin.qq.com/'
    }
}[CONFKEY]

// 创建rq请求并设置基础信息
const rq = axios.create({
    baseURL: BASECONF.baseUrl,
    timeout: 10000,
    headers: { // 请求头设置，（微信云开发数据APi采用application/json格式入参，否则导致47001错误）
        "Content-Type":"application/json; charset=utf-8"
    }
})

// axios 请求头拦截器
rq.interceptors.request.use(req => {
    // 有需要的，在此处拦截请求入参进行处理
    return req
},error => {
    return Promise.reject(error)
})

// axios 返回信息拦截器
rq.interceptors.response.use(res => {
    return res.data
},error => {
    return Promise.reject(error)
})

const $rq = { // 封装get,post请求

    get(url,params) { // axios.get(url,config)
        return rq.get(url,{
            params: params
        })
    },
    post(url,params={}) {
        return rq({ // axios(config)
            url: url,
            method: 'post',
            data:params
        })
    }
}

module.exports = {
    $rq
}

```

- **api设置 （routes/base.js）**

```javascript

var express = require('express');
var router = express.Router();
var { $rq } = require("../util/rq")
let result = require("../util/result.js")

/* GET base page. */
router.get('/', function(req, res, next) { // base pugApi说明文档
    res.render('base', { title: 'baseApi', 
        apiList:[
            {
                url:"base/getAccessToken(请求第三方Api，获取access_token)",
                method:"GET",
                params:{
                    key:"grant_type",
                    appid:"小程序appid",
                    secret: "小程序密钥"
                },
                result:{
                    "success": true,
                    "data":`{
                        "access_token":"23_w0OtD1X72LIQo4dwctVsp99kjtIRRk9Gw5bx7UOglotfL7k9LqB1gKbZw86CNht6cnCv9oKBcFEcPg5u4seXN0hJMSEocsbun2dQxCTyZarP06YcToVbdP-MOLc7o7EhMSzqR4URT__BdZc-NMLbAIARQP",
                        "expires_in":7200
                    }`
                }
            },
            {
                url:"base/getdatabase(获取指定云环境集合信息)",
                method:"post",
                params:{
                    env:"云开发数据库环境id",
                    limit:"获取数量限制,默认10",
                    offset:"偏移量,默认0"
                },
                result:{
                    "success": true,
                    "data":`{
                        {
                        "errcode": 0,
                        "errmsg": "ok",
                        "collections": [
                            {
                                "name": "geo",
                                "count": 13,
                                "size": 2469,
                                "index_count": 1,
                                "index_size": 36864
                            },
                            {
                                "name": "test_collection",
                                "count": 1,
                                "size": 67,
                                "index_count": 1,
                                "index_size": 16384
                            }
                        ],
                        "pager": {
                            "Offset": 0,
                            "Limit": 10,
                            "Total": 2
                        }
                      }
                    }`
                }
            }
        ]
    });
});

router.get('/getAccessToken', function(req, res, next) { // 请求第三方Api，获取access_token
    let urlParam = { // appID，secret信息最好是不暴露在外故在此处直接写死即可
        grant_type:"client_credential",
        appid: "appid",
        secret: "secret"
    };
    $rq.get("cgi-bin/token",urlParam).then(response=>{
        global.TOKEN_INFO = response // global nodejs 全局对象，占用内存
        let r =  result.createResult(true, response); // 返回结果包装成固定格式
        res.json(r);
    }).catch(err=>{
        let r =  result.createResult(false, err);
        res.json(r);
        console.log(err)
    })
});

router.get('/getdatabase', function(req, res, next) { // 获取指定云环境集合信息
    let urlParam = { // 获取access_token之后才能调用其他接口，其他接口的入参就无需传入access_token因为皆须要拼接在接口后
        // access_token: req.query.access_token?req.query.access_token:"",
        env: req.query.env?req.query.env:"test-3b6a08",
        limit: req.query.limit?req.query.limit:10,
        offset: req.query.offset?req.query.offset:0
    };
    $rq.post("tcb/databasecollectionget?access_token="+global.TOKEN_INFO.access_token,urlParam).then(response=>{
        let r =  result.createResult(true, response);
        res.json(r);
    }).catch(err=>{
        let r =  result.createResult(false, err);
        res.json(r);
        // console.log(err)
    })
});

module.exports = router;

```

- **配置app.js 使路由及接口生效（仅）**

```javascript

var createError = require('http-errors'); // 处理错误
var express = require('express');
var path = require('path'); // 路径
var cookieParser = require('cookie-parser'); // cookie
var logger = require('morgan'); // 日志
var sassMiddleware = require('node-sass-middleware'); // sass 中间件

var indexRouter = require('./routes/index'); // index 路由
var baseRouter = require('./routes/base') // base 路由

var app = express();

// view engine setup
app.set('views', path.join(__dirname, 'views')); // 设置视图根目录
app.set('view engine', 'pug'); // 使用 pug 模板

// 声明使用中间件
app.use(logger('dev'));
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(sassMiddleware({
  src: path.join(__dirname, 'public'),
  dest: path.join(__dirname, 'public'),
  indentedSyntax: true, // true = .sass and false = .scss
  sourceMap: true
}));
app.use(express.static(path.join(__dirname, 'public')));

app.all('/*',function (req, res, next) { // 解决跨越问题
  res.header('Access-Control-Allow-Origin', '*');
  res.header('Access-Control-Allow-Headers', 'Content-Type, Content-Length, Authorization, Accept, X-Requested-With');
  res.header('Access-Control-Allow-Methods', 'PUT, POST, GET, DELETE, OPTIONS');
  if (req.method == 'OPTIONS') {
    res.sendStatus(200);
  }
  else {
    next();
  }
});

// 声明路由
app.use('/', indexRouter);
app.use('/base', baseRouter);

// catch 404 and forward to error handler 自定义404中间件
app.use(function(req, res, next) {
  next(createError(404));
});

// error handler 自定义错误抛出中间件
app.use(function(err, req, res, next) { 
  // set locals, only providing error in development
  res.locals.message = err.message;
  res.locals.error = req.app.get('env') === 'development' ? err : {};

  // render the error page
  res.status(err.status || 500);
  res.render('error');
});

module.exports = app;

```

---

至此，小程序云开发----httpApi调用已完成。  

简单的利用**vue+elementui做个云开发小程序后台管理页面调用下上面的接口**。  

我们看下效果如下：


## 云开发小程序后台管理环境调整：

   ![后台管理环境调整](http://m.qpic.cn/psb?/V12a5mB10IdFhp/yBfumzffZjvYU6cwtEzBSXMqaNM4aX07dzAmP6bpqmo!/b/dLgAAAAAAAAA&bo=6AFoAQAAAAADF7I!&rf=viewer_4&t=5)

## 本地启动上面的接口服务及调用结果：
    
   ### 本地启动接口服务
   
   ![本地启动接口服务](http://m.qpic.cn/psb?/V12a5mB10IdFhp/LOqYwnfdrtthoj6QpK0*GT5dlcCBmR2mOSfB5Q0HzLs!/b/dDYBAAAAAAAA&bo=pwK3AwAAAAADFyM!&rf=viewer_4&t=5)
   
   ### 本地接口调用结果
   
   ![本地接口调用结果](http://m.qpic.cn/psb?/V12a5mB10IdFhp/OEOh8aWJ1.sR55vln74d4WVK*sxIMYBbAETQeu2yOeA!/b/dFMBAAAAAAAA&bo=NQfxAwAAAAADF*I!&rf=viewer_4&t=5)

## 接口上传至服务器调用结果：

   ![接口上传至服务器调用结果](http://m.qpic.cn/psb?/V12a5mB10IdFhp/1E5iYVdHdytfBHRrFMBGhzADB21WpXOallIXg89bc64!/b/dFMBAAAAAAAA&bo=NQfxAwAAAAADF*I!&rf=viewer_4&t=5)
   
   
   ---
   
   至此小程序云开发----httpApi调用完工。
   
   #### 过程中遇到的问题
   
   - 在post获取数据库集合信息时，**第三方返回错误码“47001”**  
   在网上查了下，有很多遇到这个问题的。但如何解决说的大都不明不白，或者未解决，或者解决了帖子未更新。   
   
   - 本人遇到该问题时，先是在官方社区搜索了相关提问，发现官方回复，在postman上尝试调用如果无恙请检查自身代码。
   
   - 依言自行在postMan上自行查验一波，发现我不论如何变更入参格式依然是“47001”的报错。此时我的入参如下：
   ```js
   
        {
            access_token:"获取到的access_token",
            env: "云开发环境Id",
            limit: 10,
            offset: 0
        }
   
   ```
   - 多次查看对应httpApi文档，不断思索问题出在哪里。自身代码也没啥毛病啊，这是为啥呢？会不会是入参的问题呢？access_token已经在请求url上拼过一次是不是入参的时候就不需要了呢？入参的格式是什么呢？post默认的“application/x-www-form-urlencoded”,还是“application/json;”然后再一篇博客中看到，微信提供的接口入参格式为“application/json”。
   
   - 锁定了入参格式，但是再postMan上我是把所有的入参格式试了一遍的呀，那再试试入参里面去掉access_token呢？
   
   - ok,大功告成。终于见到了正常的返回数据。
   
   - 总结两点：
   
   **1，入参格式采用“application/json; charset=utf-8”；**
   
   **2，需要拼接access_token的接口入参请干掉access_token如上文中的代码**
   
   ## 源码链接
   [https://gitee.com/jioawoxiaoqi/vitaeServer/blob/master/routes/base.js](https://gitee.com/jioawoxiaoqi/vitaeServer/blob/master/routes/base.js)
