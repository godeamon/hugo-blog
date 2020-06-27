---
title: "Go Modules"
date: 2020-03-14T23:06:15+08:00
draft: true
---



# 1. GOPATH

## 1.1. 简介

​	GOPATH 大家肯定都非常熟悉了，想必大家第一次安装 Go 语言环境的时候已经了解了，这里就不多介绍了。但是大家在做项目时，对于 Go 的代码是如何管理的呢？我们很多时候会用到 ``go get `` 命令，主要就是从 github 上将代码下载到本地，而下载的代码就存在本地 GOPATH 的位置。这样我们对于同一个包的多个版本的管理就有问题了。还有很多不方便的地方，这里就不一一列举了。因此有了  go modules 的解决方案。在此之前也有 vendor 的解决方案，今天主要就是和大家介绍一下如何使用 go modules 来管理项目。



# 2. Go modules

## 2.1. 简介

​	modules 是在 Go 1.11 版本中提出来的，go1.12版功能不断改进，再到go1.13版完善优化，目前也是官方推荐的工具。并且也被认为是 GOPATH 的替代方式。也就是说，我们的 Go 代码可以不用在 ``$GOPATH/src`` 下了（不知道大家有没有觉得轻松了）。



## 2.2. 安装

​	modules 的安装非常简单，安装 GO 1.13 版本或者升级到此版本。比此版本低个人不建议使用。