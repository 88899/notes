---
title: 
description: 
date: 
lastmod: 
author: 
avatar: 
categories: 
tags: 
series: 
image: 
draft: "true"
---

Memos Worker是一款基于 Cloudflare 全生态（Workers + D1 + R2 + KV + Pages）构建的无服务器笔记应用，无需购买服务器，免费套餐内即可稳定运行，为你提供 **私有化、零成本、高性能** 的笔记与知识库解决方案。

## 🚀 部署总览

本次部署仅专注 **Cloudflare Worker 方式**，全程涉及 3 个平台，按以下顺序操作（无需跨平台来回切换）：

1. **GitHub 平台**：获取项目代码 + 配置资源绑定信息
2. **Cloudflare 平台**：创建核心资源 + 部署应用 + 配置环境变量
3. **本地环境**（可选）：命令行部署 + 开发调试

## 一、GitHub 平台操作（仅需做 2 件事：拿代码 + 填配置）

### 目标

获取项目代码仓库，并提前配置资源绑定信息，为后续 Cloudflare 部署铺路。

### 前置准备

### 步骤 1：获取项目代码（二选一）

#### 选项 A：Fork 仓库（推荐，方便后续更新）

适合想同步官方新功能、修复的用户，仓库公开不影响安全性（敏感信息通过环境变量配置）。

1. 访问 Memos Worker 官方仓库（假设仓库地址为 `https://github.com/xxx/memos-worker`，替换为实际地址）
2. 点击页面右上角的 **「Fork」** 按钮（绿色 / 灰色，位置在导航栏右侧）
3. 在弹出的页面中，直接点击 **「Create fork」**（无需修改其他设置），等待几秒即可完成

#### 选项 B：使用模板创建私有仓库

适合需要仓库私有化的用户，缺点是无法自动同步后续更新，需手动维护。

1. 访问 Memos Worker 官方仓库
2. 点击页面右上角的 **「Use this template」** → 选择 **「Create a new repository」**
3. 填写仓库名称（如 `memos-worker-my`），选择 **「Private」**（私有），点击 **「Create repository from template」**

### 步骤 2：配置 `wrangler.toml`（关键：绑定后续 Cloudflare 资源）

`wrangler.toml` 是 Cloudflare Worker 的配置文件，需提前填写资源占位符，后续在 Cloudflare 创建资源后补充完整。

1. 打开你的 GitHub 仓库 → 找到根目录下的 `wrangler.toml` 文件 → 点击 **「Edit」**（编辑）
2. 找到以下配置段，按注释提示预留占位符（后续从 Cloudflare 复制信息填入）：

toml

```toml
# D1 数据库配置（后续从 Cloudflare D1 复制）
[[d1_databases]]
binding = "DB"          # 绑定变量名，必须为 DB，不可改
database_name = "notes-db"  # 后续 Cloudflare 创建的 D1 数据库名称（默认填推荐名）
database_id = "【替换为 Cloudflare D1 数据库 ID】"  # 后续从 Cloudflare 复制

# KV 命名空间配置（后续从 Cloudflare KV 复制）
[[kv_namespaces]]
binding = "NOTES_KV"    # 绑定变量名，必须为 NOTES_KV，不可改
id = "【替换为 Cloudflare KV 命名空间 ID】"  # 后续从 Cloudflare 复制

# R2 存储桶配置（后续从 Cloudflare R2 复制）
[[r2_buckets]]
binding = "NOTES_R2_BUCKET"  # 绑定变量名，必须为 NOTES_R2_BUCKET，不可改
bucket_name = "notes-r2-bucket"  # 后续 Cloudflare 创建的 R2 存储桶名称（默认填推荐名）
```

3. 点击页面底部的 **「Commit changes」**（提交修改），无需填写备注，直接提交即可。

### GitHub 平台操作总结

完成后，你已拥有一个属于自己的代码仓库，且 `wrangler.toml` 已预留资源绑定位置，接下来所有操作将集中在 Cloudflare 平台，无需再切换 GitHub。

## 二、Cloudflare 平台操作（核心：创建资源 + 部署 + 配置）

### 目标

创建 Memos 所需的核心资源（D1 数据库、KV 命名空间、R2 存储桶），部署应用，配置登录和功能开关，全程在 Cloudflare 控制台完成。

### 前置准备

- 一个 Cloudflare 账号（免费注册：[cloudflare.com](https://www.cloudflare.com/)）
- 已完成 GitHub 平台操作（拥有代码仓库和预留配置的 `wrangler.toml`）

### 步骤 1：创建核心资源（按顺序创建，记好关键信息）

需创建 3 类资源，每类资源的「推荐名称」和「绑定变量名」必须与 `wrangler.toml` 一致，否则部署失败。

|资源类型|推荐名称|绑定变量名（必须一致）|用途|
|---|---|---|---|
|D1 数据库|`notes-db`|`DB`|存储笔记、标签、文档等核心数据|
|KV 命名空间|`notes-kv`|`NOTES_KV`|存储会话、分享链接、用户设置等|
|R2 存储桶|`notes-r2-bucket`|`NOTES_R2_BUCKET`|存储上传的图片、文件附件等|

#### 1.1 创建 D1 数据库（存储核心数据）

1. 登录 Cloudflare 控制台 → 左侧导航栏找到 **「Workers & Pages」** → 点击顶部标签 **「D1」**
2. 点击 **「Create database」**，填写数据库名称（必须与 `wrangler.toml` 一致，推荐 `notes-db`），点击 **「Create」**
3. 数据库创建后，点击进入该数据库 → 点击顶部 **「Console」**（控制台）
4. 回到你的 GitHub 仓库 → 找到 `schema.sql` 文件（通常在根目录或 `src` 文件夹下）
5. 复制 `schema.sql` 中的**全部内容** → 粘贴到 Cloudflare D1 控制台的输入框中 → 点击 **「Run」**（执行）
6. 执行成功后，记录两个关键信息（用于更新 GitHub 的 `wrangler.toml`）：
    - 数据库 ID（`database_id`）：在数据库详情页的 **「Settings」** → **「Database ID」** 中查看
    - （可选）数据库名称：若未使用推荐名，需记录实际名称

#### 1.2 创建 KV 命名空间（存储临时数据）

1. 回到 Cloudflare 控制台 → 左侧导航栏 **「Workers & Pages」** → 顶部标签 **「KV」**
2. 点击 **「Create a namespace」**，填写名称（必须与 `wrangler.toml` 一致，推荐 `notes-kv`），点击 **「Create」**
3. 记录其 **「ID」**（在命名空间列表的「ID」列），用于更新 GitHub 的 `wrangler.toml`

#### 1.3 创建 R2 存储桶（存储文件附件）

1. 回到 Cloudflare 控制台 → 左侧导航栏 **「R2」** → 点击 **「Create bucket」**
2. 填写存储桶名称（必须与 `wrangler.toml` 一致，推荐 `notes-r2-bucket`），其他设置默认 → 点击 **「Create」**
3. （可选）若未使用推荐名，记录实际存储桶名称，用于更新 GitHub 的 `wrangler.toml`

### 步骤 2：更新 GitHub 的 `wrangler.toml`（关键：绑定资源）

1. 回到你的 GitHub 仓库 → 再次编辑 `wrangler.toml` 文件
2. 将步骤 1 记录的关键信息填入对应的占位符：
    - `database_id`：粘贴 Cloudflare D1 数据库 ID
    - `id`（KV 部分）：粘贴 Cloudflare KV 命名空间 ID
    - （可选）若资源名称与推荐名不一致，同步修改 `database_name` 和 `bucket_name`
3. 点击 **「Commit changes」** 提交修改（这是最后一次操作 GitHub）

### 步骤 3：部署应用到 Cloudflare Workers

1. 回到 Cloudflare 控制台 → 左侧导航栏 **「Workers & Pages」** → 点击 **「Create application」**
2. 选择 **「Workers」** → 点击 **「Connect to Git」**（连接 Git 仓库）
3. 在「Connect to Git」页面：
    - 选择你的代码仓库平台（GitHub），点击 **「Authorize Cloudflare」**（按提示授权）
    - 授权后，在仓库列表中找到你 Fork / 创建的 Memos Worker 仓库 → 点击 **「Select」**
4. 部署配置页面保持默认（框架选择「None」，构建命令 / 输出目录留空）→ 点击 **「Save and Deploy」**
5. 等待部署完成（约 1-2 分钟），页面会显示「Deployment successful」（部署成功）

### 步骤 4：配置环境变量（登录 + 功能开关）

部署完成后，需添加环境变量才能正常登录和使用部分功能：

1. 进入你的 Worker 项目页面 → 左侧导航栏 **「Settings」**（设置）→ 点击 **「Environment Variables」**（环境变量）
2. 点击 **「Add variable」**，按以下表格添加变量（必填项必须填，可选项按需填）：

| 变量名                       | 是否必填 | 说明                                       | 示例值                                          |
| ------------------------- | ---- | ---------------------------------------- | -------------------------------------------- |
| `USERNAME`                | 是    | 登录用户名（自定义，如 admin、你的邮箱）                  | `my_memos`                                   |
| `PASSWORD`                | 是    | 登录密码（建议复杂一些，至少 6 位）                      | `Xx123456!`                                  |
| `TELEGRAM_BOT_TOKEN`      | 否    | Telegram 机器人 Token（需先创建机器人，不使用可留空）       | `123456789:ABCdefGhIJKlmNoPQRstUvWxYz123456` |
| `TELEGRAM_WEBHOOK_SECRET` | 否    | 随机长字符串（用于验证 Telegram Webhook，如不使用可留空）    | `random_secret_123456`                       |
| `AUTHORIZED_TELEGRAM_IDS` | 否    | 授权使用 Telegram Bot 的用户 ID（多个用逗号分隔，不使用可留空） | `12345678,87654321`                          |

3. 每添加一个变量，填写「Variable name」和「Value」，点击 **「Save」**
4. 所有变量添加完成后，点击页面顶部的 **「Deploy」** → 「Deploy again」（重新部署），确保变量生效

### 步骤 5：访问你的 Memos 应用

1. 回到 Worker 项目页面 → 顶部导航栏 **「Overview」**（概览）
2. 找到「Worker URL」（如 `https://memos-worker-my.cloudflareworkers.com`），点击该链接
3. 进入登录页面，输入你设置的 `USERNAME` 和 `PASSWORD`，即可登录使用！

### 步骤 6：(可选) 激活 Telegram Bot 功能

如果需要通过 Telegram 发送笔记，继续在 Cloudflare 平台完成以下操作：

1. 准备两个关键信息：
    
    - `TELEGRAM_BOT_TOKEN`：你的 Telegram 机器人 Token（在 [@BotFather](https://t.me/BotFather) 处创建机器人获取）
    - `TELEGRAM_WEBHOOK_SECRET`：你在步骤 4 中设置的随机字符串
    - 你的 Worker 域名：即步骤 5 的「Worker URL」（如 `https://memos-worker-my.cloudflareworkers.com`）
2. 构造 Webhook 激活链接（替换以下占位符）：
    
    plaintext
    
    ```plaintext
    https://api.telegram.org/bot<TELEGRAM_BOT_TOKEN>/setWebhook?url=https://<你的Worker域名>/api/telegram_webhook/<TELEGRAM_WEBHOOK_SECRET>&secret_token=<TELEGRAM_WEBHOOK_SECRET>
    ```
    
3. 示例（替换后）：
    
    plaintext
    
    ```plaintext
    https://api.telegram.org/bot123456789:ABCdefGhIJKlmNoPQRstUvWxYz123456/setWebhook?url=https://memos-worker-my.cloudflareworkers.com/api/telegram_webhook/random_secret_123456&secret_token=random_secret_123456
    ```
    
4. 打开浏览器，粘贴构造好的链接，按下回车。如果页面返回以下内容，说明激活成功：
    
    json
    
    ```json
    {"ok":true,"result":true,"description":"Webhook was set"}
    ```
    
5. 测试：给你的 Telegram 机器人发送一条消息（如「测试笔记」），刷新 Memos 应用，即可看到笔记已同步。
    

### Cloudflare 平台操作总结

完成后，你的 Memos 应用已正式部署并可用，所有核心功能均已配置完成，无需再操作 Cloudflare 控制台（除非需要后续维护）。

## 三、本地环境操作（可选：命令行部署 + 开发调试）

### 目标

适合需要修改代码、本地调试功能的用户，全程在本地命令行操作，不影响已部署的线上应用。

### 前置准备

- 本地安装 Node.js（v18+，下载地址：[nodejs.org](https://nodejs.org/)）
- 已完成 GitHub 和 Cloudflare 平台操作（拥有代码仓库和 Cloudflare 资源）
- 本地安装 Git（用于克隆仓库）

### 步骤 1：安装必要工具

1. 打开本地命令行（Windows：CMD/PowerShell；Mac/Linux：Terminal）
2. 安装 Cloudflare Wrangler 工具（Worker 命令行工具）：
    
    bash
    
    ```bash
    npm install -g wrangler
    ```
    

### 步骤 2：克隆代码仓库到本地

1. 复制你的 GitHub 仓库地址（仓库页面 → 点击「Code」→ 复制 HTTPS 地址，如 `https://github.com/你的用户名/你的仓库名.git`）
2. 在命令行中执行以下命令（替换仓库地址）：
    
    bash
    
    ```bash
    git clone https://github.com/你的用户名/你的仓库名.git
    cd 你的仓库名
    ```
    

### 步骤 3：配置本地环境变量

1. 在项目根目录创建 `.dev.vars` 文件（文件名固定，Git 会自动忽略，避免泄露密钥）
2. 用记事本 / VS Code 打开该文件，写入以下内容（替换为你的实际配置）：
    
    ini
    
    ```ini
    USERNAME="你的登录用户名"
    PASSWORD="你的登录密码"
    # 可选：添加 Telegram 相关变量（如使用）
    # TELEGRAM_BOT_TOKEN="你的机器人Token"
    # TELEGRAM_WEBHOOK_SECRET="你的随机字符串"
    ```
    

### 步骤 4：本地部署 / 调试

#### 4.1 本地模拟调试（不连接云端资源）

1. 初始化本地 D1 数据库（仅首次需要）：
    
    bash
    
    ```bash
    npx wrangler d1 execute notes-db --local --file=./schema.sql
    ```
    
2. 启动本地开发服务器：
    
    bash
    
    ```bash
    npx wrangler dev
    ```
    
3. 启动后，访问 `http://localhost:8787` 即可本地调试，修改代码会自动热更新。

#### 4.2 连接云端资源调试（使用 Cloudflare 真实资源）

bash

```bash
npx wrangler dev --remote
```

#### 4.3 本地命令行部署到 Cloudflare

bash

```bash
npx wrangler deploy
```

部署成功后，命令行会输出你的 Worker URL，与 Cloudflare 控制台部署的地址一致。

### 本地环境操作总结

本地环境仅用于开发调试，线上应用仍以 Cloudflare 控制台部署的版本为准，无需担心本地操作影响线上使用。

## 💡 小白必看使用技巧

1. **预览文本附件**：右键点击 `.txt`、`.md`、`.json` 等文本类附件，选择「在新标签页打开」，即可直接预览原始内容，无需下载。
2. **Telegram Proxy 选项**：
    - 开启：视频 / 文件不存储到 R2，通过代理链接访问，节省存储空间（风险：Telegram 删除文件后链接失效）
    - 关闭：自动下载视频 / 文件到 R2，永久存储（推荐，适合需要长期保存的场景）
3. **Keep Time 复选框**：
    - 勾选（默认）：编辑笔记后保留原始时间戳，不改变笔记在时间线中的位置
    - 取消勾选：编辑后更新时间戳，笔记置顶到最新位置

## ❌ 常见问题排查

1. **登录失败**：检查 Cloudflare 环境变量 `USERNAME` 和 `PASSWORD` 是否填写正确，是否重新部署生效。
2. **数据库连接失败**：检查 GitHub 的 `wrangler.toml` 中 `database_id` 和 `database_name` 是否与 Cloudflare D1 一致。
3. **文件上传失败**：检查 R2 存储桶名称是否与 `wrangler.toml` 一致，确保 R2 未设置访问限制。
4. **Telegram 消息不同步**：检查 Webhook 激活链接是否正确，`TELEGRAM_BOT_TOKEN` 和 `TELEGRAM_WEBHOOK_SECRET` 是否匹配。

按「GitHub → Cloudflare → 本地环境」的顺序操作，无需跨平台来回切换，小白也能轻松部署 Memos Worker 私有化笔记应用！