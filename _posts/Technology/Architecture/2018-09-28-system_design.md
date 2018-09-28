---

layout: post
title: 系统设计的一些体会
category: 技术
tags: Architecture
keywords: 系统设计

---

## 简介（未完成）

业务设计
工程实现

1. 不可以混淆
1. 业务设计中，匹配可能只是一句话，而在工程实现中则是一个很复杂的项目。


业务设计

1. 解决有什么抽象，设定几个概念术语来描述系统、向别人解释系统
2. 基本理念，比如推送系统，宁愿不推也不要打扰用户


工程实现

1. 如何表示

	* 页面上
	* 面向对象代码上
		* 控制层
		* 服务层
	* 缓存上
	* 数据库中
2. 如何存储

	* 存在哪里
	* 结构 还是 非结构
	* 检索需求
3. 如何生效