---
title: 本地导入vue和element-ui
date: 2021-04-17
tag: 计算机配置
categories: 前端学习
top: false
mathjax: true
---



> 经过快两年时间的学习，以及我对computer science 和 software industry的认识，我基本以及决定将「前端」作为自己以后的职业方向。这篇是我第一篇前端文章，以后应该也会写很多吧。加油。



### 0. 选择方向的思考

我认为当今时代的互联网还远没有达到饱和，杞人忧天式的对互联网行业的唱衰是不可取的。我们应该看到还有很多企事业机关、中小微企业还没有真正接入互联网。我认为他们不应该是被互联网格局抛弃的一环。市面上针对中小微企业成熟的、规模大的廉价产品我认为有很大的发展空间，可以支撑起很多中小互联网公司发展。比如中小企业在本地自建商城；企事业、机关的管理系统都是很好的业务场景。

还应该注意到，传统的互联网巨头会将重心放在性能优化上，但随着计算机硬件水平的飞速发展，我认为在中小规模企业使用的系统上性能优化的优先级应该会降低。应该将重点放在商业模式的推广和业务场景和业务逻辑的探索。「农村包围城市」这句话我认为在我国永远不会过时。下沉市场的红利远没有被完全发掘出来。新进的拼多多、头条系都证实了这一点。

以我的认识，随着node技术的发展，前端成长为全栈的可能性逐渐清晰了起来。



### 1. 导入Vue.js

Vue.js 是一个由国人 Evan You（尤大）写的框架，国内使用较多，而且还挺方便的。

Vue.js 有开发版和生产版两个版本，主要区别在于尺寸、速度以及是否有 cmd warnings，导入时可以将 [vue-dev.js](https://cdn.jsdelivr.net/npm/vue/dist/vue.js) 或 [vue-production.js](https://cdn.jsdelivr.net/npm/vue) 下载并导入，或者用脚手架搭建，不过初学时不建议用脚手架，什么时候我能离开初学阶段啊qaq。



### 2. 导入 Element-UI

Element-UI 是饿了么团队的一个UI库，样式好看，可以方便我们写页面。

使用时可以导入 [element.js](https://unpkg.com/element-ui@2.4.11/lib/index.js) [element.css](https://unpkg.com/element-ui@2.15.1/lib/theme-chalk/index.css) 实现本地导入。

此外还需将字体文件 [element-icons.woff](https://unpkg.com/element-ui@2.3.6/lib/theme-chalk/fonts/element-icons.woff) [element-icons.ttf](https://unpkg.com/element-ui@2.3.6/lib/theme-chalk/fonts/element-icons.ttf) 下载并将这四个文件放入一个文件夹下，到时导入后就可本地使用了。



### 3. 导入 mdui

mdui 是遵循 Material Design 规范的 UI 库，导入时需将 [mdui.min.js](https://cdn.jsdelivr.net/npm/mdui@1.0.1/dist/js/mdui.min.js) [mdui.min.css](https://cdn.jsdelivr.net/npm/mdui@1.0.1/dist/css/mdui.min.css) 导入到一个文件夹下即可实现本地使用。