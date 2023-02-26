---
title: AI绘画 MAC安装Stable Diffusion webUI保姆级教程
top: false
cover: false
toc: true
mathjax: true
date: 2023-02-26 17:46:29
password:
summary:
tags:
img: /medias/featureimages/1.png
categories:
---
# 【AI绘画】Mac安装stable-diffusion-webui绘制AI妹子保姆级教程

## 1.作品图

![](https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/ai/girl/1.png)

![](https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/ai/girl/3.png)

## 2.准备工作

目前网上能搜到的stable-diffusion-webui的安装教程都是Window和Mac M1芯片的，而对于因特尔芯片的文章少之又少，这就导致我们还在用老Intel 芯片的Mac本，看着别人生成美女图片只能眼馋。所以小卷这周末折腾了一天，总算是让老Mac本发挥作用了。先来说说准备工作：

* Mac笔记本操作系统版本 >= 13.2.1 （亲测10.0版本各种问题无法运行，无奈花了一小时升级系统）
* Python3.10.6版本（已安装其他版本也不要紧，后面我们用Conda做版本控制）
* stable-diffusion-webui代码下载，下载地址：https://github.com/AUTOMATIC1111/stable-diffusion-webui

## 3.安装步骤

### 3.1 依赖安装

从github上把stable-diffusion-webui的源代码下载下来，进入到stable-diffusion-webui目录下，执行

```python
pip install -r requirements_versions.txt
```

这一步是安装Python项目运行所有需要的依赖，这步很大概率出现无法安装gfpgan的问题：Couldn't install gfpgan

![](https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/ai/stable-diffusion-webui/1.png)

解决方法：

网络连接超时的问题，更改pip使用国内镜像库，重试几次。这个问题暂无明确解法，如果无法解决可继续往下走

### 3.2pip更换国内镜像库

更换方法参考：https://blog.csdn.net/qq_45770232/article/details/126472610

### 3.3安装anaconda

这一步是方便对Python做版本控制，避免卸载重新安装不同版本的Python。

下载安装地址：https://www.anaconda.com/

从官网下载一路点击安装就行。

#### Conda添加环境变量

安装完成后，打开终端，输入conda，如果是无法识别的命令。需要配置环境变量，配置方法：

修改`.bash_profile`添加自己安装conda的路径，命令如下：

```shell
vim ~/.bash_profile

# 打开文件后，写入下面这行到文件里，注意替换路径
export PATH="/Users/(你自己的路径)/anaconda3/bin:$PATH"
```

接着`:wq`保存退出，`source ~/.bash_profile`使配置生效

#### 修改conda源为国内镜像库

执行命令如下：

```shell
# 如果没有会创建condarc文件
vim ~/.condarc

# 打开文件后，把下面的内容粘贴进去保存
channels:
  - https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
  - https://mirrors.ustc.edu.cn/anaconda/cloud/conda-forge/
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
  - defaults
show_channel_urls: true
```

### 3.4 创建虚拟环境

执行命令：

```shell
conda create -n sd python=3.10.6
```

这样就创建了一个名称为`sd`的虚拟环境

### 3.5 安装依赖

按上面的操作把pip替换为国内镜像源后，激活虚拟环境，并安装需要的依赖包

执行命令：

```shell
# 进入stable-diffusion-webui的文件目录
cd stable-diffusion-webui

# 激活虚拟环境
conda activate sd

# 安装所需依赖
pip3 install -r requirements_versions.txt
```

这一步如果没任何问题，安装过程算是有惊无险完成了一半。如果有问题，请自行百度谷歌搜索解决，欢迎留言遇到的问题和解法

## 4. 模型安装

### 4.1下载模型

**官方模型下载（checkpoint模型）**

下载地址：https://huggingface.co/CompVis/stable-diffusion-v-1-4-original

下载 `sd-v1-4.ckpt` 或者 `sd-v1-4-full-ema.ckpt`。

**LoRA模型**

这个应该是大家最喜欢的模型了，懂的都懂。。。

下载地址：https://civitai.com/models/6424/chilloutmix

![](https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/ai/stable-diffusion-webui/5.png)

右上角Download下载，其他模型大家可自行在这个网站上探索，非常的多，这里推荐几个热门的:

[korean-doll-likeness](https://civitai.com/models/7448/korean-doll-likeness)

![](https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/ai/stable-diffusion-webui/6.png)

### 4.2 安装模型

- 对于**checkpoint模型**，请移动到`stable-diffusion-webui/models/Stable-diffusion`⽬录下
- 对于**LoRA模型**，请移动到`stable-diffusion-webui/models/Lora`目录下
- 其他模型按对应的类型移到对应的目录下



## 5. 运行项目

### 5.1 跳过GPU检测

前面说了，咱们用的是老Mac本了，Intel芯片，显卡也用不了。只能用CPU进行计算，跳过GPU的配置如下：

执行命令：

```shell
# 打开配置文件
vim ~/.bash_profile

# 把下面两行拷贝进去，保存后source命令使其生效
export COMMANDLINE_ARGS="--lowvram --precision full --no-half --skip-torch-cuda-test"
export PYTORCH_ENABLE_MPS_FALLBACK=1
```

### 5.3 项目代码修改

因为网络访问的问题，我们需要将代码里有些地方进行修改。修改如下：

**修改lanuch.py文件**

* 修改def prepare_environment()方法下的两处位置

1. torch_command中修改`torch==1.13.1 torchvision==0.14.1`把原有的版本号数字后面的其他内容去掉

2. 该方法下所有`https://github.com`开头的链接，前面都加上`https://ghproxy.com/`这样链接就变成如下格式了：`https://ghproxy.com/https://github.com/`

如图所示

![](https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/ai/stable-diffusion-webui/7.png)



### 5.3 运行项目

上面我们使用conda进入了虚拟环境，然后再运行项目即可，执行命令：

```shell
# 激活虚拟环境sd
conda activate sd 

# 进入到stable-diffusion-webui目录下
cd stable-diffusion-webui

# 运行项目
python launch.py
```

这一步如果人品好的话，第一次就能全部正常运行完，运行完之后，出现`http://127.0.0.1:7860`字样说明运行成功了，浏览器打开这个地址就能开始愉快地玩耍了，玩耍方式自行探索哦~

![](https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/ai/stable-diffusion-webui/9.png)

## 6.相关问题

### pip install -r requirements.txt时报错，有一些依赖没有安装上

解决方法：手动安装一下依赖包

```shell
pip install 缺少的依赖包
```

## 7.模型下载及图片下载

文章里用到的模型和图片下载方式：公众号`卷福同学`内发关键词`AI绘画`获取

![](https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/ai/stable-diffusion-webui/11.png)

## 关于我

欢迎关注我的技术公众号：**卷福同学**

技术文章第一时间会在公众号上发布哦~

![公众号](https://raw.githubusercontent.com/longbig/hexo-blogs/master/source/img/wechat/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.jpeg)
