“众所周知，云开发目前只支持Node js,如何突破这个限制？且看高手们如何用Python打通云开发七经六脉，让云开发的使用更加行云流水。”

### 一、项目背景
由于BBC六分钟官网直接访问，平时看BBC新闻也是颇费力气。于是我想，为何不自己做个“BBC新闻摘要小程序”呢？

### 二、小程序实现策略
做一个方便自己的小程序，不需要很复杂的架构。：用闲置的DigitalOcean服务器下载音频和对话脚本，传回国内COS。然后用小程序展示就搞定啦。

我还清晰地记得，5月17号，微信开发者社区推送了一条消息：可以外网上传文件到云存储了！

### [云开发文档](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/reference-http-api/index.html)

说实话，之前也考虑过小程序里的云存储，但好像只能通过开发者工具人工上传。所以收到这条推送之后第一时间就去看了文档，理了一遍小程序云存储上传逻辑是：

#### step1: 用小程序Appid 和Appsecret 拿 Access_token
#### step2: 用拿到的Access_token拿文件上传URL和相关参数
#### step3: 用拿到的URL和相关参数拼接完整的POST请求来上传文件

从写抓取脚本和小程序制作上线花了大概一天的时间。

### 三、如何用Python实现云开发的文件上传？
理顺了逻辑，接下来就是写代码了。 Python用来http请求的，选用requests。 首先：拿access_token
```javascript
def get_token():
    token_url = "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=" + ID + "&secret=" + SECRET
    try:
        token = requests.get(token_url)
        token = token.json()
        return token["access_token"]
    except Exception as e:
        logging.error(e)
```
然后，用access_token获取文件上传相关参数
```javascript
def get_upload_url(token, env, path):

    post_url = "https://api.weixin.qq.com/tcb/uploadfile?access_token=" + token
    playload = json.dumps({"env":env, "path":path})

    try:
        upload = requests.post(post_url, data=playload)
        return upload.json()
    except Exception as e:
        logging.error(e)
```
拿到参数后需要解析重新拼接来完成上传：
```javascript
def parse_form(res):

    form = {}
    
    form["key"] = res["url"].split("/")[-1]
    form["Signature"] = res["authorization"]
    form["x-cos-security-token"] = res["token"]
    form["x-cos-meta-fileid"] = res["cos_file_id"]
    return (form, res["url"])
```

最后，就是上传了：
```javascript
def upload(res, file):
    
    form = res[0]
    upload_url = res[1]
    with open(file, "rb") as f:
        form["file"] = f.read()
        
    try:
        success = requests.post(upload_url, files=form)
    except Exception as e:
        logging.error(e)
```
完整源码可以点击阅读原文进行下载。

### 四、延展思考
其实Python实现小程序·云开发的文件上传，只是一个小功能实战，但是由此给我们的启示是，可以利用云开发的HTTP API去实现各类语言和云开发的对接。

关于云开发HTTP API的使用文档，可参考《[云开发新能力，支持HTTP调用API](https://mp.weixin.qq.com/s/O1I2c5zirqwrdlPy2CvRbg)》

最后放上小程序二维码，以及效果预览。
![](https://puui.qpic.cn/vupload/0/20190724_1563955777872_efm3wxb2pg5.jpeg/0)

![](https://puui.qpic.cn/vupload/0/20190724_1563955806475_b9tib1bgdw5.jpeg/0)

# 源码链接
[https://mp.weixin.qq.com/s/H8pMH0X5_cnd5BJcZ3qr6w](https://mp.weixin.qq.com/s/H8pMH0X5_cnd5BJcZ3qr6w)
