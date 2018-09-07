---

layout: post
title: 微服务治理框架的几个点
category: 技术
tags: Architecture
keywords: dubbo micro service

---

## 简介（未完成）

## 三大基本套路

微服务框架是什么

1. 服务发现、路由、监控报警 等细节
2. 分拆业务 的一种方式
3. 将 公共的 技术能力沉淀到 PaaS层，运维不再只是提供 物理机/jvm 虚拟机（服务的访问形式是ip:port），还提供服务发现、负载均衡等能力（服务的访问形式是一个标识）。

服务发现、路由、监控报警 等事情 谁去干

1. 以框架为中心
2. 以PaaS 平台为中心
3. service mesh

左耳朵耗子 提出来的一系列方法论。

## 从0 到 1 实现一个服务治理框架

单体服务 ==> 通过 rpc 分割为微服务  ==> 解决微服务 带来的问题

### 基本能力

1. 序列化协议
2. 服务注册和发现

同步的话，请求 ==> 线程 ==> 获取一个连接 ==> connect 阻塞还好，服务端一直处理不过来 不给响应 阻塞掉 尴尬了。
异步的话，请求 ==> 线程 ==> 获取一个连接 ==> 发请求 ==> 接口调用结束。
异步的问题在于，通过队列缓冲了请求，若是客户端请求的速度远超服务端响应的速度， 队列会增长，会oom

### 附加能力


## 路由

Router
请求路由器，由服务端配置，在客户端工作，帮助调用方找到合适的被调用实例。

## 集群


假设一个服务有3个实例ABC，称为集群cluster，客户端去调用A

1. failfast，A 实例故障，则调用失败
2. failover，A 实例故障，则重试BC
3. failsafe，A 实例故障，返回实现配置的默认值

## Filter

请求过滤器，扩展接口，允许在调用链上加入自定义的逻辑。

![](/public/upload/architecture/service_manage_1.png)
