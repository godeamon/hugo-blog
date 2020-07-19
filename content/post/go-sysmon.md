---
title: "Go Sysmon"
date: 2020-07-19T15:14:51+08:00
draft: true
keywords: ["go"]
description: "go"
tags: ["go"]
categories: ["go"]
---

# 1. 简介

很多系统中都有守护进程，Go 中也不例外。Go 的系统监控主要目的就是隔一段时间来检查一下运行时是否进入异常状态。本文会介绍一下 sysmon 的创建以及其在系统中的作用。

关于本文整理了一个大体的概括图（建议保存此图片，以后复习就就不需要再通篇读文章了）：

![](/Go-sysmon.png)



# 2. sysmon 的创建

对于其创建非常简单，在 runtime.main 函数（位于：go/src/runtime/proc.go 文件中）中启动程序时创建。基本流程如下：

* 调用 runtime.newm 函数，创建一个 runtime.m 结构对象。
* runtime.newm 函数再调用 runtime.newm1 函数，然后调用 runtime.newsproc 函数，系统调用 clone 一个线程。
* 此时在创建好的 runtime.m 对象中已经存在了 sysmon 函数，在创建的线程中执行此函数，开始系统监控。
    * 第一步检查是否有死锁。
    * 然后开始无限循环。
* 循环最开始休眠时间为20us。
* 大约1ms后每次休眠时间翻倍（50个循环中都没有唤醒 goroutine，休眠时间翻倍）。
* 最长每次休眠时间10ms。

上面就是一个整体流程，本文没有将所有源码都复制过来，大家可以参考源码来理解上面这一过程。



# 3. sysmon 的作用

系统监控的左右主要如下：

* debug 监控程序
* 强制垃圾回收
    * 本质就是定期检查上一次垃圾回收是什么时间，如果长时间未进行垃圾回收，就强制执行一次。
* 网络轮询
    * 10ms进行网络轮询
    * 检查是否有待执行的 fd
    * 将就绪的 goroutine 假如全局队列
    * 如果存在空闲P，就用这个P来执行任务
* 抢占
    * 可抢占系统调用的 P：
        * 如果你不了解 Go 的 runtime 或者 MPG 模型，那么这部分理解可能困难一些。
        * 我们知道一个 G 是运行的 P 上的，那么当这个 G 进行系统调用时，当前的 P 会怎样呢？是的，这时 P 其实就在等当前的 G，那么假如对于 P 的本地 G 队列还有在等待的 G时，对于这种情况我们肯定是不允许的。那么我们就需要对这个 P 进行抢占了。
        * 在任务量过多时，所有的处理器（P）都在认真工作，此时对于系统调用的 P 就需要抢占了。
        * 除了以上两种情况，假如这次系统调用时间过长，那么也会发起抢占的。
    * 可抢占运行时间过长的 P：
        * 当一个 G 在 P 上运行了很长时间，说明这个 G 开始欺负其他的 G 了，此时就需要对这个运行时间过长的 G 做出惩罚，抢占其 P 并将这个 G 放入全局队列。

此时关于 sysmon 的东西就讲完了，大家可以边看源码遍理解此文章，但是建议对 Go 的 runtime 有一些了解之后再阅读抢占部分的内容。


