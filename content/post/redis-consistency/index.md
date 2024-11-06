---
title: "Redis 解决数据一致性问题"
description: "解决在 Java 中 Redis 与 Mysql 数据库一致性问题。" 
tags: ["Redis"]
categories: ["Redis"]
date: 2024-10-20T21:12:22+08:00
image: "redis-consistency-cover.png"
hidden: false
draft: true
---

## 先更新缓存，再更新数据库

<div align="center">
  <img src="redis-consistency-1.png">
</div>

由于网络问题，请求顺序无法保证，可能出现先更新缓存的请求，后更新数据库，而后更新缓存的请求反而更新了数据库，这样就出现了缓存数据为20，数据库数据为10，数据不一致。

## 先更新数据库，再更新缓存

## 先删除缓存，再更新数据库