---
title: 白嫖vercel来搭建自己的博客
date: 2024-05-22
updated: 2024-05-22
---

### 缘起

*友人花费亿元稿费请我写了这篇教程，感恩。*

想着很多朋友都有买个服务器部署博客的需要，行走江湖，哪能没有个博客尼，都4202年了，不能玩传统的博客了，买服务器，搭建lnmp,搭建bt面板，都已经过时了，考虑到国内的监管现状，我推荐新时代的博客应该是这样的。

- 免费
- 免费
- 还是免费

经过多方打听，我选择了 *vercel* 做为部署博客的平台，很多人可能没怎么听过这个名字，但要说起 Next.js 相信大家都很熟悉吧，他就是  *vercel*  的项目。

其实除了博客，他还支持很多功能，有兴趣下次白嫖他的时候再给大家介绍。

#### 提前准备

大家记得自行注册个github账号，这次我们打算部署的博客是 Hexo，想要实现的效果是代码一推送，自动CICD，打包发布。

- github账号
- 域名（可选）

#### 准备git仓库

为了方便大家体验，我把本博客的仓库贴上了，大家fork到自己账号下即可体验

blog的git仓库 https://github.com/knightgao/blog

#### 登录 vercel

vercel 登陆的时候选择github登录，对应的权限给好，一切顺利的话会看到这个页面

#### 部署CI/CD

点击添加

![Pasted image 20240522084628](https://cdn.jsdelivr.net/gh/knightgao/public-img-oss/img/Pasted%20image%2020240522084628.png)

会跳出这个页面，选择刚刚的仓库

![Pasted image 20240522084551](https://cdn.jsdelivr.net/gh/knightgao/public-img-oss/img/Pasted%20image%2020240522084551.png)
可以按需进行修改，也可以使用默认的配置，点部署

![Pasted image 20240522084737](https://cdn.jsdelivr.net/gh/knightgao/public-img-oss/img/Pasted%20image%2020240522084737.png)

会迅速进入流水线进行CI/CD

![Pasted image 20240522084843](https://cdn.jsdelivr.net/gh/knightgao/public-img-oss/img/Pasted%20image%2020240522084843.png)
等待一段时间

![Pasted image 20240522084939](https://cdn.jsdelivr.net/gh/knightgao/public-img-oss/img/Pasted%20image%2020240522084939.png)

看到这说明已经完成

#### 自定义域名

如果需要自定义域名

点击设置

![Pasted image 20240522085037](https://cdn.jsdelivr.net/gh/knightgao/public-img-oss/img/Pasted%20image%2020240522085037.png)

可以把你的域名加入，*注意合规*，vercel 的服务器毕竟在国外，如果国内访问的话尽量国内备案，很容易触发反诈铁拳，网站提示有风险的


### 查看结果

访问上面的路径 https://blog-xd5a.vercel.app/

![Pasted image 20240522085551](https://cdn.jsdelivr.net/gh/knightgao/public-img-oss/img/Pasted%20image%2020240522085551.png)

have fun!