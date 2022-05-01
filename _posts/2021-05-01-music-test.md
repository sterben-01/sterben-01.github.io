---
title: 太脑瘫了！
date: 2022-05-01 02:00:00 -0500
categories: [随笔]
tags: [生活]
pin: false
author: 01

toc: true
comments: true
typora-root-url: ../../Sterben-01.github.io
math: false
mermaid: true

---

# 折腾了五个小时发现为什么不能部署了 

这里每次部署都会弹出来

```c++
remote: Permission to Sterben-01/Sterben-01.github.io.git denied to github-actions[bot]. 
fatal: unable to access https://github.com/Sterben-01/Sterben-01.github.io/: The requested URL returned error: 403 
Error: Process completed with exit code 128.
```

结果折腾了五个小时偶然间发现我的仓库的Workflow permission 是tm只读的。赶紧改过来了。

音乐测试