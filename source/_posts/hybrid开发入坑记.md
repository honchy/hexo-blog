title: hybrid开发入坑记
date: 2017-10-09 22:14:53
tags:
---

缘起
===

hybrid 是平衡用户体验和开发效率的一种方案。

早期的hybrid类似微信开发的jssdk时期。通过bridge为webiew做功能增强，使得web开发获得native的一些能力。

现在微信小程序的出现，有了一种新的做法和思路，分离UI展现部分和逻辑计算部分。DOM界面的绘制完全在webview中进行而逻辑计算均在javascriptcore中进行。


刚刚入坑hybrid开发，从微信小程序的这个思路出发造轮子。


native和js通信
===

hybrid开发核心的一个基础技术点就是native和js之间的通信。

ios UIWebView和native通信
---

ios WKWebView和native通信
---

ios javascriptcore和native通信
---

android webview和native通信
---

整体设计
===

解决了通信的问题以后，就可以设计整体的信息流，结构组成和时序。




