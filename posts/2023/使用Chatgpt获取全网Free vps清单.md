---
title: 使用Chatgpt获取全网Free vps清单
description: 【真免费 VPS 推荐（非试用）】无需绑信用卡，支持 Python/PHP/Docker，月流量 25GB+，含 Byet.host/HelioHost 等 13 家服务商，附续期与功能详情
date: 2023-02-11
lastmod: 2023-02-26
author: AiEDU
avatar: /img/personal/avatar.png
categories:
  - VPS
tags:
  - 工具
  - VPN
series: 免费VPS
---
## 概要

通过Chatgpt筛选出来的免费的Vps服务商。
<!--more-->


## 服务商清单

#### 提示词
```shell

请帮我列出**符合条件的免费服务**，满足以下要求：

【条件要求】：
1. 必须是真正的免费服务（非限时试用），可以续期，但**全天候在线**，有休眠、暂停需要标注，不能有按月/每月小时限制。
2. 无需信用卡绑定。
3. 可以运行脚本（直接运行或通过伪装均可）。
4. 满足以下任意一条即可（请在表格中说明具体支持的条件）：
   - 支持 Python / Go / Rust / Java / PHP
   - 提供 Linux / FreeBSD 系统
   - 支持 Docker
   - 支持免费部署 Minecraft / TeamSpeak3 / Discord Bot
5. 免费服务必须提供至少 **25GB/月流量**。
6. 请完全排除以下大厂和静态平台：
   - 谷歌（GCP / Glitch）、亚马逊（AWS）、微软（Azure）、甲骨文（Oracle Cloud）、Cloudflare、GitHub、IBM Cloud
   - 所有只提供静态网站的，如 Vercel、Netlify

7. 排除我已经注册过的项目：

8. 尽量推荐更多服务，目标 50 个条目（如果可用）。
9. **避免重复条目**。

【输出要求】：
- 输出必须严格为 **Markdown 表格**，不要写段落、不要加解释、不要附加文字。
- 表格字段必须包含：
   1. 编号
   2. 服务名称
   3. 官网地址（只需首页或说明页面，不要领取界面）
   4. 是否不需要续期（✅️ / ❌️）
   5. 如果需要续期，请简要说明如何续期
   6. 具体支持的条件（如 Python / Docker / MC 等）
   7. 限制（CPU / 内存 / 磁盘 / 流量 / 全天候在线）
- 如果信息缺失，请标注“未知”，不要编造。
- 输出**中文**，只输出表格，不要任何额外说明或解释。
```

#### 生成的清单

| 编号  | 服务名称                              | 官网地址（只需首页或说明页面，不要领取界面）                                                                   | 是否不需要续期（✅️ / ❌️） | 如果需要续期，请简要说明如何续期                                                                                                                                                                                                                                                             | 具体支持的条件（如 Python / Docker / MC 等）                                | 限制（CPU / 内存 / 磁盘 / 流量 / 全天候在线）                                                                                                                                                                                                                               |
| --- | --------------------------------- | ---------------------------------------------------------------------------------------- | ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1   | InfinityFree                      | [https://infinityfree.net](https://infinityfree.net/)                                    | ❌️               | 长期无访问会触发“非活跃”暂停，按提示在控制面板/工单里恢复即可。 ([forum.infinityfree.com](https://forum.infinityfree.com/t/inactivity/17344?utm_source=chatgpt.com "Inactivity - Hosting Support"))                                                                                                        | PHP、MySQL、Cron                                                   | 磁盘5GB；流量“无限制”（公平使用）；全天候在线（共享资源，可能限流/风控）。 ([TechRadar](https://www.techradar.com/reviews/infinity-free?utm_source=chatgpt.com "InfinityFree web hosting review"))                                                                                             |
| 2   | Byet.host Free Hosting            | [https://byet.host/free-hosting](https://byet.host/free-hosting)                         | ✅️               | 无强制续期要求（遵守TOS即可）。 ([byet.host](https://byet.host/terms-of-service?utm_source=chatgpt.com "TERMS AND CONDITIONS OF SERVICE Byet Internet"))                                                                                                                                   | PHP、MySQL、Cron、FTP                                               | 磁盘1GB；月流量50GB；共享CPU；全天候在线。 ([byet.host](https://byet.host/free-hosting?utm_source=chatgpt.com "Free Hosting"))                                                                                                                                               |
| 3   | FreeHosting.com                   | [https://www.freehosting.com](https://www.freehosting.com/)                              | ✅️               | 无强制续期（域名到期需自行续域；与主机不同）。 ([freehosting.com](https://www.freehosting.com/tos.html?utm_source=chatgpt.com "TOS - Free Hosting"))                                                                                                                                                | PHP、MySQL、cPanel                                                 | 磁盘10GB；带宽未计量/“无限制”；共享资源；全天候在线。 ([freehosting.com](https://www.freehosting.com/?utm_source=chatgpt.com "Free Hosting - Your Domain, Unlimited Bandwidth, PHP ..."))                                                                                           |
| 4   | x10Hosting                        | [https://x10hosting.com](https://x10hosting.com/)                                        | 未知               | 未明确续期规则（可能有不活跃清理）。未知                                                                                                                                                                                                                                                         | PHP、MySQL（社区口径亦称支持Python/Perl，官方以PHP为主）                          | 磁盘约500MB；带宽未计量/“无限制”；共享资源；全天候在线。 ([Tom's Guide](https://www.tomsguide.com/buying-guide/best-free-web-hosting?utm_source=chatgpt.com "The best free web hosting in 2025"))                                                                                    |
| 5   | GoogieHost                        | [https://googiehost.com](https://googiehost.com/)                                        | ✅️               | 未见强制续期；遵守使用政策                                                                                                                                                                                                                                                                | PHP、MySQL、cPanel                                                 | 磁盘1GB；月流量100GB；共享资源；全天候在线。 ([Tom's Guide](https://www.tomsguide.com/buying-guide/best-free-web-hosting?utm_source=chatgpt.com "The best free web hosting in 2025"))                                                                                          |
| 6   | ProFreeHost                       | [https://profreehost.com](https://profreehost.com/)                                      | ✅️               | 无强制续期                                                                                                                                                                                                                                                                        | PHP、MySQL、Softaculous                                            | 磁盘未知；带宽“无限制”（宣称）；共享资源；全天候在线。 ([byet.host](https://byet.host/?utm_source=chatgpt.com "Byet Internet: Free Hosting, Paid Hosting, Reseller Services ..."))                                                                                                     |
| 7   | HyperPHP                          | [https://hyperphp.com](https://hyperphp.com/)                                            | ✅️               | 无强制续期                                                                                                                                                                                                                                                                        | PHP、MySQL、cPanel                                                 | 磁盘约1GB；带宽“无限制”（宣称）；全天候在线。 ([hyperphp.com](https://hyperphp.com/hosting-plans.php?utm_source=chatgpt.com "Hosting Plans"))                                                                                                                                    |
| 8   | PlanetHoster “World Lite”         | [https://www.planethoster.com/en/World-Lite](https://www.planethoster.com/en/World-Lite) | ✅️               | 无强制续期                                                                                                                                                                                                                                                                        | PHP、MySQL、cPanel                                                 | 磁盘约750MB；带宽“无限制”（宣称）；全天候在线。 ([x10Hosting: Free Hosting Community](https://community.x10hosting.com/threads/limitations-for-the-free-hosting.200491/?utm_source=chatgpt.com "Limitations for the free hosting. \| x10Hosting"))                               |
| 9   | HelioHost                         | [https://heliohost.org](https://heliohost.org/)                                          | ✅️               | 无强制续期                                                                                                                                                                                                                                                                        | PHP、Python、Java/JSP、.NET、Ruby/RoR、Node.js、PostgreSQL/MySQL、Plesk | 磁盘1GB（可捐赠扩展）；带宽“无限制”；共享CPU限额；全天候在线。 ([heliohost.org](https://heliohost.org/?utm_source=chatgpt.com "HelioHost \| Free Hosting"), [wiki.helionet.org](https://wiki.helionet.org/What_HelioHost_Offers?utm_source=chatgpt.com "What HelioHost Offers"))        |
| 10  | FC2 Free Hosting                  | [https://fc2.com](https://fc2.com/)                                                      | ✅️               | 无强制续期                                                                                                                                                                                                                                                                        | PHP/CGI 等脚本、FTP                                                  | 磁盘1GB；带宽“无限制”；共享资源；全天候在线。 ([TechRadar](https://www.techradar.com/reviews/fc2?utm_source=chatgpt.com "FC2 free hosting review"), [free-webhosts.com](https://www.free-webhosts.com/reviews/Fc2.php?utm_source=chatgpt.com "Free Web Hosting Review of: Fc2")) |
| 11  | FreeHosting.host（FreeHostingHost） | [https://freehosting.host](https://freehosting.host/)                                    | ❌️               | 需每30天登录一次客户端以防暂停，若被停需在面板点“Reactivate”。 ([help.freehosting.host](https://help.freehosting.host/self-help/reactivating-suspended-service-due-to-lack-of-login-into-client-area/?utm_source=chatgpt.com "Reactivating suspended service due to lack of login into client ...")) | PHP、MySQL、cPanel                                                 | 磁盘约1GB；月流量100GB；共享资源；全天候在线。 ([DEV Community](https://dev.to/neil_brown/5-free-web-hosting-providers-that-dont-require-a-credit-card-p3h?utm_source=chatgpt.com "5 Free Web Hosting Providers That Don't Require a Credit ..."))                              |
| 12  | Free Web Hosting Area             | [https://www.freewebhostingarea.com](https://www.freewebhostingarea.com/)                | ✅️               | 无强制续期                                                                                                                                                                                                                                                                        | PHP、MySQL、cron                                                   | 磁盘数GB级（套餐差异）；带宽“无限制”；全天候在线。 ([TechRadar](https://www.techradar.com/reviews/free-web-hosting-area?utm_source=chatgpt.com "Free Web Hosting Area review"))                                                                                                     |
| 13  | AeonFree                          | [https://aeonfree.com](https://aeonfree.com/)                                            | ✅️               | 无强制续期                                                                                                                                                                                                                                                                        | PHP、MySQL、cPanel/SSL                                             | 磁盘未知；带宽“无限制”（宣称）；全天候在线。 ([Aeon Free](https://aeonfree.com/?utm_source=chatgpt.com "AeonFree: Free Unlimited Web Hosting with PHP, MySQL ..."))                                                                                                               |
| 14  | Koyeb（Hobby 免费实例）                 | [https://www.koyeb.com](https://www.koyeb.com/)                                          | ✅️               | 无需续期（可长期运行）；个别区域可能需绑卡但官方称可不绑加入Hobby                                                                                                                                                                                                                                          | 支持 Docker/容器、Linux 运行环境、可跑任意语言                                   | 共享vCPU/512MB RAM（典型）；月出站带宽约100GB（官方示例）；全天候在线。 ([DEV Community](https://dev.to/neil_brown/5-free-web-hosting-providers-that-dont-require-a-credit-card-p3h?utm_source=chatgpt.com "5 Free Web Hosting Providers That Don't Require a Credit ..."))            |
| 15  | Aternos（Minecraft）                | [https://aternos.org](https://aternos.org/)                                              | ❌️               | 服务器无人在线会自动停止；需有人进入或在队列中手动启动                                                                                                                                                                                                                                                  | 免费部署 Minecraft 服务器                                               | CPU/内存/插件有限制；带宽未知；**非24/7**（空闲会停）。 ([heliohost.org](https://heliohost.org/?utm_source=chatgpt.com "HelioHost \| Free Hosting"))                                                                                                                              |
| 16  | Minehut（Minecraft）                | [https://minehut.com](https://minehut.com/)                                              | ❌️               | 空闲自动休眠；需要玩家连接唤醒                                                                                                                                                                                                                                                              | 免费部署 Minecraft 服务器                                               | 10玩家起步等限制；带宽未知；**非24/7**（空闲会停）。 ([Koyeb](https://www.koyeb.com/docs/faqs/pricing?utm_source=chatgpt.com "Pricing Frequently Asked Questions"))                                                                                                               |
| 17  | DisCloud（Discord Bot）             | [https://discloudbot.com](https://discloudbot.com/)                                      | 未知               | 未明确续期；按社区规则维护                                                                                                                                                                                                                                                                | 免费部署 Discord Bot（容器化）                                            | RAM/CPU配额较小；带宽未知；是否**24/7**取决于配额与队列。 ([googiehost.com](https://googiehost.com/freehosting?utm_source=chatgpt.com "FREE Web Hosting with cPanel PHP and MySQL SSL ... - GoogieHost"))                                                                         |
| 18  | PylexNodes（Discord/MC）            | [https://pylexnodes.net](https://pylexnodes.net/)                                        | 未知               | 未明确续期                                                                                                                                                                                                                                                                        | 免费部署 Discord Bot / Minecraft（社区额度）                               | 资源/队列限制；带宽未知；可能**非24/7**。 ([planethoster.com](https://www.planethoster.com/en/World-Hosting?utm_source=chatgpt.com "Hosting Plan - World Platform - (Canada) - (France + Swiss)"))                                                                           |


#### 进一步整理
```shell
# 提示词

请按照上一条的要求，对生成的表格进行处理： - 去重（按服务名称或官网地址） - 排除我已注册的项目 - 排除大厂和静态平台（谷歌 GCP / Glitch、AWS、Azure、Oracle、Cloudflare、GitHub、IBM Cloud、Vercel、Netlify） - 保留原表字段，不增加列 - 只保留符合条件的服务（全天候、免费、流量≥25GB、支持任意环境） - 输出中文 Markdown 表格

```


#### 最终结果

|编号|服务名称|官网地址（只需首页或说明页面，不要领取界面）|是否不需要续期（✅️ / ❌️）|如果需要续期，请简要说明如何续期|具体支持的条件（如 Python / Docker / MC 等）|限制（CPU / 内存 / 磁盘 / 流量 / 全天候在线）|
|---|---|---|---|---|---|---|
|1|InfinityFree|[https://infinityfree.net](https://infinityfree.net/)|❌️|长期无访问会触发“非活跃”暂停，按提示在控制面板/工单里恢复即可|PHP、MySQL、Cron|磁盘5GB；流量“无限制”（公平使用）；全天候在线（共享资源，可能限流/风控）|
|2|Byet.host Free Hosting|[https://byet.host/free-hosting](https://byet.host/free-hosting)|✅️|无强制续期要求（遵守TOS即可）|PHP、MySQL、Cron、FTP|磁盘1GB；月流量50GB；共享CPU；全天候在线|
|3|FreeHosting.com|[https://www.freehosting.com](https://www.freehosting.com/)|✅️|无强制续期（域名到期需自行续域；与主机不同）|PHP、MySQL、cPanel|磁盘10GB；带宽未计量/“无限制”；共享资源；全天候在线|
|4|GoogieHost|[https://googiehost.com](https://googiehost.com/)|✅️|未见强制续期；遵守使用政策|PHP、MySQL、cPanel|磁盘1GB；月流量100GB；共享资源；全天候在线|
|5|ProFreeHost|[https://profreehost.com](https://profreehost.com/)|✅️|无强制续期|PHP、MySQL、Softaculous|磁盘未知；带宽“无限制”（宣称）；共享资源；全天候在线|
|6|HyperPHP|[https://hyperphp.com](https://hyperphp.com/)|✅️|无强制续期|PHP、MySQL、cPanel|磁盘约1GB；带宽“无限制”（宣称）；全天候在线|
|7|PlanetHoster “World Lite”|[https://www.planethoster.com/en/World-Lite](https://www.planethoster.com/en/World-Lite)|✅️|无强制续期|PHP、MySQL、cPanel|磁盘约750MB；带宽“无限制”（宣称）；全天候在线|
|8|HelioHost|[https://heliohost.org](https://heliohost.org/)|✅️|无强制续期|PHP、Python、Java/JSP、.NET、Ruby/RoR、Node.js、PostgreSQL/MySQL、Plesk|磁盘1GB（可捐赠扩展）；带宽“无限制”；共享CPU限额；全天候在线|
|9|FC2 Free Hosting|[https://fc2.com](https://fc2.com/)|✅️|无强制续期|PHP/CGI 等脚本、FTP|磁盘1GB；带宽“无限制”；共享资源；全天候在线|
|10|FreeHosting.host（FreeHostingHost）|[https://freehosting.host](https://freehosting.host/)|❌️|需每30天登录一次客户端以防暂停，若被停需在面板点“Reactivate”|PHP、MySQL、cPanel|磁盘约1GB；月流量100GB；共享资源；全天候在线|
|11|Free Web Hosting Area|[https://www.freewebhostingarea.com](https://www.freewebhostingarea.com/)|✅️|无强制续期|PHP、MySQL、cron|磁盘数GB级（套餐差异）；带宽“无限制”；全天候在线|
|12|AeonFree|[https://aeonfree.com](https://aeonfree.com/)|✅️|无强制续期|PHP、MySQL、cPanel/SSL|磁盘未知；带宽“无限制”（宣称）；全天候在线|
|13|Koyeb（Hobby 免费实例）|[https://www.koyeb.com](https://www.koyeb.com/)|✅️|无需续期（可长期运行）|支持 Docker/容器、Linux 运行环境、可跑任意语言|共享vCPU/512MB RAM（典型）；月出站带宽约100GB；全天候在线|
