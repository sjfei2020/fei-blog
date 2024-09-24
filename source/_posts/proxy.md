---
title: 终端如何代理
date: 2023-10-20
updated: 2024-05-22
---


## 本地代理

### PowerShell

```  powershell
$env:http_proxy="http://127.0.0.1:7890"
$env:https_proxy="http://127.0.0.1:7890"
```

### CMD

``` cmd
set http_proxy=http://127.0.0.1:7890
set https_proxy=http://127.0.0.1:7890
```

### MAC

``` mac
export https_proxy=http://127.0.0.1:7890
http_proxy=http://127.0.0.1:7890
all_proxy=socks5://127.0.0.1:7890
```

### 取消代理

直接关闭终端即可

### 全局生效

全局生效的话设置全局的环境变量即可，不推荐新手使用
