---
title: ComfyUI的生态搭建
tags:
  - ai
  - AIGC
  - ComfyUI
categories:
  - [ 人工智能, AIGC ]
top_img: /img/top.png
cover: /img/ai.jpg
series: ComfyUI
series_order: 1
---

> 注意： 因为内部所有的生态皆依赖于github开源社区，进行资源下载时最好挂上梯子。

## ComfyUI安装

![](comfyui-panel.jpg)

### 下载

[Git地址](https://github.com/comfyanonymous/ComfyUI?tab=readme-ov-file)

![](comfyui-download.png)

### 安装方式

直接解压，双击run_nvidia_gpu.bat文件即可直接运行。

> [WIKI地址](https://comfyui-wiki.com/zh)

## ComfyUI-manage安装

ComfyUI-manage是一套对ComfyUI插件模型进行统一化管理的一套插件。安装后可以很方便的对ComfyUI的模型进行下载管理
![](cm-panel.png)

### 下载

[Git地址](https://github.com/ltdrdata/ComfyUI-Manager/tags)
> 直接下载最新的tag里的zip压缩包
![](cm-dw.png)

### 安装方式

将zip包解压放到ComfyUI地址\ComfyUI\custom_nodes\文件夹内，重启ComfyUI即可。重启后界面出现图中的图标打开即可。
![](cm-s1.png)
![](cm-s2.png)

## 基础算法模型使用

### 使用comfyUI的Model Manager界面下载模型（不推荐，很多下载下来的模型存在问题，无法使用）

![](cmk-dw.png)

### 通过国内的模型网站下载

* 可以从[liblib](https://www.liblib.art/)下载国内玩家自己训练的基础模型和lora.使用基础模型即可完成最简易的AIGC工作流的使用。
* 下载后的包直接放入`/ComfyUI/models/...`对应的文件夹下即可
  ![](model-path.png)

> 上诉步骤执行完毕即可开始使用我们的ComfyUI进行基础的图文AI处理了

## SDXL模型使用

#### 安装SeargeSDXL面板插件

[searge下载地址](https://github.com/SeargeDP/SeargeSDXL/releases/tag/v4.3.2)
> 不要使用ComfyUi-manage下载，有问题

下载SeargeSDXL-4_3_2.zip解压放至`\ComfyUI\custom_nodes\`文件夹内，
之后按照[文档](https://github.com/SeargeDP/SeargeSDXL)点击下载以下组件并放入对应的文件夹
![](seagre-dw.png)
> 注意：这里不推荐使用文档中的一键安装工具，因为github对于国内网络节点的速度支持很差，基本上都是安装失败。

#### 使用SeargeSDXL

通过ComfyUI界面打开工作流，位置在`<安装位置>/ComfyUI/custom_nodes/SeargeSDXL/workflow/`中。加载最新版本的JSON工作流文件即可。
![](sdxl-flow.png)
> 旧版本的工作流文件不用搭理。SeargeSDXL无论文生图，还是图生图都是同一套工作流界面
![](sdxl-show.png)

