---
title: go mod 速记
date: 2019-08-02 22:36:10
tags: go
---
> go mod 机制是一种用于摆脱 GOPATH 然后提供一个可以方便再部署的包管理工具。

# 常见命令
* 初始化: go mod init github.com/you/hello
* 获取新依赖包: go get
* 查看所有依赖包的版本: go list -m all
* 更新所有直接和间接依赖到最新版本: go get -u
* 去除所有不需要的依赖: go mod tidy
* 根据目前情况创建一个 vendor: go mod vendor