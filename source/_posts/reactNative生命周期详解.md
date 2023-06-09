---
title: reactNative生命周期详解
date: 2023-06-09 15:55:40
tags:
---
**React Native组件的生命周期可以分为三个阶段：挂载阶段、更新阶段和卸载阶段。**

# 挂载阶段
```constructor(props)```：组件被创建时调用，可以在这里初始化state对象或绑定成员函数的this指向。

```static getDerivedStateFromProps(props, state)```：每次组件被重新渲染前都会调用，它可以返回一个对象来更新state，或者返回null来表示新props不需要更新state。

```render()```：根据组件的props和state属性，生成虚拟DOM，即React元素。

```componentDidMount()```：组件渲染完成后调用，通常在这里进行一些网络请求、添加订阅等操作。

# 更新阶段
```static getDerivedStateFromProps(props, state)```：和挂载阶段一样，在每次更新前都会被调用。

```shouldComponentUpdate(nextProps, nextState)```：组件更新前会调用这个方法，它返回一个布尔值，表示是否允许更新，默认返回true。

```render()```：同挂载阶段。

```getSnapshotBeforeUpdate()```：在最近一次渲染输出（提交到DOM节点）之前调用，它可以在组件发生变化前捕获一些信息（例如滚动位置），并在 ```componentDidUpdate()```中进行处理。

```componentDidUpdate(prevProps, prevState, snapshot)```：组件更新后调用，通常在这里进行一些DOM操作、网络请求等操作。

# 卸载阶段
```componentWillUnmount()```：组件卸载前调用，通常在这里进行一些清理操作，例如取消网络请求、清除定时器等。