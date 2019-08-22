## 1. 小程序功能

- 古诗词大全
- 成语大全
- 成语接龙
- 诗词飞花令
- 诗词分享、收藏
- 诗词接龙
- 唐诗宋词起名字
- 百家姓
- 猜谜语

## 2. 小程序预览：

<br>
<img src="https://puui.qpic.cn/vupload/0/20190809_1565314668113_0ascs92ffzmi.jpeg/0" width="240" />
<br>

## 3. 部分截图

####  首页

<img src="https://puui.qpic.cn/vupload/0/20190808_1565261440163_r41j6cqlqhg.png/0" width="225" height="400"  />


####  列表页

<img src="https://www.cnblogs.com/images/cnblogs_com/cckui/1107952/o_1558438667474.jpg" width="225" height="400"  />

####  详情页 分享页


<img src="https://puui.qpic.cn/vupload/0/20190808_1565261462797_mrto0hl9w6k.png/0" width="225" height="400"  />

####  唐诗宋词

<img src="https://puui.qpic.cn/vupload/0/20190809_1565314768436_v7jcaf044iq.jpeg/0" width="225" height="400"  />



####  成语接龙

<img src="https://www.cnblogs.com/images/cnblogs_com/cckui/1107952/o_1558438538843.jpg" width="225" height="400"  />


## 4. 项目结构

```
.
├── README.md
├── project.config.json                              // 项目配置文件
├── cloudfunctions | 云环境                           // 存放云函数的目录
│   ├── login                                        // 用户登录云函数
│   │   ├── index.js
│   │   └── package.json
│   └── collection_get                               // 数据库查询云函数
│   │   ├── index.js
│   │   └── package.json
│   └── collection_update                               // 数据库更新云函数
│       ├── index.js
│       └── package.json
└── miniprogram
    ├── images                                        // 存放小程序图片
    ├── lib                                           // 配置文件
    ├── pages                                         // 小程序各种页面
    |   ├── index                                     // 首页
    |   └── menu                                      // 分类页
    |   └── user                                      // 用户中心
    |   └── search                                    // 搜索页
    |   └── list                                      // 列表页 搜索结果页
    |   └── detail                                    // 详情页
    |   └── collection                                // 收藏页
    |   └── find                                      // 发现页
    |   └── idiom-jielong                             // 成语接龙页
    |   └── poet                                      // 作者页
    |   └── baijiaxing                                // 百家姓
    |   └── xiehouyu                                  // 歇后语
    |   └── poet                                      // 作者页
    |   └── suggest                                   // 建议反馈
    |   └── ...                                       // 其他
    ├── style                                         // 样式文件目录
    ├── app.js                                        // 小程序入口文件
    ├── app.json                                      // 全局配置
    └── app.wxss                                      // 全局样式

```

## 5. 封装云函数操作数据库

本项目是使用的小程序云开发。云开发提供了一个 JSON 数据库，用户可以直接在云端进行数据库增删改查，但是，小程序对用户操作数据的权限进行了一定的限制（例如数据update、一次性get记录的条数限制等），所以，这里主要采用云函数来操作数据库。

### 查询数据、分页查询

函数根目录上右键，在右键菜单中，选择创建一个新的 Node.js 云函数，我们将该云函数命名为 collection_get。

编辑 index.js：

```
// 云函数入口文件
const cloud = require('wx-server-sdk')
cloud.init()

const db = cloud.database()

exports.main = async (event, context) => {

  /**
   * page: 第几页
   * num: 每页几条数据
   * condition： 查询条件，例如 { name: '李白' }
   */

  const {database, page, num, condition} = event
  console.log(event)

  try {
    return await db.collection(database)
                  .where(condition)
                  .skip(num * (page - 1))
                  .limit(num)
                  .get()
  } catch (err) {
    console.log(err)
  }
}

```

#### 使用 collection_get 云函数

例如，按照查询条件`{tags: '唐诗三百首'}`查询诗词列表，每页`num = 10`条数据:

```
let {list, page, num} = this.data
let that = this

this.setData({
    loading: true
})

wx.cloud.callFunction({
    name: 'collection_get',
    data: {
        database: 'gushici',
        page,
        num,
        condition: {
            tags: '唐诗三百首'
        }
    },
    }).then(res => {
        if(!res.result.data.length) { // 没搜索到
            that.setData({
                loading: false,
                isOver: true
            })
        } else {
            let res_data = res.result.data
            list.push(...res_data)
            that.setData({
                list,
                page: page + 1, // 页面加1
                loading: false
            })
        }
    })
    .catch(console.error)
}
```

### 更新数据

注意，当我们向数据库中添加记录时，系统会自动帮我们为每条记录添加上用户的 `openid` 字段，但如果，数据表是自己用 json/csv 文件导入的，就不存在 `openid` 字段，此时，当更新这个数据表时，系统会认为你不是创建者，所以也就无法更新。

此时，就需要通过云函数更新数据库，新建云函数 collection_update, 编辑 index.js:

```
// 更新数据 - 根据 _id 更新已打开人数
const cloud = require('wx-server-sdk')
cloud.init()

const db = cloud.database()
const _ = db.command

exports.main = async (event, context) => {

  const { id } = event
  console.log(event)

  try {
    return await db.collection('gushici').doc(id)
      .update({
        data: {
          opened: _.inc(1)
        },
      })
  } catch (e) {
    console.error(e)
  }
}
```

#### 使用 collection_update 云函数

更新某_id数据的打开人数：

```
let _id = e.currentTarget.dataset.id
wx.cloud.callFunction({
    name: 'collection_update',
    data: {
        id: _id
    },
}).then(res => {
    console.log(res.data)
})
.catch(console.error)
```


## 6. 数据库模糊查询

小程序云开发可以使用正则表达式进行模糊查询。例如， 根据用户输入关键词，查询标题中存在改关键词的古诗词。

```
let database = 'gushici'
let condition =  {
    name: {
        $regex:'.*'+ inputValue,
        $options: 'i'
    }
}

let { list, page, num } = this.data
let that = this

this.setData({
    loading: true
})

// 模糊查询
wx.cloud.callFunction({
    name: 'collection_get',
    data: {
        database,
        page,
        num,
        condition
    },
}).then(res => {
    if (!res.result.data.length) { // 没搜索到
        that.setData({
            loading: false,
            isOver: true
        })
    } else {
        let res_data = res.result.data
        list.push(...res_data)
        that.setData({
            list,
            loading: false
        })
    }
})
.catch(console.error)
```



## 7. 分享或转发功能


小程序中页面触发转发的方式有两种：

* 1.在小程序的右上角选择转发，需要定义函数 Page.onShareAppMessage，如果当前页面没有定义此事件，则点击后无效果。
* 2.通过给 `button` 组件设置属性 `open-type="share"`，可以在用户点击按钮后触发 Page.onShareAppMessage 事件，如果当前页面没有定义此事件，则点击后无效果。

用户还可以在 Page.onShareAppMessage 事件中自定义转发后显示的标题、图片、路径：

```
onShareAppMessage(res) {
    let id = wx.getStorageSync('shareId')
    if (res.from === 'button') {
        // 来自页面内转发按钮
        console.log(res.target)
    }
    return {
        title: `跟我一起挑战最长的成语接龙吧！`,
        path: `pages/find/find`,
        imageUrl: '/images/img.jpg',
    }
},
```

#### 注意：转发成功/失败的 callback 已经被官方废弃，所以理论上小程序是无法得知用户是否将页面分享成功的


## 8. 用户授权

详情请参考文章：[微信小程序之授权](https://www.cnblogs.com/cckui/p/10000738.html)

## 9. 需要注意的几个坑

### 查询不到数据

数据表中明明有数据，但是 collection.get 到的却为空。解决：可以在云开发控制台中打开数据库权限设置，设置权限。

### 更新数据失败

collection.update 函数调用成功单返回的却是0行记录被更新，因为小程序端不允许更新没有 openid 字段的数据。解决：可以通过云函数更新数据库。


### background 图片 url 不能为本地图片

解决：1：将图片上传到服务器，填写服务器上的图片路径地址。2：将图片转为 base64 编码。

### 往云数据库中批量导入 json 数据失败

原因：请看文档：[https://developers.weixin.qq.com/miniprogram/dev/wxcloud/guide/database/import.html](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/guide/database/import.html)

解决：去掉json数据 `{}`之间的逗号, 如果最外层为 `[]`，也必须去掉, 最终形如：

```
{
    "index": "作者_1",
    "type": "作者",
    "poet": "李白",
    "abstract": "李白（701年－762年），字太白，号青莲居士，唐朝浪漫主义诗人，被后人誉为“诗仙”..."
}
{
    "index": "作者_2",
    "type": "作者",
    "poet": "白居易",
    "abstract": "白居易（772年－846年），字乐天，号香山居士..."
}
```

## 源码链接
[https://github.com/caochangkui/miniprogram-project](https://github.com/caochangkui/miniprogram-project)
