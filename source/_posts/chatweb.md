---
title: 部署自己的chatGPT  
date: 2023-05-24
updated: 2023-05-24
---


### 1、拉取代码

```shell
git clone https://github.com/Kerwin1202/chatgpt-web.git
```

### 2、修改配置文件

```.env
// .env 文件
# Glob API URL 不用修改
VITE_GLOB_API_URL=/api
// 后端的url地址，本地与部署应该不同
VITE_APP_API_BASE_URL=http://127.0.0.1:3002/
# Whether long replies are supported, which may result in higher API fees
VITE_GLOB_OPEN_LONG_REPLY=true
# When you want to use PWA
VITE_GLOB_APP_PWA=false
```

```.env
// service\.env.example
// 复制一份新建为.env

# OpenAI API Key - https://platform.openai.com/overview
# 最重要的apikey
OPENAI_API_KEY=sk-XXXXXX

# change this to an `accessToken` extracted from the ChatGPT site's `https://chat.openai.com/api/auth/session` response
# 白嫖chatGPT的token,免费但不稳定
OPENAI_ACCESS_TOKEN=
# OpenAI API Base URL - https://api.openai.com
# 可以设置你做的反代地址，网络情况允许的时候也可以直接用官方的
OPENAI_API_BASE_URL=
# ChatGPTAPI 或者 ChatGPTUnofficialProxyAPI
OPENAI_API_MODEL:
# set `true` to disable OpenAI API debug log
# 设置为true关闭penAI API log
OPENAI_API_DISABLE_DEBUG=
# Reverse Proxy - Available on accessToken
# Default: https://ai.fakeopen.com/api/conversation
# More: https://github.com/transitive-bullshit/chatgpt-api#reverse-proxy
# 不使用OPENAI_ACCESS_TOKEN的话不用设置
API_REVERSE_PROXY=
# timeout
# 超时时间
TIMEOUT_MS=100000
# Rate Limit
# 不使用OPENAI_ACCESS_TOKEN的话不用设置
MAX_REQUEST_PER_HOUR=

# Socks Proxy Host 设置代理
SOCKS_PROXY_HOST=
# Socks Proxy Port
SOCKS_PROXY_PORT=
# Socks Proxy Username
SOCKS_PROXY_USERNAME=
# Socks Proxy Password
SOCKS_PROXY_PASSWORD=

# HTTPS PROXY 设置代理
HTTPS_PROXY=

# Title for site
# 网站的名字
SITE_TITLE="ChatGpt Web"

# Databse connection string
# MONGODB_URL=mongodb://chatgpt:xxxx@yourip:port
# MONGODB的连接数据库，注意信息安全
MONGODB_URL=mongodb://chatgpt:xxxx@database:27017

# Secret key for jwt
# If not empty, will need login
# 加密的盐，一定要设置，不设置就没登录，人人白嫖了
AUTH_SECRET_KEY=

# ----- Only valid after setting AUTH_SECRET_KEY begin ----
# Allow anyone register 是否允许每个人注册
REGISTER_ENABLED=false
# Enable register application review 是否需要再审核才激活账号
REGISTER_REVIEW=false
# The site domain, Only for registration account verification
# without end /
# 前端网址，在发送的邮件里用的上
SITE_DOMAIN=http://127.0.0.1:1002

# Allowed Email Providers, If it is empty, any mailbox is allowed
# 允许的邮箱后缀
REGISTER_MAILS=@qq.com,@sina.com,@163.com

# The roon user only email
# 管理员的邮箱
ROOT_USER=

# Password salt
PASSWORD_MD5_SALT=anysalt

# User register email verify
SMTP_HOST=smtp.exmail.qq.com
SMTP_PORT=465
SMTP_TSL=true
SMTP_USERNAME=yourname@example.com
SMTP_PASSWORD=yourpassword

# ----- Only valid after setting AUTH_SECRET_KEY end ----

```

### 3、数据库连接
假设你服务器或者本地已经有了 *MongoDB*  ，并且已经创建完了账号设置好了密码。
如果不太熟练的话为了安全不建议开启MongoDB的远程访问，只允许本地访问或者局域网就可以了，如果在服务器，建议通过 ssh tunnel

```shell
ssh -L 27017:localhost:27017 服务器用户@服务器ip
```
来访问

### 4、本地开发

上面的做完后，正式进入开发

#### 环境要求

```shell
// `node` 需要 `^16 || ^18 || ^19`，可以使用nvm切换版本
node -v

// 确保版本比较新后使用 获得pnpm
corepack enable 

```

#### 安装依赖

需要安装2份依赖，一份是在项目根目录运行，一份是进入 /service 后运行

```shell

pnpm i

```

#### 运行

```shell
// 前端，在根目录下
pnpm dev

// 后端在/service下
pnpm dev

```

#### 调试

第一种

```ts
// 第一种
// 代码中直接
console.log(XXX);
```

第二种

IDE 内置的调试
比如 VSCode JS Debug Terminal
正常打断点就可以了


#### 部署

#### 前端部署

``` shell

// 在根目录下

pnpm run build

```

打包后把dist文件上传服务器就可

#### 后端部署

``` shell

// 先将/service代码上传到要部署的位置
// 在 /service 下
pnpm run start 

// 不使用build的原因，打包出来的dist不是完整的运行文件，其实没差多少
```


### 2024-05-22更新

此部署方式已经过时，如果感兴趣，直接联系我
