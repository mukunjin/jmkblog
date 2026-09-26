---
title: "在Claude Desktop中体验Dynadot MCP"
date: 2026-09-26T16:01:32+08:00
categories: 计算机
description: "记录在Claude Desktop中配置并体验Dynadot MCP的过程，以及使用AI自动续费域名的实际体验。"
summary: "记录在Claude Desktop中配置并体验Dynadot MCP的过程，以及使用AI自动续费域名的实际体验。"
---
## 前言
早在今年的8月21日，Dynadot就发布了自家的MCP。上周有空体验了一下，感觉还不错，就趁着中秋节放假记录下来。
## 准备
首先去到[Dynadot MCP官方页面](https://www.dynadot.com/zh/domain/mcp)，下滑，找到“如何启用Dynadot MCP”，按照它的指示去[创建API和开启MCP](https://www.dynadot.com/zh/account/domain/setting/api.html)。

需要注意的是，准备完毕后确实需要等待十分钟左右才能进行下一次操作。我在等待期间让Claude尝试了好几次都是401。
## 配置连接器
在刚刚的官方里面继续下滑，选择Claude配置教程。先打开Claude Desktop，进入自定义→连接器。点击+按钮，添加自定义连接器。然后为连接器命名，再输入Dynadot MCP服务器网址:```https://mcp.dynadot.com/mcp```。接着点击“添加”，选择连接器，再点击“连接”。最后在跳转后的Dynadot页面完成授权即可。

配置好后，你可以在Connectors中看到如下界面：
![2026/claude-desktop-connectors-interface.webp](https://img.mukunjin.com/2026/claude-desktop-connectors-interface.webp)
## 使用
在任何聊天窗口，点击+→ 连接器，开启Dynadot Mcp即可使用。

之前往账户里充的美刀还剩一点，就拿来给我的```678913.xyz```测试续费了。

让Claude了解账户情况后，就直接让他续费。

![2026/let-claude-renew-directly.webp](https://img.mukunjin.com/2026/let-claude-renew-directly.webp)

再去Dynadot看一眼。

![2026/678913-xyz-renewal-successful.webp](https://img.mukunjin.com/2026/678913-xyz-renewal-successful.webp)

发现确实续费成功了，感觉挺方便的，一句话的事情。
## 总结
MCP是真好东西啊，一句话就可以执行操作。希望这次A\不会再封我的号了（）

