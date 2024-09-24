---
title: 拯救C盘大行动
date: 2023-07-12
updated: 2023-07-12
---
## 简介

公司电脑C盘空间不够了，但有个机械的D盘还挺大的，有没有办法拯救下C盘再抢救下尼？

## 背景

公司配的是127GB的SSD加上一块911GB的机械硬盘，也不是说不能用，就是时不时清下临时文件也能凑合着用，直到有天我装了个docker ，C盘空间直接变红，仅剩下100MB了,是时候出点狠活了。

## 方案分析

### 找运维申请硬盘（⭐⭐⭐）

方便， 如果你周围有人做过，这个是一般的最优解，一块硬盘现在也就几百块，直接解决问题，完美。我这主要是可能级别不够，比较特殊，此方案不一定可行。

### 删临时文件

删除系统运行过程中的临时文件，好处是方便快捷，缺点是这些东西都是缓存，就算删了，下次再运行又冒出来了，春风吹又生，没什么好一点的方式，此方案一般给小白使用比较合适，立竿见影，没啥不良后果。

### 删应用

清理不常使用的应用程序能节约出来一点空间，但单纯的删除节约出来的空间少的可怜，此方法也不推荐

### 电脑管家（⭐）

电脑管家里有很多实用的小工具，比如软件搬家，大文件查找等，适合小白用户，相对来说还是能解决很多问题的，但我这边可能级别不够，此方案不可行。

### 我的方案（⭐⭐）

软件搬家给了我很大的灵感，直接把东西放到D盘不就能解决C盘空间不够的问题了吗，直接移动文件肯定是不行的，各种绑定的路径要修改肯定是要出问题的。又想到了前端常用的一个工具*nvm*,在C盘建一个快捷方式一样的东西，实际的东西都存在其他地方。

## 开干

都3202年了，*CMD* 也是时候进化成 *PowerShell* ，所以本篇都使用 *PowerShell* 示例。

首先我们明确下目标，我这里给一份我检查的路径，可以参考：

- 包缓存
- C:\\Program Files
- C:\\Program Files (x86)
- C:\\Users\\用户名\\AppData\\Local

如果你拿不准，可以下载一个 TreeSize 来查看各个文件夹的大小占用




![软件截图](https://cdn.jsdelivr.net/gh/knightgao/public-img-oss@main/img/Pasted%20image%2020230712130354.png)

![软件截图](https://cdn.jsdelivr.net/gh/knightgao/public-img-oss@main/img/Pasted%20image%2020230712130535.png)
我们这里用这个npm-cache为例子来演示如何使用


现在C盘的剩余空间是 37.1GB

![c盘空间37.1G](https://cdn.jsdelivr.net/gh/knightgao/public-img-oss@main/img/Pasted%20image%2020230712130856.png)

在我执行到这方法前是100MB 🐕

#### 步骤拆解

- 复制一份到D盘
- 删掉原来的文件夹
- 创建一个符号链接在原来的文件夹到D盘的新文件夹

进入到路径
![c盘路径](https://cdn.jsdelivr.net/gh/knightgao/public-img-oss@main/img/Pasted%20image%2020230712131103.png)


查看下文件夹的大小，确实是5GB多
![文件大小5.78GB](https://cdn.jsdelivr.net/gh/knightgao/public-img-oss@main/img/Pasted%20image%2020230712131950.png)

第一步复制

![文件大小5.78GB](https://cdn.jsdelivr.net/gh/knightgao/public-img-oss@main/img/Pasted%20image%2020230712133456.png)


第二步删除原来的
![删除文件](https://cdn.jsdelivr.net/gh/knightgao/public-img-oss@main/img/Pasted%20image%2020230712133611.png)


第三步

创建目录链接

```PowerShell
New-Item -ItemType Junction -Path "C:\Users\knight.gao\AppData\Local\npm-cache" -Target "D:\Users\knight.gao\AppData\Local\npm-cache"
```
![运行结果](https://cdn.jsdelivr.net/gh/knightgao/public-img-oss@main/img/Pasted%20image%2020230712134228.png)
查看下C盘空间
![最后结果](https://cdn.jsdelivr.net/gh/knightgao/public-img-oss@main/img/Pasted%20image%2020230712134319.png)
别忘了测试下程序的功能，理论来说是没啥影响的

### 更方便的使用

直接写成了脚本，只需要修改originalFolder 与 targetFolder 的路径既可，只需要改脚本的前2行代码。
``` PowerShell
# 定义原始文件夹路径和目标文件夹路径
$originalFolder = "C:\Users\knight.gao\AppData\Local\pnpm-cache"
$targetFolder = "D:\Users\knight.gao\AppData\Local\pnpm-cache"

Write-Host "原始文件夹路径："$originalFolder
Write-Host "目标文件夹路径："$targetFolder

# 创建目标文件夹
New-Item -ItemType Directory -Path $targetFolder -Force

# 复制原始文件夹到目标文件夹
Copy-Item -Path $originalFolder\* -Destination $targetFolder -Recurse

# 删除原始文件夹
Remove-Item -Path $originalFolder -Recurse

# 创建链接
New-Item -ItemType Junction -Path $originalFolder -Target $targetFolder
```

脚本二 带确认版本

```PowerShell
# 定义原始文件夹路径和目标文件夹路径
$originalFolder = "C:\Users\knight.gao\AppData\Local\pnpm-cache"
$targetFolder = "D:\Users\knight.gao\AppData\Local\pnpm-cache"

Write-Host "原始文件夹路径：" $originalFolder
Write-Host "目标文件夹路径：" $targetFolder

# 确认用户是否希望继续操作
$confirmation = Read-Host "该操作将会删除原始文件夹并创建链接，是否继续？(Y/n)"
if ($confirmation -ne "Y") {
    Write-Host "操作已取消"
    exit
}

# 检查原始文件夹是否存在
if (Test-Path $originalFolder) {
    # 创建目标文件夹
    New-Item -ItemType Directory -Path $targetFolder -Force

    # 复制原始文件夹到目标文件夹
    Copy-Item -Path $originalFolder\* -Destination $targetFolder -Recurse

    # 删除原始文件夹
    Remove-Item -Path $originalFolder -Recurse

    # 创建链接
    New-Item -ItemType Junction -Path $originalFolder -Target $targetFolder

    Write-Host "操作已完成"
} else {
    Write-Host "原始文件夹不存在，请检查路径是否正确"
}
```

如此大概就行了


## 注意点

SSD与机械硬盘的差异还是要知道点的，一般来说SSD比较快，机械比较慢，所以推荐进行转移的文件一定不要是经常读取的，比较合适的是下载文件夹，微信的缓存。数据无价，进行操作前一定做好备份。
以上脚本由chatGPT提供。

## 补充

### 下载文件夹设置到D盘

选中下载文件夹右键属性，选中位置，选在D盘的位置也可

![下载设置属性](https://cdn.jsdelivr.net/gh/knightgao/public-img-oss@main/img/Pasted%20image%2020230712154338.png)