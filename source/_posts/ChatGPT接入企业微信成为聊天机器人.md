---
title: ChatGPT接入企业微信成为聊天机器人
top: false
cover: false
toc: true
mathjax: true
date: 2023-02-19 10:53:55
password:
summary:
tags:
categories:
---
# ChatGPT接入企业微信成为聊天机器人教程

## 1.聊天效果

上次给大家讲了ChatGPT接入个人微信的方法，但是个人微信容易被封号。这次就教大家接入企业微信，不会再被封号哦~ 话不多说，直接看机器人的聊天效果。基本能实现ChatGPT的聊天效果了。我们实现的是在企业微信中添加一个机器人，让机器人后台与chatGPT互通，这样和机器人对话就是和ChatGPT对话了。

![](https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/chatgpt/wechat_business/1.png)

## 2.接入步骤

### 2.1 准备工作

* 一台配置公网IP的服务器，或是有阿里云函数计算的域名
* 服务器配置了Java运行环境
* 有额度的chatGPT账号，并创建了账号的api key，创建过程可以往下看
* 以上配置有了才能进行后面的步骤哦~
* 代码获取方式：公众号 `卷福同学` 内，发关键词`ChatGPT企业微信`获取
* 需要ChatGPT账号或是API KEY的可以公众号内联系



### 2.1创建企业微信团队

首先下载企业微信，登录注册创建一个企业微信团队。不要选个人团队（亲测无法应用机器人）。具体步骤可自行探索。这里不多说了

### 2.2添加机器人

PC端登录地址：https://work.weixin.qq.com/wework_admin/frame#apps

先登录上一步创建好的企业微信账号

然后添加自建应用

<img src="https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/chatgpt/wechat_business/2.png" style="zoom:33%;" />

填写名称、上传logo图片，创建应用

<img src="https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/chatgpt/wechat_business/3.png" style="zoom:33%;" />

### 2.3设置企业可信IP

创建好应用后，在应用详情里，开发者接口配置那里，配置企业可信IP（准备工作里让你准备的服务器的公网IP）

这一步的作用是：你给机器人发消息，消息会转给配置的IP，配置可信是为了让你的IP在白名单里。否则微信不会转发消息的。

<img src="https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/chatgpt/wechat_business/4.png" style="zoom:33%;" />

### 2.4 设置API接收

上一步配置了IP，还要继续配置API接收，简单说就是微信转发消息到哪个路径里。

<img src="https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/chatgpt/wechat_business/5.png" style="zoom:33%;" />

这一步比较复杂，我们慢慢往后看。

#### 2.4.1 配置URL

第一次配置需要验证消息，就是你配置的接口，微信第一次验证通过后，才允许发消息。需要配置URL和Token

配置方式：

URL配置： IP:端口/接收消息路径

示例 http://127.0.0.1:8080/receiveMsgFromWechat  （要替换IP哦）

<img src="https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/chatgpt/wechat_business/6.png" style="zoom:33%;" />

#### 2.4.2 验证消息

接着重要的来了，我们必须按照微信要求的格式配置接口，官方文档：[接收消息与事件](https://developer.work.weixin.qq.com/document/10514)

小卷已经把代码整好了，获取地址：公众号 `卷福同学` 内，发关键词`ChatGPT企业微信`获取


需要替换application.properties文件中的5个参数，具体要换成什么看代码库里的描述

* chatgpt.apiKey
* wechat.sToken
* wechat.sEncodingAESKey
* wechat.sCorpID
* wechat.corpsecret

接口Java示例代码：

```java
    @GetMapping("/receiveMsgFromWechat")
    public String receiveMsgFromDd(@RequestParam("msg_signature") String msg_signature,
                                   @RequestParam("timestamp") String timestamp,
                                   @RequestParam("nonce") String nonce,
                                   @RequestParam("echostr") String echostr) throws Exception {
        WXBizMsgCrypt wxcpt = new WXBizMsgCrypt(sToken, sEncodingAESKey, sCorpID);

        String sEchoStr = null;
        try {
            sEchoStr = wxcpt.VerifyURL(msg_signature, timestamp,
                    nonce, echostr);
            log.info("verifyurl echostr: " + sEchoStr);
            // 验证URL成功，将sEchoStr返回
            return sEchoStr;
        } catch (Exception e) {
            //验证URL失败，错误原因请查看异常
            log.error("verifyurl error,e={}", e);
            return "";
        }
    }
```

把代码下载完后，部到你的服务器上，再点击保存，即可验证通过。验证通过后API接收消息就像下面那样

![](https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/chatgpt/wechat_business/7.png)

## 3.消息接口开发

### 3.1接收消息接口开发

上一步验证完成后，我们就可以用配置的路径接收消息了。但是微信是用的Post方式发消息，所以需要再设置个Post方式的接口，路径还是一样的，上一步配置的GET接口可以注释掉了。消息入参有改变。

官方文档：[使用接收消息](https://developer.work.weixin.qq.com/document/10514#%E4%BD%BF%E7%94%A8%E6%8E%A5%E6%94%B6%E6%B6%88%E6%81%AF)

这里小卷也把示例代码写好了，大家直接用

```java

    @PostMapping(value = "/receiveMsgFromWechat",
            consumes = {"application/xml", "text/xml"},
            produces = "application/xml;charset=utf-8")
    public String receiveMsgFromDd(@RequestParam("msg_signature") String msg_signature,
                                   @RequestParam("timestamp") String timestamp,
                                   @RequestParam("nonce") String nonce,
                                   @RequestBody WechatXmlDTO body) throws Exception {
        WXBizMsgCrypt wxcpt = new WXBizMsgCrypt(sToken, sEncodingAESKey, sCorpID);

        String sEchoStr = null;
        try {
            String msg = body.getEncrypt();
            String xmlcontent = wxcpt.decrypt(msg);
            log.info("xml content msg: " + xmlcontent);
            String data = StringUtils.substringBetween(xmlcontent, "<Content><![CDATA[", "]]></Content>");
            return data;
        } catch (Exception e) {
            //验证URL失败，错误原因请查看异常
            log.error("DecryptMsg msg error,e={}", e);
            return "";
        }
    }
```

到这一步，我们给机器人发的消息都会转发到自己服务器了，有调试需求的朋友可以自己调试试

### 3.2调Openai的接口

关键的一步来了，这步是调Openai的GPT3.0接口，使用它的文本补齐功能实现对话。需要的自行查看官方接口文档：[OpenAI官方接口文档](https://platform.openai.com/docs/api-reference/completions)

我们用Java开发HTTP POST请求就行，然后需要用到你的账号的API key

#### 3.2.1 chatGPT账号API key获取

请求Openai的接口需要账号key，获取方式：

* 登录openai官网：https://platform.openai.com/account/api-keys
* 点击`create New secret key`创建一个key，拷贝下来
* 没有chatGPT账号，或者没有key的可以找我，优惠价

![](https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/chatgpt/wechat_business/8.png)

#### 3.2.1 请求Openai功能开发

就一个POST请求，注意替换API key，示例代码如下。相关参数解释可以看官方接口文档[completion文档](https://platform.openai.com/docs/api-reference/completions)

```java
Map<String, String> header = Maps.newHashMap();
String drawUrl = "https://api.openai.com/v1/completions";
String cookie = "";
header.put("Authorization", "Bearer 【替换API KEY】");
Map<String, Object> body = Maps.newHashMap();
body.put("model", "text-davinci-003");
body.put("prompt", text);
body.put("max_tokens", 1024);
body.put("temperature", 1);
MediaType JSON1 = MediaType.parse("application/json;charset=utf-8");
RequestBody requestBody = RequestBody.create(JSON1, JSON.toJSONString(body));

String response = OkHttpUtils.post(drawUrl, cookie, requestBody, header);
```

### 3.3 回传消息给企业微信

上一步调了Openai的接口后，就得到了GPT对话的结果，现在要将结果回传到企业微信里，实现对话聊天，企业微信发送应用消息文档：[发送应用消息](https://developer.work.weixin.qq.com/document/path/90236)

具体步骤：

* 先获取接口调用的accessToken，有效期2小时
* 再通过accessToken，调应用消息推送接口，可配置具体要接收消息的人，群等等。
* 需要用到企业id，获取方法往下看
* 需要用到自建应用的secret



#### 3.3.1 获取企业ID

![](https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/chatgpt/wechat_business/9.png)



#### 3.3.2 获取自建应用的secret

![](https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/chatgpt/wechat_business/10.png)

示例代码：

```java
				String accessToken = null;
        try {
            accessToken = getAccessToken();
        } catch (Exception e) {
            log.error("sendMsg getAccessToken error,e={}", e);
        }
        String url = "https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=" + accessToken;
        String body = "{\n" +
                "   \"touser\" : \"" + touser + "\",\n" +
                "   \"msgtype\" : \"text\",\n" +
                "   \"agentid\" : 1000003,\n" +
                "   \"text\" : {\n" +
                "       \"content\" : \"" + msg + "\"\n" +
                "   },\n" +
                "   \"safe\":0,\n" +
                "   \"enable_id_trans\": 0,\n" +
                "   \"enable_duplicate_check\": 0,\n" +
                "   \"duplicate_check_interval\": 1800\n" +
                "}";
        MediaType JSON1 = MediaType.parse("application/json;charset=utf-8");
        RequestBody requestBody = RequestBody.create(JSON1, body);
        log.info("send msg:{}", requestBody);
        OkHttpUtils.post(url, "", requestBody, Maps.newHashMap());
```

到此所有开发工作都完成了，可以将服务部署试验功能了！！！


## 关于我

欢迎关注我的技术公众号：**卷福同学**

技术文章第一时间会在公众号上发布哦~

![公众号](https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/wechat/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.jpeg)







