# VPS搭建WordPress：从选配置到上线，按真实需求把服务器装明白

VPS搭建WordPress真正难的地方，通常不是“把 WordPress 上传到服务器”，而是前面几步：服务器怎么买、内存要多少、机房怎么选、系统装什么、Nginx 和 PHP 怎么配、HTTPS 怎么开，以及出了问题到底先查哪里。

好消息是，2026 年这件事已经有一套比较稳定的做法。现在 WordPress 官方推荐的运行环境是 **PHP 8.3 或更高版本、MariaDB 10.11 或更高版本或 MySQL 8.0 或更高版本，并启用 HTTPS**；Nginx 或 Apache 都可以，Nginx 是官方推荐的方案之一。当前 WordPress 最新稳定版本已经到 **7.1.2，发布于 2026 年 9 月 22 日**。

所以，今天再做 VPS 搭 WordPress，不太建议照着几年前“PHP 7.4 + MySQL 5.7 + 老系统”的教程一路抄。能跑，不等于适合作为新站的长期环境。

下面直接按“选 VPS → 配系统 → 安装 WordPress → 做安全与性能 → 判断套餐是否够用”的顺序走。

## 一、VPS 搭建 WordPress 到底需要什么

一个能长期运行的 WordPress VPS，至少可以拆成六层：

1. VPS 本身，提供 CPU、内存、磁盘和网络。

2. Linux 操作系统，推荐使用仍在维护中的发行版。

3. Web 服务器，例如 Nginx。

4. PHP-FPM，负责运行 WordPress 的 PHP 代码。

5. 数据库，使用 MariaDB 或 MySQL。

6. HTTPS、备份和基础安全策略。

WordPress 官方当前给出的现代运行基线非常明确：PHP 8.3+，MariaDB 10.11+ 或 MySQL 8.0+，HTTPS；Nginx 或 Apache 均可。

其中，VPS 最容易选错的是内存。

### 1GB 能不能装 WordPress？

能。

但“能安装”和“适合长期运行”不是一回事。

1GB 内存可以用来做个人博客、测试站、临时站点，尤其是没有控制面板、插件很少、访问量也不高的情况下。问题在于，一旦同时跑 Nginx、PHP-FPM、数据库、缓存、系统服务，再加上 WordPress 后台和插件，内存余量会迅速变小。

如果你第一次自己搭站，我更建议从 **2GB RAM** 起步。4GB 则适合已经确定要装一些插件、WooCommerce，或者准备同时跑更多后台服务的情况。

不要太迷信“核数越多越快”。普通 WordPress 博客经常先碰到的是 PHP 内存、数据库和磁盘 I/O，而不是服务器有多少个 vCore。

## 二、机房比“10Gbps”三个字更值得看

VPS 套餐页上最容易让人眼花的是端口速率。

10Gbps 看起来很漂亮，但对绝大多数 WordPress 网站来说，真正影响访问体验的不是“套餐宣传页上最大端口是多少”，而是访客和服务器之间的网络路径、服务器本身的 CPU/磁盘能力，以及网站有没有缓存。

DMIT 当前有洛杉矶、香港和东京三个主要 Cloud Instance 节点，并提供 Premium、Eyeball 和 Tier 1 三类网络系列。洛杉矶页面还明确区分了 Premium 网络的中国大陆优化路径与 Tier 1 的普通国际优化路线。

如果你的网站用户主要在中国大陆，机房选择就不能只看“美国 VPS”这个大类。

DMIT 官方目前对三类网络的定位也比较直白：

* **Premium Network**：面向中国大陆和亚太访问，使用包括 CN2 GIA 在内的优化路径。

* **Eyeball Network**：在成本和中国大陆住宅网络访问之间做折中。

* **Tier 1 Network**：重点是亚太、北美、欧洲的国际网络连接，不做专门的中国大陆线路优化。

香港节点目前位于 Equinix HK2，官方给出的参考数据是到中国大陆约 15ms 平均延迟、低于 0.1% 的参考丢包；东京 Premium 网络的官方参考延迟约 28ms。官方也特别说明，这些是参考测量值，实际延迟会随运营商、路径和时间变化。

这也是为什么同样是“2 核 4GB”，不同机房和线路，实际 WordPress 体验可能完全不同。

## 三、DMIT 当前套餐怎么读：别只看套餐名字

DMIT 现在的 Cloud Instance 定价页不是简单的一条产品线，而是一个“地区 × 网络 × 硬件平台”的矩阵。

硬件方面，官方目前列出了 **AN5、AN4、AS3** 三个平台：

* AN5：AMD EPYC 9005，Zen 5，DDR5，NVMe Gen5。

* AN4：AMD EPYC 9004，Zen 4。

* AS3：AMD EPYC 7003，Zen 3。

DMIT 自己对三者的定位分别是高性能、平衡和更看重价格。洛杉矶页面尤其说明 AN5 使用 AMD EPYC 9005、DDR5 与 PCIe 5.0 NVMe；AS3 则是成熟的 AMD EPYC 7003 平台。

对于 WordPress 来说，这意味着：

**你不一定需要直接冲最高规格。**

一个 2 vCore / 4GB 或 4 vCore / 4GB 的 VPS，配好 PHP-FPM 和缓存，完全可以拿来做正常内容站；而企业官网、电商站、高并发动态页面，则要开始认真考虑 CPU、RAM、磁盘和数据库余量。

---

## 四、DMIT 当前公开 Cloud Instance 套餐完整对照

下面把目前公开价格页和 Cloud Instance 页面中出现的 Cloud Instance 产品组去重整理。由于官网用地区、网络和硬件筛选器动态展示，同一个规格会在不同筛选条件下重复出现；这里按**实际公开 SKU/套餐族**整理，而不是把同一套餐重复打印几遍。

价格均为 DMIT 页面当前公开美元价格；官网同时提醒价格和产品可能因调整而存在页面更新滞后，因此下单前仍应以结算页实际显示为准。

| 地区 / 网络 / 系列               | 套餐        | 配置                          | 流量 / 端口               |      价格 | 周期 | 购买                                                                |
| -------------------------- | --------- | --------------------------- | --------------------- | ------: | -- | ----------------------------------------------------------------- |
| LAX / Tier 1 / AS3         | WEE       | 1 vCore / 1GB / 20GB SSD    | 1000GB Max            |  $36.90 | 年付 | [👉 查看 WEE 套餐](https://bit.ly/DmiT)             |
| LAX / Tier 1 / AS3         | TINY      | 1 vCore / 1GB / 20GB SSD    | 2000GB Max            |   $6.90 | 月付 | [👉 查看 TINY 套餐](https://bit.ly/DmiT)            |
| LAX / Tier 1 / AS3         | STARTER   | 2 vCore / 2GB / 40GB SSD    | 4000GB Max            |  $12.90 | 月付 | [👉 查看 STARTER 套餐](https://bit.ly/DmiT)         |
| LAX / Tier 1 / AS3         | MINI      | 2 vCore / 4GB / 80GB SSD    | 8000GB Max            |  $21.90 | 月付 | [👉 查看 MINI 套餐](https://bit.ly/DmiT)            |
| LAX / Tier 1 / AS3         | MICRO     | 4 vCore / 4GB / 120GB SSD   | 16000GB Max           |  $32.90 | 月付 | [👉 查看 MICRO 套餐](https://bit.ly/DmiT)           |
| LAX / Tier 1 / AN5 Volume  | V2C2G     | 2 vCore / 2GB / 40GB SSD    | 5000GB Max / 10Gbps   |  $14.90 | 月付 | [👉 查看 V2C2G](https://bit.ly/DmiT)              |
| LAX / Tier 1 / AN5 Volume  | V2C4G     | 2 vCore / 4GB / 80GB SSD    | 10000GB Max / 10Gbps  |  $23.90 | 月付 | [👉 查看 V2C4G](https://bit.ly/DmiT)              |
| LAX / Tier 1 / AN5 Volume  | V4C4G     | 4 vCore / 4GB / 120GB SSD   | 20000GB Max / 10Gbps  |  $36.90 | 月付 | [👉 查看 V4C4G](https://bit.ly/DmiT)              |
| LAX / Tier 1 / AN5 Volume  | V4C8G     | 4 vCore / 8GB / 160GB SSD   | 40000GB Max / 10Gbps  |  $52.90 | 月付 | [👉 查看 V4C8G](https://bit.ly/DmiT)              |
| LAX / Tier 1 / AN5 Volume  | V8C16G    | 8 vCore / 16GB / 240GB SSD  | 80000GB Max / 10Gbps  | $119.90 | 月付 | [👉 查看 V8C16G](https://bit.ly/DmiT)             |
| LAX / Tier 1 / AN5 Volume  | V12C24G   | 12 vCore / 24GB / 320GB SSD | 160000GB Max / 10Gbps | $199.90 | 月付 | [👉 查看 V12C24G](https://bit.ly/DmiT)            |
| LAX / Tier 1 / AN5 General | G2C4G     | 2 vCore / 4GB / 80GB SSD    | 4000GB Max / 10Gbps   |  $16.90 | 月付 | [👉 查看 G2C4G](https://bit.ly/DmiT)              |
| LAX / Tier 1 / AN5 General | G4C8G     | 4 vCore / 8GB / 160GB SSD   | 8000GB Max / 10Gbps   |  $36.90 | 月付 | [👉 查看 G4C8G](https://bit.ly/DmiT)              |
| LAX / Tier 1 / AN5 General | G8C16G    | 8 vCore / 16GB / 320GB SSD  | 12000GB Max / 10Gbps  |  $79.90 | 月付 | [👉 查看 G8C16G](https://bit.ly/DmiT)             |
| LAX / Tier 1 / AN5 General | G12C24G   | 12 vCore / 24GB / 480GB SSD | 240000GB Max / 10Gbps | $119.90 | 月付 | [👉 查看 G12C24G](https://bit.ly/DmiT)            |
| LAX / Tier 1 / AN5 General | G16C32G   | 16 vCore / 32GB / 640GB SSD | 320000GB Max / 10Gbps | $199.90 | 月付 | [👉 查看 G16C32G](https://bit.ly/DmiT)            |
| LAX / Pro / AN5            | MINI      | 4 vCore / 4GB / 80GB SSD    | 5000GB / 10Gbps       |  $79.90 | 月付 | [👉 查看 LAX Pro MINI](https://bit.ly/DmiT)       |
| LAX / Pro / AN5            | MICRO     | 4 vCore / 4GB / 160GB SSD   | 7000GB / 10Gbps       | $110.90 | 月付 | [👉 查看 LAX Pro MICRO](https://bit.ly/DmiT)      |
| LAX / Pro / AN5            | MEDIUM    | 6 vCore / 8GB / 160GB SSD   | 15000GB / 10Gbps      | $289.90 | 月付 | [👉 查看 LAX Pro MEDIUM](https://bit.ly/DmiT)     |
| LAX / Eyeball / AN5        | MINI      | 4 vCore / 4GB / 80GB SSD    | 10000GB / 10Gbps      |  $79.90 | 月付 | [👉 查看 LAX Eyeball MINI](https://bit.ly/DmiT)   |
| LAX / Eyeball / AN5        | MICRO     | 4 vCore / 4GB / 160GB SSD   | 14000GB / 10Gbps      | $110.90 | 月付 | [👉 查看 LAX Eyeball MICRO](https://bit.ly/DmiT)  |
| LAX / Eyeball / AN5        | MEDIUM    | 6 vCore / 8GB / 160GB SSD   | 30000GB / 10Gbps      | $289.90 | 月付 | [👉 查看 LAX Eyeball MEDIUM](https://bit.ly/DmiT) |
| HKG / Pro / AS3            | STARTER   | 1 vCore / 2GB / 40GB SSD    | 1000GB / 1Gbps        |  $79.90 | 月付 | [👉 查看 HKG Pro STARTER](https://bit.ly/DmiT)    |
| HKG / Pro / AS3            | MINI      | 2 vCore / 4GB / 60GB SSD    | 1500GB / 1Gbps        | $126.90 | 月付 | [👉 查看 HKG Pro MINI](https://bit.ly/DmiT)       |
| HKG / Pro / AS3            | MICRO     | 4 vCore / 4GB / 80GB SSD    | 2000GB / 1Gbps        | $179.90 | 月付 | [👉 查看 HKG Pro MICRO](https://bit.ly/DmiT)      |
| HKG / Eyeball / AS3        | STARTERv2 | 1 vCore / 2GB / 40GB SSD    | 2000GB / 2Gbps        |  $59.90 | 月付 | [👉 查看 HKG EB STARTERv2](https://bit.ly/DmiT)   |
| HKG / Eyeball / AS3        | MINIv2    | 2 vCore / 2GB / 60GB SSD    | 3000GB / 2Gbps        |  $89.90 | 月付 | [👉 查看 HKG EB MINIv2](https://bit.ly/DmiT)      |
| HKG / Eyeball / AS3        | MICROv2   | 4 vCore / 4GB / 80GB SSD    | 4000GB / 4Gbps        | $129.90 | 月付 | [👉 查看 HKG EB MICROv2](https://bit.ly/DmiT)     |
| HKG / Tier 1 / AS3         | STARTER   | 1 vCore / 2GB / 40GB SSD    | 4000GB Max            |  $12.90 | 月付 | [👉 查看 HKG T1 STARTER](https://bit.ly/DmiT)     |
| HKG / Tier 1 / AS3         | MINI      | 2 vCore / 2GB / 60GB SSD    | 8000GB Max            |  $21.90 | 月付 | [👉 查看 HKG T1 MINI](https://bit.ly/DmiT)        |
| HKG / Tier 1 / AS3         | MICRO     | 4 vCore / 4GB / 80GB SSD    | 16000GB Max           |  $32.90 | 月付 | [👉 查看 HKG T1 MICRO](https://bit.ly/DmiT)       |
| HKG / Tier 1 / AS3         | WEE       | 1 vCore / 1GB / 20GB SSD    | 1000GB Max            |  $36.90 | 年付 | [👉 查看 HKG T1 WEE](https://bit.ly/DmiT)         |
| HKG / Tier 1 / AS3         | TINY      | 1 vCore / 1GB / 20GB SSD    | 2000GB Max            |   $6.90 | 月付 | [👉 查看 HKG T1 TINY](https://bit.ly/DmiT)        |
| HKG / Tier 1 / AS3         | MEDIUM    | 4 vCore / 8GB / 160GB SSD   | 32000GB Max           |  $49.90 | 月付 | [👉 查看 HKG T1 MEDIUM](https://bit.ly/DmiT)      |
| HKG / Tier 1 / AS3         | LARGE     | 8 vCore / 16GB / 320GB SSD  | 64000GB Max           |  $99.90 | 月付 | [👉 查看 HKG T1 LARGE](https://bit.ly/DmiT)       |
| HKG / Tier 1 / AS3         | GIANT     | 8 vCore / 24GB / 640GB SSD  | 128000GB Max          | $199.90 | 月付 | [👉 查看 HKG T1 GIANT](https://bit.ly/DmiT)       |
| TYO / Pro / AS3            | STARTER   | 1 vCore / 2GB / 40GB SSD    | 1000GB / 1Gbps        |  $45.90 | 月付 | [👉 查看 TYO Pro STARTER](https://bit.ly/DmiT)    |
| TYO / Pro / AS3            | MINI      | 2 vCore / 4GB / 60GB SSD    | 2000GB / 1Gbps        |  $89.90 | 月付 | [👉 查看 TYO Pro MINI](https://bit.ly/DmiT)       |
| TYO / Pro / AS3            | MICRO     | 4 vCore / 4GB / 80GB SSD    | 4000GB / 1Gbps        | $189.90 | 月付 | [👉 查看 TYO Pro MICRO](https://bit.ly/DmiT)      |
| TYO / Tier 1 / AS3         | TINY      | 1 vCore / 1GB / 20GB SSD    | 500GB / 1Gbps         |  $21.90 | 月付 | [👉 查看 TYO T1 TINY](https://bit.ly/DmiT)        |
| TYO / Tier 1 / AS3         | STARTER   | 1 vCore / 2GB / 40GB SSD    | 1000GB / 1Gbps        |  $45.90 | 月付 | [👉 查看 TYO T1 STARTER](https://bit.ly/DmiT)     |
| TYO / Tier 1 / AS3         | MINI      | 2 vCore / 4GB / 60GB SSD    | 2000GB / 1Gbps        |  $89.90 | 月付 | [👉 查看 TYO T1 MINI](https://bit.ly/DmiT)        |
| TYO / Tier 1 / AS3         | MICRO     | 4 vCore / 4GB / 80GB SSD    | 4000GB / 1Gbps        | $189.90 | 月付 | [👉 查看 TYO T1 MICRO](https://bit.ly/DmiT)       |
| TYO / Tier 1 / AS3         | MEDIUM    | 4 vCore / 8GB / 160GB SSD   | 6000GB / 1Gbps        | $320.90 | 月付 | [👉 查看 TYO T1 MEDIUM](https://bit.ly/DmiT)      |
| TYO / Tier 1 / AS3         | LARGE     | 8 vCore / 16GB / 320GB SSD  | 8000GB / 1Gbps        | $429.90 | 月付 | [👉 查看 TYO T1 LARGE](https://bit.ly/DmiT)       |
| TYO / Tier 1 / AS3         | GIANT     | 8 vCore / 24GB / 640GB SSD  | 15000GB / 1Gbps       | $829.90 | 月付 | [👉 查看 TYO T1 GIANT](https://bit.ly/DmiT)       |

以上价格来自当前 DMIT Pricing/Cloud Instance 页面；LAX、HKG、TYO 的具体可用产品会随地区和网络系列不同而变化，HKG 官方页面还明确说明其 AN5 目前只提供 Premium，AS3 则覆盖 Eyeball 和 Tier 1；TYO 官方页面则公开 Premium 与 Tier 1 两类网络。

需要特别指出的是，**本次核验没有找到 DMIT 当前公开且能够直接验证为“全产品通用”的 2026 年优惠码**。网上确实能找到不少第三方页面宣称某些代码正在生效，但这些信息与 DMIT 当前官方 Pricing 页面并没有形成足够强的交叉验证，所以这里不把第三方代码包装成“当前已确认优惠”。下单时应以结算页实际折扣为准。

---

## 五、VPS搭建WordPress，实际操作可以怎么做

假设你已经买好一台 Linux VPS，接下来可以按这个顺序。

### 第一步：先更新系统

新 VPS 不要拿到手就开始装 WordPress。

先通过 SSH 登录服务器，更新软件包：

bash
sudo apt update
sudo apt full-upgrade -y


然后安装基础工具：

bash
sudo apt install -y curl wget unzip ca-certificates ufw


2026 年的新站，更适合使用仍在维护中的 Ubuntu 版本，并把 PHP 和数据库版本保持在 WordPress 当前推荐范围，而不是沿用已经结束生命周期的软件。WordPress 官方也明确提醒，旧 PHP/MySQL 虽然可能继续运行，但已经停止获得官方维护，存在安全风险。

### 第二步：安装 Nginx、MariaDB 和 PHP-FPM

一个常见的生产环境组合就是：

**Ubuntu + Nginx + PHP-FPM + MariaDB + Let's Encrypt**

2026 年近期的 VPS WordPress 教程基本也围绕这套架构展开。不同 Linux 仓库提供的 PHP 小版本可能不同，所以不要机械照抄几年前写死的安装命令。

安装完成后，至少确认：

text
Nginx
PHP-FPM
PHP MySQL 扩展
PHP curl
PHP GD
PHP mbstring
PHP XML
PHP ZIP
MariaDB


WordPress 官方需求页也给出了对应的 PHP、数据库和 HTTPS 基线。

### 第三步：创建 WordPress 专用数据库

不要直接拿数据库 root 账户给 WordPress 用。

应该创建单独的数据库、单独的数据库用户和单独的密码。

这样做的好处很朴素：即使未来某个插件出现权限问题，也不会让 WordPress 应用直接拿着数据库管理员权限到处跑。

### 第四步：配置 Nginx

Nginx 的核心工作只有两件：

一是把域名请求交给 WordPress 文件目录；

二是把 `.php` 请求交给 PHP-FPM。

这里最容易出问题的是 FastCGI socket 路径。PHP 升级后如果 socket 名称发生变化，而 Nginx 配置没一起修改，就会出现首页 502、后台打不开之类的问题。

所以遇到 502，不要第一反应去重装整个 WordPress。

先检查：

bash
sudo systemctl status nginx
sudo systemctl status php*-fpm


再看 Nginx 错误日志。

很多时候，问题只是在 PHP-FPM 没启动或者 Nginx 指向了不存在的 socket。

### 第五步：把域名解析到 VPS

域名至少准备：

text
example.com
www.example.com


然后将 A 记录指向 VPS 的 IPv4 地址。

如果使用 IPv6，再根据实际网络情况添加 AAAA 记录。DMIT 的 Cloud Instance 页面明确提供 SSH Key Authentication、快照、自动备份等能力，但 IP 可用性和具体资源仍会因产品系列而变化。

### 第六步：启用 HTTPS

这一步不要拖。

WordPress 官方把 HTTPS 列为推荐运行环境的一部分。

最常见的方法是使用 Let's Encrypt，然后让 Nginx 处理 443 流量。

完成以后，再检查：

* `http://example.com` 是否跳转 HTTPS
* `https://example.com` 是否正常
* WordPress 后台地址是否也变成 HTTPS
* 图片和 CSS 是否出现 Mixed Content

很多“WordPress 装好了但页面很奇怪”的问题，其实就是 HTTPS 切换没有处理完整。

---

## 六、WordPress 安装完以后，别急着装二十个插件

WordPress 真正开始吃资源，往往不是第一次打开首页的时候，而是安装一堆插件以后。

尤其要注意：

* 页面构建器
* 电商插件
* 大型统计插件
* 图像处理插件
* 实时搜索
* 备份插件
* 安全插件
* 缓存插件

它们都会占用 PHP worker、内存、数据库查询或磁盘空间。

一个 2GB VPS，如果只做文章型博客，和一个 2GB VPS 跑 WooCommerce、页面编辑器、图片处理和实时搜索，负载完全不是一个级别。

这也是为什么“WordPress 要多少内存”永远只有一个模糊答案。

### 一个实用的配置思路

个人博客、作品集、企业介绍站：

**2 vCore + 2GB RAM** 可以作为起点。

内容站、插件较多：

**2–4 vCore + 4GB RAM** 会更宽松。

WooCommerce、动态页面多、图片处理明显：

**4 vCore + 4–8GB RAM** 更合理。

多个 WordPress 网站共用一台服务器：

不要只把几个站的内存直接相加。还要给数据库、PHP-FPM、缓存、系统以及突发请求留余量。

---

## 七、DMIT 的哪些能力对 WordPress 真有意义

DMIT 当前 Cloud Instance 页面给出的功能包括全 root 权限、即时开通、快照、自动备份和 SSH Key Authentication；可部署的 Linux 镜像包括 Ubuntu、Debian、AlmaLinux、Rocky Linux、Fedora、openSUSE、Arch Linux、Alpine 等。

对 WordPress 来说，真正值得关注的是三件事。

### 快照

在你准备：

* 升级 PHP
* 升级大型插件
* 更换 Nginx 配置
* 改数据库
* 调整服务器环境

之前，先做快照。

它的价值不是让网站更快，而是让“改坏了”以后回滚更简单。

### 自动备份

备份应该和快照区分开。

快照更适合服务器级恢复；自动备份更适合长期数据恢复。

如果你经营的是实际业务网站，不能因为服务器提供了快照，就认为 WordPress 数据已经万无一失。至少应该确认备份是否独立保存、恢复流程是否真正可用。

### SSH Key

DMIT 官方 Cloud Instance 页面支持 SSH 公钥认证，并明确建议通过 SSH Key 登录，同时关闭密码认证以加强安全性。

这一步没有任何“高级感”，只是非常实用。

拿到 VPS 后：

text
SSH Key
↓
禁用 SSH 密码登录
↓
限制防火墙端口
↓
只开放必要服务


比安装一堆花哨的安全插件更值得优先做。

---

## 八、如果主要访问者在中国大陆，怎么选 DMIT

这里最容易被营销话术带跑偏。

真正应该先确定的是：

**你的访客在哪里。**

### 访客主要在中国大陆

优先关注 Premium 网络。

DMIT 官方目前明确把 Premium Network 定位成面向中国大陆及亚太用户的优化路线，并使用 CN2 GIA 等网络能力。香港节点尤其强调中国大陆访问延迟和低丢包。

### 访客是中国大陆 + 海外混合

Eyeball 可以进入比较范围。

DMIT 对 Eyeball 的定位本身就是在成本和中国住宅网络访问之间做折中；不过香港当前官方页面特别标记了 Eyeball 仍处于 Beta，网络路线还在调优，不建议把“需要高稳定性的生产站”直接建立在这个描述之上。

### 访客主要在美国、欧洲或者全球

Tier 1 可能更直接。

DMIT 把 Tier 1 定位为国际网络连接，覆盖亚太、北美和欧洲，不提供专门的中国大陆线路优化。

对于一个英语博客、SaaS 官网或者主要面向北美用户的网站，这个区别比“CN2”三个字重要得多。

---

## 九、现在搭站，WordPress 7.1.2 要不要追新？

截至 2026 年 9 月 26 日，WordPress 官方发布档显示最新稳定版本是 **7.1.2**，发布于 **2026 年 9 月 22 日**。7.0 系列同样有持续维护版本，但官方明确说明当前只有最新的 7.1 系列处于安全维护状态。

所以新建站时，不建议主动安装很旧的 WordPress 主版本。

更值得注意的是：

**插件兼容性。**

服务器本身只是基础设施。真正容易把 WordPress 弄崩的，往往是主题、插件、PHP 版本和数据库版本之间的不兼容。

实际部署时，先完成：

text
WordPress
↓
主题
↓
核心插件
↓
缓存
↓
备份
↓
再开始批量导入内容


比“装完所有东西再一起排错”省事很多。

---

## 十、真实评价怎么看：别只看星星

DMIT 的公开评价并不是只有一个方向。

例如 Trustpilot 上可以看到用户对 UDP 连接、网络稳定性以及客服处理方式的负面反馈；另一方面，Reddit 上近期也有用户分享普通美国 VPS 与优化线路在实际使用中的体验差异。不过这些都属于个体经验，不能直接当成所有用户都会遇到的结果。

尤其是 VPS 评价，很容易受到：

* 机房
* 网络系列
* 具体节点
* 时间段
* 本地运营商
* 应用类型
* 是否使用 CDN

影响。

同一家 VPS，有人拿它跑静态博客，有人拿它跑实时服务，最后写出来的“体验”可能完全不是一个故事。

因此，评价 DMIT 时，与其问“好不好”，不如问：

**你的 WordPress 用户在哪里，你需要什么线路，你愿意花多少预算，以及你能不能自己维护 Linux。**

---

## 十一、VPS搭建WordPress 最容易踩的几个坑

### 把 VPS 当成托管 WordPress

VPS 给的是服务器。

WordPress 的安装、PHP、数据库、Nginx、SSL、安全更新和备份，都需要你自己管理。

近期的 VPS WordPress 教程普遍也把“服务器维护责任”列为 VPS 与托管 WordPress 之间的核心区别。

### 只看 CPU，不看内存

2 vCore / 2GB 和 2 vCore / 4GB，可能比从 2 核升级到 4 核更适合某些 WordPress 网站。

### 不做备份恢复测试

“我设置了自动备份”不等于“我能恢复网站”。

至少应该实际尝试过一次恢复。

### 用老教程安装旧版本 PHP

现在 WordPress 官方推荐 PHP 8.3+。新服务器没有必要从旧版开始。

### 只开 HTTPS，不处理后台和站内资源

最终结果就是地址栏显示安全，但页面里面仍有 HTTP 资源。

这类问题通常出现在刚迁移、刚换域名或者刚做 HTTPS 的网站上。

### 一开始就上过大的 VPS

空 WordPress 放在 8 核 16GB 机器上，当然不会嫌快，但很可能只是花更多钱买了一堆暂时用不到的资源。

更合理的做法是根据访问量、插件数量和数据库增长逐步升级。

---

## 十二、预算怎么安排更合理

如果你只是第一次做一个博客，不需要一开始就研究几十个套餐。

更实际的是先确定这三个问题：

**访问者主要在哪？**

中国大陆、亚太、北美和欧洲，对网络路线的需求不同。

**网站是什么类型？**

纯内容站、企业站和 WooCommerce 电商站，对资源的需求完全不同。

**你自己管不管服务器？**

如果你不想碰 SSH、Nginx、数据库和安全更新，那么 VPS 便宜的服务器价格并不等于便宜的使用成本。

反过来，如果你愿意自己维护，VPS 的自由度就很高。

DMIT 的 Cloud Instance 目前提供 root 权限、即时部署、快照和自动备份能力，并且能在 LAX、HKG、TYO 之间选择不同网络方向。

对于初次搭站，我会把预算优先放在：

text
合适的机房
+
2GB~4GB RAM
+
SSD/NVMe
+
稳定的网络
+
备份


而不是单纯追求“最大核数”和“10Gbps”。

---

## 十三、一个可以直接照着执行的 WordPress VPS 上线顺序

如果你现在已经准备开工，可以按下面的顺序走：

### 服务器阶段

选择机房和网络系列 → 创建 VPS → 确认公网 IP → 设置 SSH Key。

### 系统阶段

更新系统 → 配置防火墙 → 安装 Nginx → 安装 PHP-FPM → 安装 MariaDB。

### WordPress 阶段

创建数据库 → 配置 Nginx → 上传 WordPress → 浏览器安装 → 设置固定链接。

### HTTPS 阶段

申请 Let's Encrypt → 配置 HTTPS → HTTP 跳转 HTTPS → 检查 Mixed Content。

### 安全阶段

关闭 SSH 密码登录 → 修改默认管理策略 → 最小化开放端口 → 配置自动备份。

### 性能阶段

启用页面缓存 → 优化 PHP-FPM → 配置对象缓存或 Redis（按需要）→ 压缩图片 → 使用 CDN（按访客区域决定）。

做完这些，一个真正意义上的 WordPress VPS 才算搭起来了。

---

## 结语：别把“选 VPS”变成参数收集游戏

VPS搭建WordPress 本身没有想象中复杂。

真正需要判断的，是你的站点需要什么。

如果只是个人博客，2GB 左右的机器通常已经足够起步；如果插件多、流量高或者运行 WooCommerce，再向 4GB、8GB 甚至更高升级。网络方面，则应该先看访客位置，再看 Premium、Eyeball、Tier 1 的差异。

DMIT 当前的产品矩阵比较复杂，但好处也是明显的：LAX、HKG、TYO 都有不同网络方向和硬件平台，可以按照实际访问地区选择，而不是所有站点都被迫使用同一种线路。

如果你准备直接开始部署，最简单的起点通常是：先选一个 **2 vCore / 2GB 或 2 vCore / 4GB** 的方案，安装现代 Linux + Nginx + PHP-FPM + MariaDB，再把 HTTPS、备份和 SSH 安全配置补齐。

不要一开始把服务器当实验室。

先把网站稳定跑起来，再根据真实的 CPU、内存、磁盘和访问数据决定什么时候升级。这样比一上来买一台“看起来很强”的 VPS，更容易把预算花在真正影响 WordPress 的地方。
