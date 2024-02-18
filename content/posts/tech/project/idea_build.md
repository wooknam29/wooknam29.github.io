---
title: "idea编译问题"
date: 2024-02-07T15:05:28+08:00
# draft: true
---

## 问题梳理
1、切换分支/新打开项目，报java: 程序包com.tencent.beacon.plugins.xxxx.xxxx不存在

原因：远程的包没有加载进来

解决方法：

（1）maven那里选择 ignore projects

（2）invalidate caches 全部勾选，然后进来后到pom.xml下Reload projects，发现成功