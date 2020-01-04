---
title: 从Launchpad彻底移除Parallels Windows应用程序图标
tags: [Import-8bdb]
created: '2020-01-04T12:58:44.945Z'
modified: '2020-01-04T16:19:34.610Z'
---

## 从Launchpad彻底移除Parallels Windows应用程序图标

> 原文链接：http://www.jianshu.com/p/d7ff188aff7b

### 背景

安装Parallels虚拟机时，一时手贱勾选了“添加新应用程序到Launchpad”，导致Launchpad中产生大量巨丑无比的Windows应用程序图标，不能忍。

一番尝试，删除Windows虚拟机、卸载Parallels、拖到Dock中在Finder中查看、通过CleanMyMac卸载，均无果而终。WTF，删不掉了吗？

![强迫症切勿勾选这个](http://static.ohcat.xyz/3971994-898b281727a0c41e.webp)

### 解决方法思路：

删除本地图标，清理数据库缓存。步骤：

1. 删除本地图标
	
	- 打开Finder，按下Shift+CMD+G，输入"~/Applications/"，前往。
	- 找到对应的虚拟机文件夹。每个虚拟机有2个相似的文件夹，一个有大量应用程序图标，另一个没有。若只删除Windows应用程序图标，选中前者；若只删除Windows文件夹图标，选中后者；若都删除，选中两者。
	- 删除选中的文件夹。
	- 查看Launchpad中图标是否已删除。若否，则进入下一步。

2. 清理数据库缓存

	- 打开Finder，按下Shift+CMD+G，输入"~/Library/Application Support/Dock/"，前往。
	- 右击 xxx.db 文件 -> 移到废纸篓。
	- 重启Mac。
	- Launchpad终于恢复清爽~

## setting sync 配置记录

**CODE SETTINGS SYNC DOWNLOAD SUMMARY Version: 3.1.2**

GitHub Token: 9244ee722fbf6ea7b83fa19b0796101ce4d057df
GitHub Gist: 00275e61c880e8cb169ac61046cb0998
GitHub Gist Type: Secret

**Restarting Visual Studio Code may be required to apply color and file icon theme.**

Github token 是用来记录登录的，gist 是具体的云端配置文件



## 给项目添加推荐使用的 vscode 插件

在项目根目录添加 .vscode/settings.json 文件。此文件是vscode 读取的工作空间配置，会覆盖用户配置。文件中添加如下内容：

```json
{
  "recommendations": [
    "editorconfig.editorconfig",
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "mikestead.dotenv",
    "mkxml.vscode-filesize",
    "steoates.autoimport"
  ]
}
```

插件名是 [plugin.author].[plugin.name]，可以在插件商店查看
