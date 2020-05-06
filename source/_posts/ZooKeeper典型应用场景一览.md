---
title: ZooKeeper典型应用场景一览
copyright: true
date: 2020-05-06 20:56:20
tags: [学习]
---

ZooKeeper是一个高可用的分布式数据管理与系统协调框架。基于对Paxos算法的实现，是该框架保证了分布式环境中数据的强一致性，也正是基于这样的特性，使得ZooKeeper解决很多分布式问题。

值得注意的是，ZK并非天生就是为了这些应用场景设计的，都是后来众多开发者根据其框架的特性，利用其提供的一系列API接口（或者称为原语集），摸索出来的典型使用方法。

<!--more-->

![](https://raw.githubusercontent.com/dengweiqiang/BlogImages/master/ZooKeeper典型应用场景一览/ZooKeeper典型应用场景一览_阿里中间件团队博客.png)
