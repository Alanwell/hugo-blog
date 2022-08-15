# hugo blog

#### 介绍
hugo 搭建个人博客

> Hugo 是一个基于Go 语言的框架，可以快速方便的创建自己的博客。

### 安装说明

windows 版本
首先安装 choco 包管理器，需要在管理员权限下运行 cmd

```shell
powershell -NoProfile -ExecutionPolicy unrestricted -Command "iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1'))"
```

```shell
# 设置环境变量
SET PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin
```
使用 choco 安装 hugo
```
# 墙内安装可能较慢
choco install hugo -confirm
```

macos 版本：使用 brew 安装

```shell
brew install hugo
```

linux 版本：使用 snap 安装

```shell
snap install hugo
```


#### 启动说明

```shell
hugo server
```

### 发布说明

```shell
hugo
```