---
title: diskmanager修复SD卡、U盘方法
date: 2021-01-24
tags: 踩坑文！
categories: 电脑小技巧
mathjax: true
---

### <center>diskmanager修复SD卡、U盘方法</center>



**1. 已管理员身份运行cmd**

**2. 键入diskpart进入diskpart.exe**

**3. list disk显示本机现在的磁盘信息**

**4. select disk [number you want] 选中当前想要操作的磁盘**

**5. clean 清除磁盘**

**6. creat partition primary 在选中磁盘上创建指定分区**

**7. active 将当前分区标记为活动**

**8. format fs=fat32 quick  以quick方式、文件系统为fat32格式化该分区**

**9. done!**

