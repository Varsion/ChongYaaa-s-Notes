---
title: SaaS多租户架构考虑
tags:
  - Rails SaaS
  - TaTskPad
  - 开发手札
id: '1719'
categories:
  - - Ruby on Rails
  - - SaaS架构、开发
abbrlink: 4f865012
date: 2020-11-20 14:21:07
---

> TaTsk Pad - 协同任务看板
> 
> 最近萌生的一个想法，TaTsk Pad，目前考虑的需求及功能，是一个基于SaaS架构的多租户平台，相关的开发过程也会在本篇文章中持续更新的。  
> 项目将采用 Ruby on Rails(rails 6)进行开发，同时也有考虑后期通过 Flutter 或者 Electron 开发跨平台应用。

## 角色权限访问控制

角色权限设计考虑到 SaaS多租户模式：

*   一个用户有可能同时处于多个组织。
*   单个组织中的用户涉及到 ‘入职’ 和 ‘离职’ 等业务逻辑。
*   。。。

访问控制是针对越权使用资源的防御措施，目的是为了限制访问主体（如用户等） 对访问客体（如数据库资源等）的访问权限。

SaaS生产环境中的访问控制策略一般有三种:

*   自主访问控制（DAC）
*   强制访问控制（MAC）
*   基于角色的访问控制（RBAC）

基于角色的访问控制是目前公认的解决大型企业的统一资源访问控制的有效方法。

RBAC认为权限授权实际上是 Who 、 What 、 How 的问题。在RBAC模型中，Who 、 What 、 How 构成了访问权限三元组，也就是“ Who 对 What(Which) 进行 How 的操作 ”，也就是“ 主体 ”对“ 客体 ”的操作，其中Who——是权限的拥有者或主体（如：User、Role），what——是资源或对象（Resource、Class)。

主要分为：基本模型RBAC0（Core RBAC）、角色分层模型RBAC1（Hierarchal RBAC）、角色限制模型RBAC2（Constraint RBAC）和统一模型RBAC3（Combines RBAC）。