---
title: 微信客服接入ChatGPT教程-外部群聊可用
top: false
cover: false
toc: true
mathjax: true
date: 2023-06-05 23:26:19
password:
summary:
tags:
categories:
---
# 【奶奶看了也不会】微信群聊(微信客服)接入ChatGPT教程

## 1.聊天效果展示

大家好，我是小卷。最近工作变卷了，都已经一个月没更新文章了。今天来教教大家怎么给微信群聊的智能客服接入ChatGPT。和之前企业微信机器人不同的是，这次是可以外部微信群使用的。用的人会更多哦~

先看聊天效果

<img src="https://raw.githubusercontent.com/longbig/hexo-blogs/main/source/img/chatgpt/wechat_kefu/0_3.png" style="zoom: 50%;" />

从群聊获取到客服名片，然后和客服机器人私聊对话，就和ChatGPT对话一样的效果。



## 2.微信客服配置

微信客服开发文档：[微信客服开发](https://developer.work.weixin.qq.com/document/path/94670)

登录企业微信管理后台，配置微信客服，地址：https://work.weixin.qq.com/wework_admin/frame#/app/servicer

### 2.1配置获取

获取应用程序需要的配置，对应程序里的配置名如图：

<img src="https://raw.githubusercontent.com/longbig/hexo-blogs/main/source/img/chatgpt/wechat_kefu/0_4.png" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/longbig/hexo-blogs/main/source/img/chatgpt/wechat_kefu/0_5.png" style="zoom:50%;" />

![](https://raw.githubusercontent.com/longbig/hexo-blogs/main/source/img/chatgpt/wechat_kefu/0_6.png)

上面那些配置先拷贝下来，我们等会要用



### 2.2创建客服账号

按图里的位置建个客服账号即可，头像名称自己设置

![](https://raw.githubusercontent.com/longbig/hexo-blogs/main/source/img/chatgpt/wechat_kefu/0_7.png)



### 2.3 设置通过API管理微信客服账号

页面滑到最下面，我们将刚刚创建的客服账号通过API的方式进行管理。客服账号需要配置人工接待人员哦~

![](https://raw.githubusercontent.com/longbig/hexo-blogs/main/source/img/chatgpt/wechat_kefu/0_8.png)

![](https://raw.githubusercontent.com/longbig/hexo-blogs/main/source/img/chatgpt/wechat_kefu/0_9.png)



## 3.应用部署

### 3.1应用配置

将2.1节获取的4个配置，加上你的ChatGPT账号的key放到我的代码配置文件`application.properties`里进行配置。

代码获取方式：公众号`卷福同学`内，发关键词`ChatGPT企业微信`获取。

确实没ChatGPT账号的友友们，也可以公号内联系购买

![](https://raw.githubusercontent.com/longbig/hexo-blogs/main/source/img/chatgpt/wechat_kefu/0_10.png)

### 3.2应用发布

我们用maven打包该应用，命令行执行`mvn package`打包整个项目为`application.jar`文件，接着把该文件上传到自己的海外服务器上（香港的不行哦）

服务器上执行命令

```shell
nohup java -jar application.jar >log.txt &
```

然后`tail -f log.txt`查看启动日志，看到`Started MultiFunctionApplication in 3.297 seconds`类似字样的日志说明启动成功了。详细使用部署步骤可参考我之前写的文章：[【奶奶看了都会】ChatGPT3.5接入企业微信，可连续对话](https://zhuanlan.zhihu.com/p/611555021)

![](https://raw.githubusercontent.com/longbig/hexo-blogs/main/source/img/chatgpt/wechat_kefu/0_11.png)



## 4.设置接收事件服务器

配置`URL`和之前让你拷贝的`Token`、`EncodingAESKey`保存没问题就配置好了

URL格式为：`http://[替换为你的服务器公网IP]:8080/receiveMsgFromWechatKf`

注意认证的企业微信不能用IP，需要用备案域名，DNS解析到你的服务器上

![](https://raw.githubusercontent.com/longbig/hexo-blogs/main/source/img/chatgpt/wechat_kefu/0_12.png)

## 5.创建群聊

通过`升级服务` 转到`客户群服务`配置里，新增一个外部群，然后手机端登录企业微信，管理员在这个群开启`客服助理`功能。这样我们的微信群聊里就有这个客服机器人了

![](https://raw.githubusercontent.com/longbig/hexo-blogs/main/source/img/chatgpt/wechat_kefu/0_13.png)

![](https://raw.githubusercontent.com/longbig/hexo-blogs/main/source/img/chatgpt/wechat_kefu/0_14.png)

## 6.聊天效果

<img src="https://raw.githubusercontent.com/longbig/hexo-blogs/main/source/img/chatgpt/wechat_kefu/0_3.png" style="zoom:33%;" />



## 7.群聊说明

目前该方法仍然只能支持私聊，不支持群聊。

这就有小伙伴有疑问了，绕了一圈，想要的群聊功能还是没有。。。

说明：微信官方没有提供群聊的交互接口，现在你看到的微信群聊和ChatGPT、midjourney互动的机器人，都是通过wechaty或者itchat微信代理的方式，有需要的可以联系交流

## 8.代码获取

文章所用代码获取方式，公众号`卷福同学`内，发关键词`ChatGPT企业微信`获取





