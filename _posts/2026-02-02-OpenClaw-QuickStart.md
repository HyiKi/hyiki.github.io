---
title: OpenClaw 快速启动指南
date:  2026-02-02 20:30:00 +0800
category: Original
tags: [AI, OpenClaw, Assistant]
excerpt: OpenClaw 是一个可以在自己设备上运行的个人 AI 助手，支持多种聊天渠道。本文记录了完整的快速启动流程。
---

* content
{:toc}

# OpenClaw 快速启动指南

OpenClaw 是一个可以在自己设备上运行的个人 AI 助手。它可以在你已有的渠道上回答你（WhatsApp、Telegram、Slack、Discord、Google Chat、Signal、iMessage、Microsoft Teams、WebChat），以及扩展渠道如 BlueBubbles、Matrix、Zalo 等。它可以在 macOS/iOS/Android 上说话和听，并可以渲染一个你可以控制的实时 Canvas。

## 什么是 OpenClaw

OpenClaw 是一个开源的个人 AI 助手项目，让你完全掌控自己的 AI 助手。Gateway 只是控制平面，真正的产品是助手本身。

* **项目地址**: [https://github.com/openclaw/openclaw](https://github.com/openclaw/openclaw)
* **官方网站**: [https://openclaw.ai](https://openclaw.ai)
* **文档**: [https://docs.openclaw.ai](https://docs.openclaw.ai)

## 安装前准备

### 系统要求

* macOS / Linux / Windows (WSL2)
* Node.js 22+ （推荐使用 nvm 管理 Node 版本）
* 基本的命令行操作知识

### 安装命令

```bash
# 使用代理运行安装（根据你的网络环境调整）
proxy_run openclaw onboard --install-daemon
```

## 完整启动流程详解

![image-20260202112353103](https://raw.githubusercontent.com/HyiKi/picgo-asset/main/image-20260202112353103.png)

![image-20260202112408306](https://raw.githubusercontent.com/HyiKi/picgo-asset/main/image-20260202112408306.png)

### 步骤 0: 启动安装命令

```bash
# 使用代理运行安装（根据你的网络环境调整）
proxy_run openclaw onboard --install-daemon
```

**为什么需要 `--install-daemon`**: 这个参数会安装 Gateway 服务作为系统守护进程，让 OpenClaw 在后台持续运行，即使关闭终端也不会停止。

**命令说明**:

* `proxy_run`: 如果你的网络环境需要代理才能访问某些服务，使用代理运行命令
* `openclaw onboard`: 启动交互式配置向导
* `--install-daemon`: 安装后台服务

### 步骤 1: 安全警告确认

启动时会显示安全警告，需要仔细阅读并确认：

**为什么需要安全警告**:

* OpenClaw 是一个业余项目，仍处于测试阶段，可能存在未知问题
* 这个机器人可以读取文件并在启用工具时运行操作，具有系统访问权限
* 恶意提示可能会诱骗它执行不安全操作
* 需要用户充分理解风险后再继续

**安全建议**:

* 使用配对/白名单 + 提及门控
* 使用沙箱 + 最小权限工具
* 定期运行安全审计：`openclaw security audit --deep`

**操作**: 选择 "Yes" 继续

**重要文档**: [https://docs.openclaw.ai/gateway/security](https://docs.openclaw.ai/gateway/security)

### 步骤 2: 选择配置模式

系统会询问 "Onboarding mode"，推荐选择 **QuickStart**。

**为什么选择 QuickStart**:

* 快速配置，适合首次使用
* 使用默认的安全设置（Loopback 绑定，Token 认证）
* 稍后可以随时调整配置

**配置选项**:

* **QuickStart**: 快速开始，使用默认设置
* **Custom**: 自定义配置，适合有经验的用户

### 步骤 3: 配置检测与处理

如果之前已经配置过 OpenClaw，系统会检测到现有配置：

```
workspace: ~/.openclaw/workspace
model: moonshot/kimi-k2-0905-preview
gateway.mode: local
gateway.port: 18789
gateway.bind: loopback
skills.nodeManager: npm
```

**为什么需要检测现有配置**:

* 避免覆盖已有配置
* 可以选择使用现有配置或重新配置
* 保留之前的工作空间和会话数据

**操作**: 选择 "Use existing values" 使用现有配置，或选择其他选项重新配置

### 步骤 4: 模型提供商配置

#### 4.1 选择模型提供商

系统会询问 "Model/auth provider"，可以选择：

* **Qwen**: 通义千问（国内用户推荐）
* **Moonshot**: 月之暗面
* **其他**: 根据你的需求选择

**为什么需要配置模型**:

* OpenClaw 需要 AI 模型来生成回复
* 不同模型有不同的特点和价格
* 需要 API 密钥或 OAuth 认证

#### 4.2 Qwen OAuth 认证流程

如果选择 Qwen 作为模型提供商：

1. **选择认证方式**: 选择 "Qwen OAuth"
   * **为什么使用 OAuth**: OAuth 比 API Key 更安全，token 会自动刷新

2. **获取授权链接**: 系统会生成一个授权链接和用户代码

   ```
   https://chat.qwen.ai/authorize?user_code=HKUSO5_M&client=qwen-code
   用户代码: HKUSO5_M
   ```

3. **完成授权**:
   * 在浏览器中打开链接
   * 如果提示，输入用户代码
   * 完成授权后返回终端

4. **验证配置**: 系统会显示 "Qwen OAuth complete"

**配置结果**:

* **默认模型**: `qwen-portal/coder-model`
* **Base URL**: `https://portal.qwen.ai/v1`（可在配置中覆盖）

**为什么选择 coder-model**: 这是一个针对代码和开发场景优化的模型，适合技术助手使用。

### 步骤 5: 通道配置（可选）

OpenClaw 支持多种聊天渠道，在快速启动中可以选择 "Skip for now" 稍后配置。

**支持的渠道**:

* **Telegram**: 最简单的方式，注册一个 bot 即可开始
* **WhatsApp**: 使用自己的号码，建议使用单独的手机 + eSIM
* **Discord**: 目前支持良好
* **Google Chat**: Google Workspace Chat 应用，使用 HTTP webhook
* **Slack**: 支持（Socket Mode）
* **Signal**: 需要 signal-cli 链接设备
* **iMessage**: 仍在开发中
* **飞书 (Feishu)**: 企业级聊天应用，支持文档、知识库等高级功能（详见下方配置章节）
* **其他**: Nostr、Microsoft Teams、Mattermost、Nextcloud Talk、Matrix、BlueBubbles、LINE、Zalo 等

**DM 安全机制**:

* **默认模式**: 配对模式，未知 DM 会收到配对代码
* **批准配对**: `openclaw pairing approve <channel> <code>`
* **公开 DM**: 需要 `dmPolicy="open" + allowFrom=["*"]`

**为什么默认使用配对模式**: 防止未授权用户直接与机器人对话，提高安全性。

### 步骤 6: Skills 配置

Skills 是 OpenClaw 的功能扩展，提供各种工具和能力。

**Skills 状态**:

* **Eligible**: 7 个可用的技能（已满足所有依赖）
* **Missing requirements**: 38 个缺少依赖的技能（需要安装依赖）
* **Blocked by allowlist**: 0 个被阻止的技能

**为什么需要配置 Skills**:

* Skills 提供各种功能，如文件操作、网络搜索、代码执行等
* 某些 Skills 需要额外的依赖或配置
* 可以根据需求选择安装

**操作选择**:

* **安装缺失的依赖**: 推荐，可以启用更多功能
* **跳过稍后配置**: 可以稍后通过 `openclaw configure` 配置

**节点管理器**: 选择 npm 作为技能安装的包管理器（推荐，兼容性最好）

### 步骤 7: Hooks 配置

Hooks 允许你在代理命令发出时自动化操作。

**可用的 Hooks**:

* **boot-md**: 启动时执行，可以初始化工作空间
* **command-logger**: 命令日志记录，记录所有命令执行
* **session-memory**: 会话记忆保存，保存对话上下文

**为什么需要 Hooks**:

* 自动化常见操作
* 记录和追踪代理行为
* 持久化会话数据

**推荐配置**: 启用 `boot-md`、`command-logger`、`session-memory`

**管理命令**:

```bash
# 查看所有 hooks
openclaw hooks list

# 启用 hook
openclaw hooks enable <name>

# 禁用 hook
openclaw hooks disable <name>
```

### 步骤 8: Gateway 服务安装

Gateway 是 OpenClaw 的控制平面服务，负责管理代理、会话和工具。

#### 8.1 运行时选择

QuickStart 使用 Node.js 作为 Gateway 服务运行时（稳定且受支持）。

**为什么使用 Node.js**:

* 稳定且受官方支持
* 跨平台兼容性好
* 社区支持完善

#### 8.2 安装过程

1. **检查现有安装**: 如果已安装，可以选择重新安装
2. **Node.js 版本检查**: 系统会检查 Node.js 版本（需要 Node 22+）
   * 如果系统 Node 版本不符合要求，会使用 nvm 中的 Node 版本
   * 建议安装 Node 22+ 从 nodejs.org 或 Homebrew
3. **安装 LaunchAgent**:
   * macOS: 安装到 `~/Library/LaunchAgents/ai.openclaw.gateway.plist`
   * 服务会在系统启动时自动运行
4. **日志位置**: `~/.openclaw/logs/gateway.log`

**为什么需要守护进程**:

* 让 Gateway 在后台持续运行
* 即使关闭终端也不会停止服务
* 系统重启后自动启动

#### 8.3 服务信息

安装完成后，系统会显示服务信息：

* **Web UI**: `http://127.0.0.1:18789/`（需要 token 认证）
* **Web UI (with token)**: `http://127.0.0.1:18789/?token=<your-token>`（可直接访问）
* **Gateway WS**: `ws://127.0.0.1:18789`（WebSocket 连接）
* **Gateway token**: 存储在 `~/.openclaw/openclaw.json` 的 `gateway.auth.token` 中

**获取带 token 的链接**:

```bash
openclaw dashboard --no-open
```

**为什么需要 token**: 保护 Gateway 不被未授权访问，确保只有你才能控制代理。

### 步骤 9: TUI 启动与使用

TUI（Terminal User Interface）是启动代理的最佳方式，提供交互式对话界面。

#### 9.1 启动 TUI

```bash
openclaw tui - ws://127.0.0.1:18789 - agent main - session main
```

**参数说明**:

* `tui`: 启动终端用户界面
* `ws://127.0.0.1:18789`: Gateway WebSocket 地址
* `agent main`: 使用名为 "main" 的代理
* `session main`: 使用名为 "main" 的会话

#### 9.2 TUI 界面说明

启动后会显示：

```
System: [2026-02-02 09:47:04 GMT+8] Node: MacBook Pro (4) (192.168.18.98)
app 2026.1.29 (8345) · mode local

agent main | session main | qwen-portal/coder-model | tokens 16k/128k (13%)
```

**界面元素**:

* **系统信息**: 节点名称、IP 地址、应用版本、运行模式
* **连接状态**: `connected | idle` 表示已连接并空闲
* **代理信息**: 当前使用的代理、会话、模型和 token 使用情况

#### 9.3 首次对话

**首次启动**: 系统会发送 "Wake up, my friend!" 来初始化会话，代理会回复问候语。

**开始对话**: 直接在 TUI 中输入消息，按 Enter 发送。

**常用命令**:

* `/new`: 开启新对话，清除当前会话上下文
* `/reset`: 重置会话，等同于 `/new`
* `/help`: 查看帮助信息

#### 9.4 TUI 使用技巧

1. **多行输入**: 使用 Shift+Enter 换行，Enter 发送
2. **查看历史**: 使用上下箭头键浏览历史消息
3. **退出**: 使用 Ctrl+C 或输入 `/exit` 退出 TUI
4. **Token 使用**: 注意观察 token 使用情况，避免超出模型限制

**为什么使用 TUI**:

* 最直接的交互方式
* 可以看到完整的对话历史
* 适合调试和开发
* 不需要额外的浏览器或应用

## 飞书机器人集成

飞书（Feishu/Lark）是企业级协作平台，OpenClaw 支持通过飞书机器人进行对话，并可以访问飞书文档、知识库和云空间。

![image-20260202161326979](https://raw.githubusercontent.com/HyiKi/picgo-asset/main/image-20260202161326979.png)

### 安装飞书插件

```bash
# 安装飞书插件
openclaw plugins install @m1heng-clawd/feishu
```

**如果安装失败**（Windows 上可能出现 `spawn npm ENOENT` 错误）：

```bash
# 1. 下载插件包
curl -O https://registry.npmjs.org/@m1heng-clawd/feishu/-/feishu-0.1.3.tgz

# 2. 从本地安装
openclaw plugins install ./feishu-0.1.3.tgz
```

**插件项目**: [https://github.com/m1heng/Clawdbot-feishu](https://github.com/m1heng/Clawdbot-feishu)

### 飞书开放平台配置

#### 1. 创建应用

1. 访问 [飞书开放平台](https://open.feishu.cn/)（国内）或 [Lark Developer](https://open.larksuite.com/)（国际）
2. 登录你的飞书账号
3. 进入 **开发者后台** → **创建企业自建应用**
4. 填写应用信息：
   * **应用名称**: 例如 "OpenClaw Assistant"
   * **应用描述**: 个人 AI 助手
   * **应用图标**: 上传一个图标（可选）

#### 2. 获取凭证

在应用后台的 **凭证与基础信息** 页面：

* **App ID**: `cli_xxxxx`（复制保存）
* **App Secret**: 点击"重置"生成密钥（复制保存）

**重要**: 这些凭证需要保密，不要泄露给他人。

#### 3. 配置权限

在 **权限管理** 页面，申请以下权限：

##### 必需权限（消息收发）

| 权限 | 范围 | 说明 |
|------|------|------|
| `contact:user.base:readonly` | 用户信息 | 获取用户基本信息（用于解析发送者姓名） |
| `im:message` | 消息 | 发送和接收消息 |
| `im:message.p2p_msg:readonly` | 私聊 | 读取发给机器人的私聊消息 |
| `im:message.group_at_msg:readonly` | 群聊 | 接收群内 @机器人 的消息 |
| `im:message:send_as_bot` | 发送 | 以机器人身份发送消息 |
| `im:resource` | 媒体 | 上传和下载图片/文件 |

##### 可选权限（高级功能）

| 权限 | 范围 | 说明 |
|------|------|------|
| `im:message.group_msg` | 群聊 | 读取所有群消息（敏感） |
| `im:message:readonly` | 读取 | 获取历史消息 |
| `im:message:update` | 编辑 | 更新/编辑已发送消息 |
| `im:message:recall` | 撤回 | 撤回已发送消息 |
| `im:message.reactions:read` | 表情 | 查看消息表情回复 |

##### 工具权限（文档/知识库/云空间）

**只读权限**（最低要求）：

| 权限 | 工具 | 说明 |
|------|------|------|
| `docx:document:readonly` | feishu_doc | 读取文档 |
| `drive:drive:readonly` | feishu_drive | 列出文件夹、获取文件信息 |
| `wiki:wiki:readonly` | feishu_wiki | 列出空间、列出节点、获取节点详情、搜索 |

**读写权限**（可选，用于创建/编辑/删除操作）：

| 权限 | 工具 | 说明 |
|------|------|------|
| `docx:document` | feishu_doc | 创建/编辑文档 |
| `docx:document.block:convert` | feishu_doc | Markdown 转 blocks（write/append 必需） |
| `drive:drive` | feishu_doc, feishu_drive | 上传图片到文档、创建文件夹、移动/删除文件 |
| `wiki:wiki` | feishu_wiki | 创建/移动/重命名知识库节点 |

**重要**: 申请权限后需要等待审核通过（通常几分钟到几小时）。

#### 4. 配置事件订阅 ⚠️

**这是最容易遗漏的配置！** 如果机器人能发消息但收不到消息，请检查此项。

在 **事件与回调** 页面：

1. **事件配置方式**: 选择 **使用长连接接收事件**（推荐）
   * 为什么使用长连接：更稳定，不需要配置公网地址

2. **添加事件订阅**，勾选以下事件：

| 事件 | 说明 |
|------|------|
| `im.message.receive_v1` | 接收消息（必需） |
| `im.message.message_read_v1` | 消息已读回执 |
| `im.chat.member.bot.added_v1` | 机器人进群 |
| `im.chat.member.bot.deleted_v1` | 机器人被移出群 |

1. **确保事件订阅的权限已申请并通过审核**

#### 5. 发布应用

1. 在 **版本管理与发布** 页面
2. 创建版本（至少发布到测试版本）
3. 设置可用范围（选择你的账号或组织）
4. 提交审核（如果需要）

**为什么需要发布**: 只有发布的应用才能在飞书中搜索和使用。

### 云空间访问权限配置 ⚠️

**重要**: 机器人没有自己的"我的空间"（根目录）。机器人只能访问**被分享给它的文件/文件夹**。

要让机器人管理文件：

1. 在你的飞书云空间创建一个文件夹
2. 右键文件夹 → **分享** → 搜索机器人名称
3. 授予相应权限（查看/编辑）

如果不做这一步，`feishu_drive` 的 `create_folder` 等操作会失败，因为机器人没有根目录可以创建文件夹。

### 知识库空间权限配置 ⚠️

**重要**: 仅有 API 权限不够，还需要将机器人添加到知识库空间。

1. 打开需要机器人访问的知识库空间
2. 点击 **设置**（齿轮图标）→ **成员管理**
3. 点击 **添加成员** → 搜索机器人名称
4. 选择权限级别（查看/编辑）

如果不做这一步，即使 API 权限正确，`feishu_wiki` 也会返回空结果。

参考文档：[知识库常见问题 - 如何将应用添加为知识库成员](https://open.feishu.cn/document/ukTMukTMukTM/uUDN04SN0QjL1QDN)

### OpenClaw 配置

#### 使用命令行配置

```bash
# 设置 App ID
openclaw config set channels.feishu.appId "cli_xxxxx"

# 设置 App Secret
openclaw config set channels.feishu.appSecret "your_app_secret"

# 启用飞书通道
openclaw config set channels.feishu.enabled true
```

#### 手动编辑配置文件

编辑 `~/.openclaw/openclaw.json`，添加以下配置：

```json
{
  "channels": {
    "feishu": {
      "enabled": true,
      "appId": "cli_xxxxx",
      "appSecret": "your_app_secret",
      "domain": "feishu",
      "connectionMode": "websocket",
      "dmPolicy": "pairing",
      "groupPolicy": "allowlist",
      "requireMention": true,
      "mediaMaxMb": 30,
      "renderMode": "auto"
    }
  }
}
```

#### 配置选项说明

| 选项 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `enabled` | boolean | `false` | 是否启用飞书通道 |
| `appId` | string | - | 飞书应用 App ID（必需） |
| `appSecret` | string | - | 飞书应用 App Secret（必需） |
| `domain` | string | `"feishu"` | 域名：`"feishu"`（国内）或 `"lark"`（国际） |
| `connectionMode` | string | `"websocket"` | 连接模式：`"websocket"`（推荐）或 `"webhook"` |
| `dmPolicy` | string | `"pairing"` | 私聊策略：`"pairing"` \| `"open"` \| `"allowlist"` |
| `groupPolicy` | string | `"allowlist"` | 群聊策略：`"open"` \| `"allowlist"` \| `"disabled"` |
| `requireMention` | boolean | `true` | 群聊是否需要 @机器人 |
| `mediaMaxMb` | number | `30` | 媒体文件最大大小（MB） |
| `renderMode` | string | `"auto"` | 回复渲染模式：`"auto"` \| `"raw"` \| `"card"` |

#### 渲染模式说明

| 模式 | 说明 |
|------|------|
| `auto` | （默认）自动检测：有代码块或表格时用卡片，否则纯文本 |
| `raw` | 始终纯文本，表格转为 ASCII |
| `card` | 始终使用卡片，支持语法高亮、表格、链接等 |

### 重启服务

配置完成后，重启 Gateway 服务使配置生效：

```bash
# 重启服务
openclaw restart

# 或停止后启动
openclaw stop
openclaw start
```

### 在飞书中使用

1. **搜索机器人**: 在飞书搜索框中搜索机器人名称
2. **开始私聊**: 点击机器人头像开始私聊
3. **添加到群聊**: 在群聊中添加机器人，@机器人即可对话

**首次私聊**: 如果使用配对模式，会收到配对代码，需要在终端中批准：

```bash
openclaw pairing approve feishu <code>
```

### 飞书功能特性

* ✅ WebSocket 和 Webhook 连接模式
* ✅ 私聊和群聊
* ✅ 消息回复和引用上下文
* ✅ **入站媒体支持**：AI 可以看到图片、读取文件（PDF、Excel 等）、处理富文本中的嵌入图片
* ✅ 图片和文件上传（出站）
* ✅ 输入指示器（通过表情回复实现）
* ✅ 私聊配对审批流程
* ✅ 用户和群组目录查询
* ✅ **卡片渲染模式**：支持语法高亮的 Markdown 渲染
* ✅ **文档工具**：读取、创建、用 Markdown 写入飞书文档
* ✅ **知识库工具**：浏览知识库、列出空间、获取节点详情、搜索、创建/移动/重命名节点
* ✅ **云空间工具**：列出文件夹、获取文件信息、创建文件夹、移动/删除文件
* ✅ **@ 转发功能**：在消息中 @ 某人，机器人的回复会自动 @ 该用户
* ✅ **权限错误提示**：当机器人遇到飞书 API 权限错误时，会自动通知用户并提供权限授权链接

#### @ 转发功能

如果你希望机器人的回复中 @ 某人，只需在你的消息中 @ 他们：

* **私聊**: `@张三 跟他问好` → 机器人回复 `@张三 你好！`
* **群聊**: `@机器人 @张三 跟他问好` → 机器人回复 `@张三 你好！`

机器人会自动检测消息中的 @ 并在回复时带上。无需额外权限。

### 飞书常见问题

#### 机器人收不到消息

检查以下配置：

1. ✅ 是否配置了 **事件订阅**？（见上方事件订阅章节）
2. ✅ 事件配置方式是否选择了 **长连接**？
3. ✅ 是否添加了 `im.message.receive_v1` 事件？
4. ✅ 相关权限是否已申请并审核通过？
5. ✅ Gateway 服务是否正在运行？

#### 返回消息时 403 错误

确保已申请 `im:message:send_as_bot` 权限，并且权限已审核通过。

#### 如何清理历史会话 / 开启新对话

在聊天中发送 `/new` 命令即可开启新对话。

#### 消息为什么不是流式输出

飞书 API 有请求频率限制，流式更新消息很容易触发限流。当前采用完整回复后一次性发送的方式，以保证稳定性。

#### 在飞书里找不到机器人

1. 确保应用已发布（至少发布到测试版本）
2. 在飞书搜索框中搜索机器人名称
3. 检查应用可用范围是否包含你的账号

#### 文档/知识库工具无法使用

1. **文档工具**: 确保已申请 `docx:document:readonly` 权限并审核通过
2. **知识库工具**:
   * 确保已申请 `wiki:wiki:readonly` 权限并审核通过
   * **重要**: 必须将机器人添加到知识库空间的成员中（见上方知识库权限配置）
3. **云空间工具**:
   * 确保已申请 `drive:drive:readonly` 权限并审核通过
   * **重要**: 必须将文件夹分享给机器人（见上方云空间权限配置）

## 配置位置

所有配置都存储在 `~/.openclaw/` 目录下：

* **配置文件**: `~/.openclaw/openclaw.json`
* **工作空间**: `~/.openclaw/workspace`
* **会话存储**: `~/.openclaw/agents/main/sessions/sessions.json`
* **日志文件**: `~/.openclaw/logs/gateway.log`

## 常用命令

### 服务管理

```bash
# 启动 Gateway 服务
openclaw start

# 停止 Gateway 服务
openclaw stop

# 查看服务状态
openclaw status
```

### 配置管理

```bash
# 查看配置
openclaw configure

# 配置特定部分
openclaw configure --section web

# 查看仪表板链接
openclaw dashboard --no-open
```

### 安全审计

```bash
# 深度安全审计
openclaw security audit --deep

# 修复安全问题
openclaw security audit --fix
```

### 配对管理

```bash
# 批准配对
openclaw pairing approve <channel> <code>

# 查看配对列表
openclaw pairing list
```

### Hooks 管理

```bash
# 列出所有 hooks
openclaw hooks list

# 启用 hook
openclaw hooks enable <name>

# 禁用 hook
openclaw hooks disable <name>
```

## 可选功能

### Web 搜索

如果需要代理能够搜索网络，需要设置 Brave Search API key：

```bash
# 交互式配置
openclaw configure --section web

# 或设置环境变量
export BRAVE_API_KEY=your_api_key
```

### 移动端应用

* **macOS 应用**: 系统集成 + 通知
* **iOS 应用**: 相机/画布功能
* **Android 应用**: 相机/画布功能

## 故障排除

### Gateway 连接问题

如果遇到 "HTTP 401: Invalid Authentication" 错误：

1. 检查 Gateway token 是否正确
2. 确认 Gateway 服务正在运行
3. 验证配置文件中的 token 设置

### Node.js 版本问题

如果系统 Node.js 版本不符合要求：

```bash
# 使用 nvm 安装 Node 22+
nvm install 22
nvm use 22

# 或从 nodejs.org 或 Homebrew 安装
```

### 控制 UI 资源缺失

如果提示 "Missing Control UI assets"：

```bash
# 构建 UI 资源（会自动安装 UI 依赖）
pnpm ui:build
```

## 下一步

1. **阅读文档**: [https://docs.openclaw.ai](https://docs.openclaw.ai)
2. **查看示例**: [https://openclaw.ai/showcase](https://openclaw.ai/showcase)
3. **配置聊天渠道**:
   * 选择你常用的聊天应用进行配置
   * **推荐**: 配置飞书机器人，支持文档、知识库等企业级功能
4. **探索 Skills**: 安装和配置你需要的功能扩展
5. **自定义代理**: 通过 TUI 与代理对话，让它了解你的需求和工作习惯
6. **配置 Web 搜索**: 如果需要代理能够搜索网络，配置 Brave Search API key
7. **安全加固**: 运行安全审计，配置配对和白名单

## 参考资源

### OpenClaw 官方资源

* **GitHub**: [https://github.com/openclaw/openclaw](https://github.com/openclaw/openclaw)
* **文档**: [https://docs.openclaw.ai](https://docs.openclaw.ai)
* **官方网站**: [https://openclaw.ai](https://openclaw.ai)
* **安全指南**: [https://docs.openclaw.ai/gateway/security](https://docs.openclaw.ai/gateway/security)
* **控制 UI 文档**: [https://docs.openclaw.ai/web/control-ui](https://docs.openclaw.ai/web/control-ui)
* **工作空间文档**: [https://docs.openclaw.ai/concepts/agent-workspace](https://docs.openclaw.ai/concepts/agent-workspace)

### 飞书集成资源

* **飞书插件 GitHub**: [https://github.com/m1heng/Clawdbot-feishu](https://github.com/m1heng/Clawdbot-feishu)
* **飞书开放平台（国内）**: [https://open.feishu.cn/](https://open.feishu.cn/)
* **Lark Developer（国际）**: [https://open.larksuite.com/](https://open.larksuite.com/)
* **飞书开放平台文档**: [https://open.feishu.cn/document/](https://open.feishu.cn/document/)
* **知识库常见问题**: [https://open.feishu.cn/document/ukTMukTMukTM/uUDN04SN0QjL1QDN](https://open.feishu.cn/document/ukTMukTMukTM/uUDN04SN0QjL1QDN)

## 注意事项

1. **安全第一**: OpenClaw 是一个强大的工具，需要谨慎使用。确保理解安全警告并采取适当的安全措施。

2. **定期备份**: 定期备份你的工作空间配置，避免数据丢失。

3. **保持更新**: 定期更新 OpenClaw 以获取最新功能和修复。

4. **社区支持**: 遇到问题时，可以查看 GitHub Issues 或社区讨论。

---

**版本信息**: OpenClaw 2026.1.29  
**最后更新**: 2026-02-02
