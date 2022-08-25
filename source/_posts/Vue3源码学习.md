---
title: Vue3源码学习
date: 2022-01-18 10:31:08
tags:
---

# Vue3初始化流程
创建vnode -> 初始化数据 -> proxy -> setup context -> ReactiveEffect -> run


## 重点
- 依赖收集
- path
- proxy