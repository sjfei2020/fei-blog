---
title: Git&GitHub
date: 2024-10-08 17:02:48
categories:
  - 猛男技巧
#updated: 2024-10-08
---


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

###  一、版本控制工具应该具备的功能

*协同修改* ：多人并行不悖的修改服务器的同一个文档。
*数据备份* ：不仅可以保存文件的当前状态，还可以保存文件每次提交的历史状态。
*版本管理* ：提交到git仓库的每一个版本要做到保证没有一个重复的数据，节约空间，提高效率。
*权限控制* ：对团队参与开发的人员进行权限控制；对团队外开发者贡献的代码进行审核（GIT独有）
*历史记录* ：可以保存GIT中提交的每一个历史提交记录、时间、内容、日志、修改人
*分支管理* ：允许开发人员在开发过程中多条生产线同时推动任务，提高效率。

### 二、版本控制简介
####  2.1版本控制
    工程设计领域中使用版本控制管理工程蓝图的设计过程。在 IT 开发过程中也可以
    使用版本控制思想管理代码的版本迭代
    
 #### 2.2 版本控制工具
 

 - 思想 ：版本控制
 - 实现： 版本控制工具

集中式版本控制工具：~~CVS、SVN、VSS~~ 

### 三、GIT概述
官网地址 ：[https://git-scm.com/](https://git-scm.com/)
#### 3.1 git的优势

 1. 大部分操作可以在本地完成，不需要联网
 2. 保证了数据的完整性
 3. git管理中只会添加记录而不是删除数据或者修改数据
 4. 分支操作优势非常大。提高工作效率，快捷流畅。
 5. 完全兼容linux命令

#### 3.2 git的安装
git安装 ：[点击这里](https://blog.csdn.net/huangqqdy/article/details/83032408)
#### 3.3 git的结构
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2654264486a723c115aa193362f6fb9a.png#pic_center)
### 四、GIT命令行操作
打开任意文件夹，右键点击 Git Bash Here，会出现一个git小窗口，我们就在这里完成一系列git操作
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9c0a92d64ba40c25f6f4a9ff2e00ae60.png)
#### 4.1 本地库初始化
命令

```java
git init
```
效果
![注意：.git目录中存放的都是本地库相关的子目录和文件，不要删除也不要任意修改](https://i-blog.csdnimg.cn/blog_migrate/042bd9417ca29deda3b685a0aa1fed55.png)
注意：
 - .git 目录中存放的是本地库相关的子目录和文件，不要删除，也不要胡
乱修改。
#### 4.2 设置签名
形式
- 用户名：tom
- Email 地址：goodMorning@atguigu.com
- 作用：区分不同开发人员的身份
- 辨析：这里设置的签名和登录远程库(代码托管中心)的账号、密码没有任何关
系。
- 命令
- 项目级别/仓库级别：仅在当前本地库范围内有效
-  `git config user.name tom_pro`  
- `git config user.email goodMorning_pro@atguigu.com`
- 信息保存位置：**./.git/config** 文件
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/81c2b74baad7411207d660166b30b228.png)
系统用户级别：登录当前操作系统的用户范围
- `git config --global user.name tom_glb`
- `git config --global goodMorning_pro@atguigu.com`
- 信息保存位置：**~/.gitconfig** 文件
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b55a10dd77ce78af0fd146322ac18a1b.png)
- 级别优先级
- 就近原则：项目级别优先于系统用户级别，二者都有时采用项目级别
的签名
- 如果只有系统用户级别的签名，就以系统用户级别的签名为准
- 二者都没有不允许

#### 4.3 基本操作
  
#####  4.3.1 查看GIT工作区，暂存区状态
```java
git status
```
#####  4.3.2 提交

```java
git commit -m '' [文件名]
```
将刚新增的文件或者修改的文件提交到暂存区。 **m后面的单引号要写备注**
#####  4.3.3 查看历史记录

```java
git log
```
多屏显示控制方式：
空格向下翻页
b 向上翻页
q 退出

```java
git log --pretty=oneline
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/be750b0c38dc9bb624d8efb7768ca88c.png)

```java
git log --oneline
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/eda74ccdb01d1526e7ab8439c88aabff.png)

```java
git reflog
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6eadea4c1a82989e38ce6d4875402516.png)
HEAD@{移动到当前版本需要多少步}
#####  4.3.4 版本前进后退

 基于索引值操作 [推荐]
 ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/92600298ab2dabeb565fea31951279b5.png)

      

     git reset --hard [局部索引值]
     git reset --hard a6ace91
  使用^符号：只能后退

>    git reset --hard HEAD   
>     注：一个^表示后退一步，n 个表示后退 n 步

使用~符号：只能后退

>  git reset --hard HEAD~n
>  注：表示后退 n 步

#####  4.3.5  reset 命令的三个参数对比

 1. -- soft 	参数
   只在本地库移动指针
 2. -- mixed 参数 
   本地库移动指针并且重置暂存区
 3. -- hard参数
   本地库移动hard指针、重置暂存区、重置工作区。

#####  4.3.6  找回删除文件
- 前提：删除的时候已经提交到本地库
- 操作： 

   `git reset -- hard（指针位置)`

 1. 删除操作已经提交到本地库：指针位置指向历史记录
 2.  删除操作尚未提交到本地库：指针位置使用 HEAD

#####  4.3.7  比较文件差异

`git diff [文件名]`  <font color=#999AAA >将工作区中的文件和暂存区进行比较</font>
`git diff [本地库中历史版本] [文件名]`<font color=#999AAA >将工作区中的文件和本地库历史记录比较</font>
<font color='red' > 注：不带文件名比较多个文件</font>

#### 4.4 分支管理
##### 4.4.1 什么是分支
在git版本控制中，有多条线同时推动多个任务，这就是分支。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4fc868acf97c0eecb6046fd1e930dea7.png#pic_center)
##### 4.4.2 分支的好处
- 同时并行推进多个功能开发，提高开发效率
- 各个分支在开发过程中，如果某一个分支开发失败，不会对其他分支有任
何影响。失败的分支删除重新开始即可。
##### 4.4.2 分支的操作
1. 创建分支
`git branch [分支名]`
2. 查看分支
`git branch -v `
3. 切换分支
`git checkout  [分支名]`
4. 合并分支
  第一步：切换到接受修改的分支（被合并，增加新内容）上
git checkout [被合并分支名]
第二步：执行 merge 命令 git merge [有新内容分支名]

解决冲突
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5219fcf928edf09684f17646159505d7.png)
- 第一步：编辑文件，删除特殊符号
- 第二步：把文件修改到满意的程度，保存退出
- 第三步：git add [文件名]
- 第四步：git commit -m "日志信息"
- 注意：提交冲突的文件的时候 此时 commit 一定不能带具体文件名

### 五、拉取远程仓库
#### 5.1 创建远程库地址别名
`git remote -v 查看当前所有远程地址别名`
`git remote add [别名] [远程地址]`
>远程地址： 要拉取GIT上某一个仓库的地址
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8f44cbe8dfe5488ed76eb091c0621c37.png)
#### 5.2 推送 (push)
`git origin [远程地址]`
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8bcf39d5716053989b832ffb99d42041.png)
效果
- 完整的把远程库下载到本地
- 创建 origin 远程地址别名
- 初始化本地库


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/99fedc3303cf2114d04953fdbb6976e4.png)













 

