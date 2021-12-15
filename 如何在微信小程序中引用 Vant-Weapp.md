## 如何在微信小程序中引用 Vant-Weapp

### 1. 什么是 Vant-Weapp

这是一个轻量可靠的小程序 UI 组件库。可以通过 npm 方式进行引入。

### 2. 通过 npm 方式引入

#### 2.1 查看官网

查看官网文档，文档里就有详细的说明。官网地址：[https://vant-contrib.gitee.io/vant-weapp](https://vant-contrib.gitee.io/vant-weapp)

#### 2.2 需要注意的地方

1. 如果项目使用云开发，现在默认的项目结构是这样子的

```
./
 |---cloudfunctions/
 |		|---yourcloudfunction/
 |---miniprogram/
 |		|---components/
 |		|---pages/
 |		|---images/
 |		|---app.json
 |		|---app.wxss
 |		|---app.js
 |---project.config.json
 |---project.config.private.json
```

使用 npm 安装组件库时应该要让终端走到 `./` 目录下。

2. 在实际构建时，要将 `app.json` 中 `style` 属性下的以 `lazyCoding` 开头的属性也要去掉，否则很多样式将会无法应用。
3. 官方文档快速上手中的 `miniprogramNpmDistDir` 的讨论，在新版本（截至 2021-12-16，version 2.21.0）中实测不需要按照注解中改成 `./`

### 3. 如何使用

多看文档，多练习，感觉文档写的还是很不错的。