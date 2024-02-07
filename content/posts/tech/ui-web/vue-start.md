---
title: "Vue入门"
date: 2023-06-07T15:05:28+08:00
draft: true
---

参考文档：https://v2.cn.vuejs.org/v2/guide/instance.html

## VUE实例
### 一个VUE实例

一个 Vue 应用由一个通过 new Vue 创建的根 Vue 实例，以及可选的嵌套的、可复用的组件树组成。

### VUE的数据与方法
vue实例被创建的时候，data中已property加入到vue的响应式系统中，“响应式的”。
Object.freeze()会阻止修改property。

vue实例还有实例property和方法，带有前缀$
eg：
```
// $watch 是一个实例方法
vm.$watch('a', function (newValue, oldValue) {
  // 这个回调将在 `vm.a` 改变后调用
})
```

### 实例声明周期钩子
每个 Vue 实例在被创建时都要经过一系列的初始化过程——例如，需要设置数据监听、编译模板、将实例挂载到 DOM 并在数据变化时更新 DOM 等



## 模板语法
### 插值
{{}}插值

### 指令
指令的职责是，当表达式的值改变时，将其产生的连带影响，响应式地作用于 DOM

v-on @ 监听DOM事件

v-bind : 响应式的更新HTML attribute

.prevent可修饰指令

## 计算属性和侦听器
### 计算属性
计算属性缓存替代方法、侦听属性watch。
计算属性默认只有getter，可以设置setter。

侦听器watch也有用的地方：等待一个异步操作


## Class 与 Style 绑定
v-bind:class
v-bind:style

## 条件渲染
v-if
v-else
### \<template\>元素上使用 v-if 条件渲染分组
### 用 key 管理可复用的元素

## 列表渲染
v-for

## 事件处理

## 表单输入绑定
v-model
<input v-model="message" placeholder="edit me">
## 组件基础