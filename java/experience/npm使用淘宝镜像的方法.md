[TOC]: # "npm使用淘宝镜像的方法"



1. 永久使用

```cmd
npm config set registry https://registry.npm.taobao.org
```

> 这样配置以后都是用的淘宝的镜像下载资源 国内的访问速度会比较快


2. 直接安装使用

```cmd
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

> 以后使用cnpm代替npm

`后话`

> 推荐使用第一种方式 不改变原来的名称 只是换了一个下载资源
