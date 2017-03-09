# frp

[![Build Status](https://travis-ci.org/fatedier/frp.svg)](https://travis-ci.org/fatedier/frp)

[README](README.md) | [中文文档](README_zh.md)

frp 是一个高性能的反向代理应用，可以帮助您轻松地进行内网穿透，对外网提供服务，支持 tcp, udp, http, https 等协议类型，并且 web 服务支持根据域名进行路由转发。

## 目录

<!-- vim-markdown-toc GFM -->
* [1、frp 的作用](#1frp-的作用)
* [2、开发状态](#2开发状态)
* [3、架构](#3架构)
* [4、使用方法](#4使用方法)
* [5、配置文件frps.ini、frpc.ini的写法示例](#5配置文件frpsinifrpcini的写法示例)
    * [5.1、配置文件基本结构](#51配置文件基本结构)
    * [5.2、特权模式--建议新手使用](#52特权模式--建议新手使用)
* [6、不同访问方式的配置文件写法](#6不同访问方式的配置文件写法)
    * [通过 ssh 访问公司内网机器](#通过-ssh-访问公司内网机器)
    * [通过自定义域名访问部署于内网的 web 服务](#通过自定义域名访问部署于内网的-web-服务)
    * [转发 DNS 查询请求](#转发-dns-查询请求)
* [功能说明](#功能说明)
    * [Dashboard](#dashboard)
    * [身份验证](#身份验证)
    * [加密与压缩](#加密与压缩)
    * [服务器端热加载配置文件](#服务器端热加载配置文件)
    * [特权模式](#特权模式)
        * [端口白名单](#端口白名单)
    * [连接池](#连接池)
    * [修改 Host Header](#修改-host-header)
    * [通过密码保护你的 web 服务](#通过密码保护你的-web-服务)
    * [自定义二级域名](#自定义二级域名)
    * [URL 路由](#url-路由)
    * [通过 HTTP PROXY 连接 frps](#通过-http-proxy-连接-frps)
* [开发计划](#开发计划)
* [为 frp 做贡献](#为-frp-做贡献)
* [捐助](#捐助)
    * [支付宝扫码捐赠](#支付宝扫码捐赠)
    * [Paypal 捐赠](#paypal-捐赠)
* [贡献者](#贡献者)

<!-- vim-markdown-toc -->

## 1、frp 的作用

* 利用处于内网或防火墙后的机器，对外网环境提供 http 或 https 服务。
* 对于 http 服务支持基于域名的虚拟主机，支持自定义域名绑定，使多个域名可以共用一个80端口。
* 利用处于内网或防火墙后的机器，对外网环境提供 tcp 服务，例如在家里通过 ssh 访问处于公司内网环境内的主机。
* 可查看通过代理的所有 http 请求和响应的详细信息。（待开发）

## 2、开发状态

frp 目前正在前期开发阶段，master 分支用于发布稳定版本，dev 分支用于开发，您可以尝试下载最新的 release 版本进行测试。

**目前的交互协议可能随时改变，不能保证向后兼容，升级新版本时需要注意公告说明。**

## 3、架构

![architecture](/doc/pic/architecture.png)

## 4、使用方法

4.1、程序下载：根据对应的操作系统及架构，从 [Release](https://github.com/fatedier/frp/releases) 页面下载最新版本的程序并解压。

4.2、服务器端设置和运行：将 frps 及 frps.ini 放到有公网 IP 的机器上（即可提供外网访问的服务器如vps），并对frps文件增加执行权限（+x），同时根据自身需求修改frps.ini文件（需与客户端的frpc.ini文件对应匹配，在后续的示例文件中详述）。
通过./frps -c ./frps.ini命令启动服务端

4.3、客户端设置和运行：将 frpc 及 frpc.ini 放到处于内网环境的机器上（即你准备提供frp内网服务的机器，可以是nas、路由器或者linux、windows、Mac的计算机，可以是拟提供http或tcp等服务的本机，也可以是与其处于同一内网的其他机器，并对frpc文件增加执行权限（+x），同时根据自身需求修改frpc.ini文件（需与客户端的frps.ini文件对应匹配，在后续的示例文件中详述））。
通过./frpc -c ./frpc.ini命令启动服务端

【建议：鉴于大多数情况下服务器端和内网客户端的操作系统是不相同的，新用户可考虑先将不同操作系统对应的release版本程序都下载到本地并解压至对应的不同目录，然后再根据本地客户端操作系统和服务器端操作系统的实际情况上传frpc、frps文件到客户端和服务器端。】

## 5、配置文件frps.ini、frpc.ini的写法示例
### 5.1、配置文件基本结构
配置文件通过[]来定义不同的功能板块（http、https、tcp、udp等等），在此[]下书写该功能版块的设置参数。

[common]

[ssh]

[web]

[dns]

[url]

......

1)通用配置，以[common]关键字开头，frps.ini及frpc.ini均需首先申明或定义的内容，包括服务器地址、端口信息、身份验证信息、控制台配置等通用配置信息——注意：[]内文字不能更改。常见配置示例如下（以服务器端frps.ini配置为例，后续有详细介绍）：
```ini
服务器端frps.ini设置示例：
# frps.ini
[common]
bind_port = 7000
#服务器端frp服务的监听端口为7000
privilege_mode = true
#启动特权模式,否则可赋值false（同时将下面一行注释掉）
privilege_token = 1234
#设置特权密码为1234，最好设置为比较复杂的密码
vhost_http_port = 80
#虚拟主机访问端口，若没有在客户端建立网站需求可不填
vhost_https_port = 443
#虚拟主机ssl加密访问端口，若没有在客户端建立加密网站需求可不填
subdomain_host = frp.yumi.com
#你准备用来提供frp服务的域名，可以是主域名也可以是主域名（推荐用"frp.你的域名"方式来提供frp服务，直观明了），但需将其及其子域名泛解析到你的服务器ip上
dashboard_port = 7500
#设置frp服务的控制台访问端口为7500，即用frp.yumi.com:7500可以访问到frp服务的控制台
dashboard_user = admin
dashboard_pwd = password￥%#￥#%￥
#设置登录frp服务控制台的登录用户名和密码，防止被所有人滥用
#dashbord项也是可以不申明的，这样即使是你本人也无法访问控制台了
```
2)各功能版块设置（不同关键字或类型对应不同的代理服务）
```
[ssh]#---代表shell访问功能
type=tcp
...

[web]#---代表文本访问（http）访问功能
type=http
...

[dns]#---转发dns查询
type=udp
...

[url]#---url路由转发服务
type=http
...
```
可以实现shell访问、web网站访问、转发DNS查询、url路由等功能，[]内文字可以更改，但指向同一个服务器地址的名称不能重复，最好能够与你对应的服务意义相近，以便于识别(关键是由此关键字以下的类型设置、本地或远程服务器端端口号、访问域名等的设置来区别到底是哪一类型的服务)。同时，每一类功能在同一个配置文件里都可以设置多个，只要端口号不同（针对ssh等）或域名不同（针对web访问类）即可。

下面即分别介绍其写法。

### 5.2、特权模式--建议新手使用
基于frp的设置方式需要分别在服务器端frps.ini和客户端frpc.ini中对应定义有关转发穿透的参数（如穿透的类型是http、https、udp还是tcp等等，服务器端端口、客户端端口等等），而且必须保持一致性，否则将出错，这对于新手来说一开始可能有些难度。

如果想要避免每次增加代理（穿透）都需要同时操作服务器端和客户端，可以启用特权模式。特权模式被启用后，代理（穿透）的所有配置信息都可以在 frpc.ini 中配置，无需在服务器端做任何操作。启用特权模式只需在frps.ini 中设置启用特权模式并设置 privilege_token，客户端需要配置同样的 privilege_token 就能能使用特权模式创建代理。

```ini
服务器端frps.ini设置示例：
# frps.ini
[common]
bind_port = 7000
#服务器端frp服务的监听端口为7000
privilege_mode = true
#启动特权模式
privilege_token = 1234
#设置特权密码为1234
vhost_http_port = 80
#虚拟主机访问端口，若没有在客户端建立网站需求可不填
vhost_https_port = 443
#虚拟主机ssl加密访问端口，若没有在客户端建立加密网站需求可不填
subdomain_host = frp.yumi.com
#你准备用来提供frp服务的域名，可以是主域名也可以是主域名（推荐用"frp.你的域名"方式来提供frp服务，直观明了），但需将其及其子域名泛解析到你的服务器ip上
dashboard_port = 7500
#设置frp服务的控制台访问端口为7500，即用frp.yumi.com:7500可以访问到frp服务的控制台
dashboard_user = admin
dashboard_pwd = password￥%#￥#%￥
#设置登录frp服务控制台的登录用户名和密码，防止被所有人滥用
#dashbord项也是可以不申明的，这样即使是你本人也无法访问控制台了

客户端frpc.ini设置示例：
# frpc.ini
[common]
server_addr = x.x.x.x
#服务器地址，可以是ip地址，也可是域名
server_port = 7000
#服务器端frp服务的监听端口为7000，与服务器端frps.ini保持一致
privilege_token = 1234
#特权密码为1234，与服务器端frps.ini中设置的保持一致

[ssh]
#定义一个shell代理功能
type=tcp
privilege_mode = true
#申明采用特权模式
local_ip = 127.0.0.1
#定义此shell代理访问127.0.0.1即运行frpc客户端的本机，若将其填写为与本机同一局域网的其它ip如192.168.1.11，则是访问192.168.1.11
local_port = 22
#本地（客户端）提供shell服务的端口为22
remote_port = 6000
#服务器端的端口为6000，如此设置后，则以x.x.x.x:6000方式即可实现对客户端（127.0.0.1或其它IP地址如192.168.1.11对应主机）的shell访问

[web]
#定义一个http代理
type=http
privilege_mode = true
#申明采用特权模式
local_ip = 192.168.1.11
#将访问192.168.1.11主机上建立的网站，该主机应该与运行frpc客户端的主机在同一局域网内
local_port = 80
#本地（客户端）提供http服务的端口为80
custom_domains = web01.yourdomain.com,yourdomain2.com,yourdomain3.com
#通过web01.yourdomain.com,yourdomain2.com,yourdomain3.com来访问192.168.1.11上建立的网站,不同域名之间用","隔开
subdomain=web
#通过web.yumi.com来访问192.168.1.11上建立的网站，一次只能指定一个域名

[web01]
#定义一个http代理
type=http
privilege_mode = true
#申明采用特权模式
local_ip = 127.0.0.1
#将访问本机上建立的网站
local_port = 80
#本地（客户端）提供http服务的端口为80
custom_domains = web04.yourdomain.com,yourdomain4.com,yourdomain5.com
#通过web04.yourdomain.com,yourdomain4.com,yourdomain5.com来访问本机上建立的网站,不同域名之间用","隔开
subdomain=web01
#通过web01.yumi.com来访问本机上建立的网站，一次只能指定一个域名

[web02]
#定义一个https代理
type=https
privilege_mode = true
#申明采用特权模式
local_ip = 127.0.0.1
#将访问本机上建立的网站
local_port = 443
#本地（客户端）提供https服务的端口为443
custom_domains = web04.yourdomain.com,yourdomain4.com,yourdomain5.com
#通过https://web04.yourdomain.com,yourdomain4.com,yourdomain5.com来访问本机上建立的加密网站,不同域名之间用","隔开
subdomain=web01
#通过https://web01.yumi.com来访问本机上建立的加密网站，一次只能指定一个域名

```
### 5.3、普通模式
### 5.4、混合模式
## 6、不同访问方式的配置文件写法
### 6.1、通过 ssh 访问公司内网机器
  ```ini
  # frps.ini
  [common]
  bind_port = 7000
  
  [ssh]
  listen_port = 6000
  auth_token = 123
  ```
  
  ```ini
   # frpc.ini
  [common]
  server_addr = x.x.x.x
  server_port = 7000
  auth_token = 123
  
  [ssh]
  local_ip = 127.0.0.1
  local_port = 22
  ```

 则通过 ssh 访问内网机器用如下命令（假设登录用户名为 test，当然也可以用putty、shell5等工具软件）：

  `ssh -oPort=6000 test@x.x.x.x`

### 6.2、通过自定义域名访问部署于内网的 web 服务

有时想要让其他人通过域名访问或者测试我们在本地搭建的 web 服务，但是由于本地机器没有公网 IP，无法将域名解析到本地的机器，通过 frp 就可以实现这一功能，以下示例为 http 服务，https 服务配置方法相同， vhost_http_port 替换为 vhost_https_port， type 设置为 https 即可。

1. 修改 frps.ini 文件，配置一个名为 web 的 http 反向代理，设置 http 访问端口为 8080，绑定自定义域名 `www.yourdomain.com`：

  ```ini
  # frps.ini
  [common]
  bind_port = 7000
  vhost_http_port = 8080

  [web]
  type = http
  custom_domains = www.yourdomain.com
  auth_token = 123
  ```

2. 启动 frps；

  `./frps -c ./frps.ini`

3. 修改 frpc.ini 文件，设置 frps 所在的服务器的 IP 为 x.x.x.x，local_port 为本地机器上 web 服务对应的端口：

  ```ini
  # frpc.ini
  [common]
  server_addr = x.x.x.x
  server_port = 7000
  auth_token = 123

  [web]
  type = http
  local_port = 80
  ```

4. 启动 frpc：

  `./frpc -c ./frpc.ini`

5. 将 `www.yourdomain.com` 的域名 A 记录解析到 IP `x.x.x.x`，如果服务器已经有对应的域名，也可以将 CNAME 记录解析到服务器原先的域名。

6. 通过浏览器访问 `http://www.yourdomain.com:8080` 即可访问到处于内网机器上的 web 服务。

### 转发 DNS 查询请求

DNS 查询请求通常使用 UDP 协议，frp 支持对内网 UDP 服务的穿透，配置方式和 TCP 基本一致。

1. 修改 frps.ini 文件，配置一个名为 dns 的反向代理：

  ```ini
  # frps.ini
  [common]
  bind_port = 7000
  
  [dns]
  type = udp
  listen_port = 6000
  auth_token = 123
  ```

2. 启动 frps：

  `./frps -c ./frps.ini`

3. 修改 frpc.ini 文件，设置 frps 所在服务器的 IP 为 x.x.x.x，转发到 Google 的 DNS 查询服务器 `8.8.8.8` 的 udp 53 端口：

  ```ini
  # frpc.ini
  [common]
  server_addr = x.x.x.x
  server_port = 7000
  auth_token = 123
  
  [dns]
  type = udp
  local_ip = 8.8.8.8
  local_port = 53
  ```

4. 启动 frpc：

  `./frpc -c ./frpc.ini`

5. 通过 dig 测试 UDP 包转发是否成功，预期会返回 `www.google.com` 域名的解析结果：

  `dig @x.x.x.x -p 6000 www.goolge.com`

## 功能说明

### Dashboard

通过浏览器查看 frp 的状态以及代理统计信息展示。

需要在 frps.ini 中指定 dashboard 服务使用的端口，即可开启此功能：

```ini
[common]
dashboard_port = 7500
# dashboard 用户名密码可选，默认都为 admin
dashboard_user = admin
dashboard_pwd = admin
```

打开浏览器通过 `http://[server_addr]:7500` 访问 dashboard 界面，用户名密码默认为 `admin`。

![dashboard](/doc/pic/dashboard.png)

### 身份验证

出于安全性的考虑，服务器端可以在 frps.ini 中为每一个代理设置一个 auth_token 用于对客户端连接进行身份验证，例如上文中的 [ssh] 和 [web] 两个代理的 auth_token 都为 123。

客户端需要在 frpc.ini 中配置自己的 auth_token，与服务器中的配置一致才能正常运行。

需要注意的是 frpc 所在机器和 frps 所在机器的时间相差不能超过 15 分钟，因为时间戳会被用于加密验证中，防止报文被劫持后被其他人利用。

这个超时时间可以在配置文件中通过 `authentication_timeout` 这个参数来修改，单位为秒，默认值为 900，即 15 分钟。如果修改为 0，则 frps 将不对身份验证报文的时间戳进行超时校验。

### 加密与压缩

这两个功能默认是不开启的，需要在 frpc.ini 中通过配置来为指定的代理启用加密与压缩的功能，无论类型是 tcp, http 还是 https：

```ini
# frpc.ini
[ssh]
type = tcp
listen_port = 6000
auth_token = 123
use_encryption = true
use_gzip = true
```

如果公司内网防火墙对外网访问进行了流量识别与屏蔽，例如禁止了 ssh 协议等，通过设置 `use_encryption = true`，将 frpc 与 frps 之间的通信内容加密传输，将会有效防止流量被拦截。

如果传输的报文长度较长，通过设置 `use_gzip = true` 对传输内容进行压缩，可以有效减小 frpc 与 frps 之间的网络流量，加快流量转发速度，但是会额外消耗一些 cpu 资源。

### 服务器端热加载配置文件

当需要新增一个 frpc 客户端时，为了避免将 frps 重启，可以使用 reload 命令重新加载配置文件。

reload 命令仅能用于修改代理的配置内容，[common] 内的公共配置信息无法修改。

1. 首先需要在 frps.ini 中指定 dashboard_port：

  ```ini
  # frps.ini
  [common]
  bind_port = 7000
  dashboard_port = 7500
  ```

2. 启动 frps：

  `./frps -c ./frps.ini`

3. 修改 frps.ini 增加一个新的代理 [new_ssh]:

  ```ini
  # frps.ini
  [common]
  bind_port = 7000
  dashboard_port = 7500

  [new_ssh]
  listen_port = 6001
  auth_token = 123
  ```

4. 执行 reload 命令，使 frps 重新加载配置文件，实际上是通过 7500 端口发送了一个 http 请求

  `./frps -c ./frps.ini --reload`

5. 之后启动 frpc，[new_ssh] 代理已经可以使用。

### 特权模式

如果想要避免每次增加代理都需要操作服务器端，可以启用特权模式。

特权模式被启用后，代理的所有配置信息都可以在 frpc.ini 中配置，无需在服务器端做任何操作。

1. 在 frps.ini 中设置启用特权模式并设置 privilege_token，客户端需要配置同样的 privilege_token 才能使用特权模式创建代理：

  ```ini
  # frps.ini
  [common]
  bind_port = 7000
  privilege_mode = true
  privilege_token = 1234
  ```

2. 启动 frps：

  `./frps -c ./frps.ini`

3. 在 frpc.ini 配置代理 [ssh]，使用特权模式创建，无需事先在服务器端配置：

  ```ini
  # frpc.ini
  [common]
  server_addr = x.x.x.x
  server_port = 7000
  privilege_token = 1234

  [ssh]
  privilege_mode = true
  local_port = 22
  remote_port = 6000
  ```

  remote_port 即为原先在 frps.ini 的代理中配置的 listen_port 参数，使用特权模式后需要在 frpc 的配置文件中指定。

4. 启动 frpc：

  `./frpc -c ./frpc.ini`

5. 通过 ssh 访问内网机器，假设用户名为 test：

  `ssh -oPort=6000 test@x.x.x.x`

#### 端口白名单

启用特权模式后为了防止端口被滥用，可以手动指定允许哪些端口被使用，在 frps.ini 中通过 privilege_allow_ports 来指定：

```ini
# frps.ini
[common]
privilege_mode = true
privilege_token = 1234
privilege_allow_ports = 2000-3000,3001,3003,4000-50000
```

privilege_allow_ports 可以配置允许使用的某个指定端口或者是一个范围内的所有端口，以 `,` 分隔，指定的范围以 `-` 分隔。

### 连接池

默认情况下，当用户请求建立连接后，frps 才会请求 frpc 主动与后端服务建立一个连接。当为指定的代理启用连接池后，frp 会预先和后端服务建立起指定数量的连接，每次接收到用户请求后，会从连接池中取出一个连接和用户连接关联起来，避免了等待与后端服务建立连接以及 frpc 和 frps 之间传递控制信息的时间。

这一功能比较适合有大量短连接请求时开启。

1. 首先可以在 frps.ini 中设置每个代理可以创建的连接池上限，避免大量资源占用，默认为 100，客户端设置超过此配置后会被调整到当前值：

  ```ini
  # frps.ini
  [common]
  max_pool_count = 50
  ```

2. 在 frpc.ini 中为指定代理启用连接池，指定预创建连接的数量：

  ```ini
  # frpc.ini
  [ssh]
  type = tcp
  local_port = 22
  pool_count = 10
  ```

### 修改 Host Header

通常情况下 frp 不会修改转发的任何数据。但有一些后端服务会根据 http 请求 header 中的 host 字段来展现不同的网站，例如 nginx 的虚拟主机服务，启用 host-header 的修改功能可以动态修改 http 请求中的 host 字段。该功能仅限于 http 类型的代理。

```ini
# frpc.ini
[web]
privilege_mode = true
type = http
local_port = 80
custom_domains = test.yourdomain.com
host_header_rewrite = dev.yourdomain.com
```

原来 http 请求中的 host 字段 `test.yourdomain.com` 转发到后端服务时会被替换为 `dev.yourdomain.com`。

### 通过密码保护你的 web 服务

由于所有客户端共用一个 frps 的 http 服务端口，任何知道你的域名和 url 的人都能访问到你部署在内网的 web 服务，但是在某些场景下需要确保只有限定的用户才能访问。

frp 支持通过 HTTP Basic Auth 来保护你的 web 服务，使用户需要通过用户名和密码才能访问到你的服务。

该功能目前仅限于 http 类型的代理，需要在 frpc 的代理配置中添加用户名和密码的设置。

```ini
# frpc.ini
[web]
privilege_mode = true
type = http
local_port = 80
custom_domains = test.yourdomain.com
http_user = abc
http_pwd = abc
```

通过浏览器访问 `test.yourdomain.com`，需要输入配置的用户名和密码才能访问。

### 自定义二级域名

在多人同时使用一个 frps 时，通过自定义二级域名的方式来使用会更加方便。

通过在 frps 的配置文件中配置 `subdomain_host`，就可以启用该特性。之后在 frpc 的 http、https 类型的代理中可以不配置 `custom_domains`，而是配置一个 `subdomain` 参数。

只需要将 `*.subdomain_host` 解析到 frps 所在服务器。之后用户可以通过 `subdomain` 自行指定自己的 web 服务所需要使用的二级域名，通过 `{subdomain}.{subdomain_host}` 来访问自己的 web 服务。

```ini
# frps.ini
subdomain_host = frps.com
```

将泛域名 `*.frps.com` 解析到 frps 所在服务器的 IP 地址。

```ini
# frpc.ini
[web]
privilege_mode = true
type = http
local_port = 80
subdomain = test
```

frps 和 fprc 都启动成功后，通过 `test.frps.com` 就可以访问到内网的 web 服务。

需要注意的是如果 frps 配置了 `subdomain_host`，则 `custom_domains` 中不能是属于 `subdomain_host` 的子域名或者泛域名。

同一个 http 或 https 类型的代理中 `custom_domains`  和 `subdomain` 可以同时配置。

### URL 路由

frp 支持根据请求的 URL 路径路由转发到不同的后端服务。

通过配置文件中的 `locations` 字段指定一个或多个 proxy 能够匹配的 URL 前缀(目前仅支持最大前缀匹配，之后会考虑正则匹配)。例如指定 `locations = /news`，则所有 URL 以 `/news` 开头的请求都会被转发到这个服务。

```ini
# frpc.ini
[web01]
privilege_mode = true
type = http
local_port = 80
custom_domains = web.yourdomain.com
locations = /

[web02]
privilege_mode = true
type = http
local_port = 81
custom_domains = web.yourdomain.com
locations = /news,/about
```

按照上述的示例配置后，`web.yourdomain.com` 这个域名下所有以 `/news` 以及 `/about` 作为前缀的 URL 请求都会被转发到 web02，其余的请求会被转发到 web01。

### 通过 HTTP PROXY 连接 frps

在只能通过代理访问外网的环境内，frpc 支持通过 HTTP PROXY 和 frps 进行通信。

可以通过设置 `HTTP_PROXY` 系统环境变量或者通过在 frpc 的配置文件中设置 `http_proxy` 参数来使用此功能。

```ini
# frpc.ini
server_addr = x.x.x.x
server_port = 7000
http_proxy = http://user:pwd@192.168.1.128:8080
```

## 开发计划

计划在后续版本中加入的功能与优化，排名不分先后，如果有其他功能建议欢迎在 [issues](https://github.com/fatedier/frp/issues) 中反馈。

* frps 记录 http 请求日志。
* frps 支持直接反向代理，类似 haproxy。
* frpc 支持负载均衡到后端不同服务。
* frpc debug 模式，控制台显示代理状态，类似 ngrok 启动后的界面。
* frpc http 请求及响应信息展示。
* frpc 支持直接作为 webserver 访问指定静态页面。
* frpc 完全控制模式，通过 dashboard 对 frpc 进行在线操作。
* 支持 udp 打洞的方式，提供两边内网机器直接通信，流量不经过服务器转发。

## 为 frp 做贡献

frp 是一个免费且开源的项目，我们欢迎任何人为其开发和进步贡献力量。

* 在使用过程中出现任何问题，可以通过 [issues](https://github.com/fatedier/frp/issues) 来反馈。
* Bug 的修复可以直接提交 Pull Request 到 dev 分支。
* 如果是增加新的功能特性，请先创建一个 issue 并做简单描述以及大致的实现方法，提议被采纳后，就可以创建一个实现新特性的 Pull Request。
* 欢迎对说明文档做出改善，帮助更多的人使用 frp，特别是英文文档。
* 贡献代码请提交 PR 至 dev 分支，master 分支仅用于发布稳定可用版本。
* 如果你有任何其他方面的问题，欢迎反馈至 fatedier@gmail.com 共同交流。

**提醒：和项目相关的问题最好在 [issues](https://github.com/fatedier/frp/issues) 中反馈，这样方便其他有类似问题的人可以快速查找解决方法，并且也避免了我们重复回答一些问题。**

## 捐助

如果您觉得 frp 对你有帮助，欢迎给予我们一定的捐助来维持项目的长期发展。

frp 交流群：606194980 (QQ 群号)

### 支付宝扫码捐赠

![donate-alipay](/doc/pic/donate-alipay.png)

### Paypal 捐赠

海外用户推荐通过 [Paypal](https://www.paypal.me/fatedier) 向我的账户 **fatedier@gmail.com** 进行捐赠。

## 贡献者

* [fatedier](https://github.com/fatedier)
* [Hurricanezwf](https://github.com/Hurricanezwf)
* [Pan Hao](https://github.com/vashstorm)
* [Danping Mao](https://github.com/maodanp)
* [Eric Larssen](https://github.com/ericlarssen)
* [Damon Zhao](https://github.com/se77en)
* [Manfred Touron](https://github.com/moul)
* [xuebing1110](https://github.com/xuebing1110)
* [Anbitioner](https://github.com/bingtianbaihua)
* [LitleCarl](https://github.com/LitleCarl)
