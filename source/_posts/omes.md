---
title: 一款工业化项目场景的设备管理系统OMES
tags:
  - 工业化
categories:
  - [ 工业化 ]
top_img: /img/top.png
cover: /img/bg.jpg
date: 2026-02-04 00:00:00
---

### 一、背景
随着时代的发展,信息化的普及。越来越多工业中小型项目需要运用到微信小程序、APP、数据管理平台进行项目设备的进行管理及数据采集。  
在此情形下,市面上大部分的设备厂商都组建了自己的设备云平台进行设备管理。提供了或免费或收费的云平台机制。大致可分为：
> 1. 平台免费使用,提高设备单价。
> 2. 设备保持原价,平台根据设备接入数量.少量免费,过量递增收费。(根据现有的调研来看，大部分需要用到信息化管理的项目场景，免费量级无法满足)。  
 
其中,厂商提供的设备管理平台又分为以下几大类:
> 1. 基于本地SCADA的设备管理云平台。
>    1. 优点：由于画面可完全自定义，适配度极高。组态的画面信息数据本质还是在本地设备存储。不用担心数据泄露问题。
>    2. 缺点：开发成本高。只适用于简易的设备场景。如果使用不同厂商的设备网关还需要定制开发不同画面。性能体验感差（基于组态渲染画面的必然缺陷）
>    3. 应用场景: 小微项目，一旦设备超过100的量级。其开发成本会大大提高。过500量级的设备量基本可以完全抛弃该模式。
> 2. 基于云端SCADA的设备管理云平台。
>    1. 优缺点基本等同于基于本地SCADA的管理平台。只是所有数据都需要经过云端。存在云端厂商泄露使用数据的风险。
> 3. 基于产品化的云端设备管理平台。
>    1. 优点：只需要买了厂商的设备，即通过扫码、管理平台等方式将设备快速添加进入平台。简单快捷无开发成本。
>    2. 缺点：不够灵活，所有的页面样式固定。本质上是运用产品化量化贩卖降低了单价。一般也不会接受定制化需求。
>    3. 应用场景: 小、中、大型项目。
> 4. 基于产品化的本地化设备管理平台。常见案例：工业MES管理系统
>    1. 优点: 同云端。同时数据本地化存储，数据安全度高。正常会接受高度定制化的软件需求。灵活度高，往往会接受跨厂商设备的接入。
>    2. 缺点: 同云端。同时由于本地化部署，项目需要额外支出相应的服务器运维费用。定制化费用往往高于原始产品的售价。
>    3. 应用场景: 小、中、大型项目。基于不用的项目规模服务器费用也会存在极大价格差。

### 二、OMES系统介绍
由背景产生了以下疑问:
> 1. 所有厂商的设备云平台都是基于自身产品的。我们是否可以跨厂商使用。
> 2. 工业化场景往往需要工艺画面，而纯粹的软件开发过于复杂。那是否可以结合SCADA软件组态结合使用。
> 3. 针对于现在市场项目信息化的场景，要提供一个微信小程序、APP、网页端的多端管理应用。
> 4. 项目场景的定制化，往往需要一些不同数据图表的呈现。

为了解决以上的疑问。OMES，一款基于产品化的本地化设备管理平台被设计了出来。系统的主要功能围绕以下几块：
> 1. 多场景的管理模式。
>    1. 不同场景管理不同设备。
>    2. 根据场景的动态化数据采集适配，自动生成对应的采集报表。
>    3. 与不用厂商SCADA组态工艺画面的集成。
> 2. 配置化的设备管理。
>    1. 设备基础状态：运行、在线、报警的集成, 实时在线展示、历史变化采集。运行趋势图表。
>    2. 设备动态化属性采集配置集成。实时属性数据管理、历史数据采集、属性变化折线图。
>    3. 设备GIS在线展示
> 3. 用户权限的管理
>    1. 手机端的场景权限分配
>    2. 管理端的管理权限分配
> 4. 报警消息体系
>    1. 手机端应用接收设备报警
>    2. 系统消息通知

### 三、OMES功能展示
#### OMES-ADMIN管理平台 
###### 首页数据视图
>![admin-overview.png](admin-overview.png)
![admin-overview2.png](admin-overview2.png)

###### 设备-实时在线卡片
>![admin-equip.png](admin-equip.png)
![admin-equip-2.png](admin-equip-2.png)
![admin-equip-3.png](admin-equip-3.png)
![admin-equip-4.png](admin-equip-4.png)

###### 设备-动态化数据采集配置
>![img.png](config-equip.png)

###### 设备-GIS在线展示
>![img.png](set-equip.png)
![admin-gis-1.png](admin-gis-1.png)
![admin-gis-2.png](admin-gis-2.png)

###### 设备-设备运行态数据采集
>![report-run.png](report-run.png)
![report-online.png](report-online.png)
![report-alarm.png](report-alarm.png)

###### 场景-配置化组态画面
> ![img.png](scada-workshop.png)
![admin-scada.png](admin-scada.png)

###### 场景-动态化数据采集
>![img.png](config-workshop.png)
![workshop-report-2.png](workshop-report-2.png)

###### 权限-账户角色
>![img.png](admin-account.png)
![img.png](admin-role.png)
![img.png](admin-role2.png)

###### 消息-系统通知|报警通知
>![img.png](admin-notify.png)

#### OMES智设备（微信小程序/APP）
###### 设备
>![img.png](app-equp1.png)
![img.png](app-equip3.png)
![img.png](app-equip4.png)
![img.png](app-equip5.png)

###### 场景
>![img.png](app-workshop3.png)
![img.png](app-workshop1.png)
![img.png](app-workshop2.png)

###### 消息
>![img.png](app-message1.png)
![img.png](app-message2.png)
![img.png](app-message3.png)

### 联系我
个人开发者，接工业化相关定制开发。
微信: ty434713950