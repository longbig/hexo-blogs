---
title: ChatGPT接入个人微信
top: false
cover: false
toc: true
mathjax: true
date: 2023-02-12 14:56:35
password:
summary:
tags:
categories:
---
# ChatGPT接入个人微信（国内也能正常使用）

chatGPT最近突然又大火起来了，而且这次不是一般的火，带有浓浓的商业气息火了。各个互联网大厂都开始进军了，感觉要来一场ChatGPT的军备竞赛一样，看看谁先获取国内的地盘。

作为吃瓜群众，我们也能个人使用ChatGPT，现在小卷来教大家更高级的玩法，就是用个人微信接入ChatGPT，个人微信变成一个聊天机器人，话不多说，先看效果，群聊或者私聊都可以触发

（最好用微信小号试验，有封号的风险）

<img src="https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/chatgpt/weixin/6.jpeg" style="zoom: 25%;" />



## 1.注册账号

注册教程看我之前的文章：[ChatGPT保姆级注册教程](https://longbig.github.io/2023/02/12/ChatGPT%E4%BF%9D%E5%A7%86%E7%BA%A7%E6%B3%A8%E5%86%8C%E6%95%99%E7%A8%8B/)

**如果有不会用梯子的，或是注册困难，或想用现成的账号，可以咨询我啊，提供有偿帮助**

## 2.创建API keys

账号建好之后，登录openai，并创建一个API keys，这个key非常重要，这个是我们程序访问openai接口必须的

登录地址：https://platform.openai.com/login/

登录之后右上角头像那里点击`View API keys`进入创建页面

<img src="https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/chatgpt/weixin/1.png" style="zoom:25%;" />



到创建界面后，创建一个新的秘钥，注意啊，只有第一次创建的时候可以copy下来。所以我们创建好后直接拷贝到本地，找个地方存着，一会要用

![](https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/chatgpt/weixin/2.png)

## 3.安装部署程序

### 3.1准备工作

* 自己电脑上安装golang运行环境，程序是用Go语言写的。安装方法可自行百度或Google
* 下载程序源码，下载地址：https://gitee.com/longbig/chatGpt_wechat

### 3.2配置修改

源码下载后，在chatGpt_wechat目录下找到config.json文件，修改配置

![](https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/chatgpt/weixin/4.png)



修改内容为将`api_key`替换为上面自己在openai上创建的API Keys，其他配置无需修改

![](https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/chatgpt/weixin/4_1.png)

### 3.3运行程序

命令行在chatGpt_wechat目录下，运行main函数，命令如下：

```go
go run main.go

// 或者在项目目录下执行  go build ，编译出可执行程序后，执行可执行程序即可
```

程序运行后，会在命令行窗口出现一个二维码，用微信扫码登录即可。注意微信号需要实名认证啊，扫码的微信号就是聊天机器人

（先用微信小号进行测试，有封号的麻烦）

![](https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/chatgpt/weixin/5.png)

### 3.4测试效果

出现登录成功的提示后就可以开始聊天了，如下图是我在微信问机器人的对话以及命令行打印的日志，可以看到无需梯子，能正常使用

<img src="https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/chatgpt/weixin/6.jpeg" style="zoom:25%;" />

![](https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/chatgpt/weixin/7.png)

### 3.5 相关问题处理

终止go程序 重新运行后会报一个错

**[WARNING]2023/02/10 20:00:03 logger.go:33: login error: write storage.json: bad file descriptor**

解决方法：

把chatGpt_wechat目录下的`storage.json`文件删除，再重新登录即可。


## 关于我

欢迎关注我的技术公众号：**卷福同学**

技术文章第一时间会在公众号上发布哦~

![公众号](https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/wechat/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.jpeg)

