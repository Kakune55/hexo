---
layout: _posts
title: 记一次Mysql异常连接问题的解决
date: 2023-07-20 15:47:40
tags: ["笔记"]
---
## 随便写点儿

困扰了十几天的问题终于解决了  
自己的数据库总是莫名其妙的block客户机的ip show status后发现Aborted_connects值一直在增高,导致到达阈值后被禁止访问 只能重启mysql。  
但是我的客户机访问数据库的配置都没有问题啊，今天晚上仔细盯了一会儿错误值发现这玩意涨的还很规律，似乎是两分钟一次。
两分钟一次不正好是我的服务监控的请求频率吗，原来的时候偷懒，监控数据库就直接tcping 3306，感觉也没什么问题。
  翻了一堆资料突然有一个文章提到了复现连接错误的方法是ping一下数据库端口。  
这一下直接顿悟了，马上去尝试复现一下，试着用cmd tcping了4下数据库端口。  
再一看Aborted_connects好巧不巧就涨了4。

问题找到了

原来 tcping 的数据包对于数据库来说是错误连接 一切都说的通了 调整服务监控方式后问题顺利解决了

心满意足睡觉去了

## 附帮我解决问题的来源：
[MySQL状态变量Aborted_connects与Aborted_clients浅析 - 潇湘隐者 - 博客园 (cnblogs.com)](https://www.cnblogs.com/kerrycode/p/9206787.html)  
十分感谢这篇文章
