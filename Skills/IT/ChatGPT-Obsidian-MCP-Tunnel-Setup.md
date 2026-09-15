---
title: ChatGPT ↔ Obsidian MCP Tunnel Setup
status: working
tags:
  - obsidian
  - mcp
  - chatgpt
  - tunnel
  - knowledge-base
updated: 2026-09-01
---

# ChatGPT ↔ Obsidian MCP Tunnel Setup

## 目标

让 ChatGPT 能通过本地 Tunnel 访问 Mosman Interiors 的 Obsidian Vault，从而实现：

- 读取 Obsidian 知识库
- 搜索笔记
- 写入 / 追加 / 修改 Markdown
- 获取当前打开的文件
- 后续将公司 SOP、Odoo Workflow、Skills / Plugin 文档统一沉淀到 Vault

## 整体架构

```text
ChatGPT
  ↓
OpenAI / MCP Connector
  ↓
Local Tunnel Client (Windows)
  ↓
Obsidian Local REST API / MCP Server
  ↓
Obsidian Vault
```

核心思想：Obsidian 仍然运行在本地电脑，Vault 文件仍然保存在本地；Tunnel 负责把本地 MCP / REST 能力安全地暴露给 ChatGPT 使用。

## 当前 Windows 端环境

Tunnel 工作目录：

```text
C:\OpenAI-Tunnel
```

使用的 Profile：

```text
obsidian-http
```

曾使用以下命令检查连接状态：

```powershell
.\tunnel-client.exe doctor --profile obsidian-http --explain
```

## 已确认的健康状态

Doctor / runtime 检查曾确认：

- MCP session 初始化成功
- 本地 MCP Server：`obsidian-local-rest-api 1.0.0`
- OpenAI control plane 连接成功
- `/healthz` 返回 `200 live`
- `/readyz` 返回 `200 ready`

这几个结果同时成立时，可以认为 Tunnel 本身已正常工作。

## 关于 OAuth 检查警告

`doctor` 中曾出现 OAuth 静态检查与 Obsidian API-key 认证不兼容的提示。

这不一定表示连接失败。

如果实际运行时已经：

```text
MCP session initialized
/healthz = 200 live
/readyz = 200 ready
```

则应优先以实际 runtime ready 状态判断连接是否成功。

Obsidian Local REST API 使用 API Key 认证，而不是典型 OAuth，因此静态 OAuth 检查可能出现不适配。

## Codex Tunnel Plugin

检查结果曾显示：

```text
Codex tunnel plugin not installed
```

这并不阻止当前 ChatGPT ↔ Obsidian Tunnel 工作。

当前架构依赖的是 Tunnel Client + Obsidian Local REST API / MCP Server，而不是 Codex Tunnel Plugin。

## Tunnel 启动后的操作

当以下条件都满足时：

1. Obsidian 正在运行
2. Obsidian Local REST API / MCP Server 正在运行
3. Tunnel Client 正在后台运行
4. `/healthz` 为 live
5. `/readyz` 为 ready

下一步就是在 ChatGPT 的 Connector / MCP 设置中测试：

```text
Mosman Obsidian MCP Tunnel
```

测试成功后，ChatGPT 即可通过该连接器读取和操作 Vault。

## Token / API Key 原则

不要把任何 API Key、Tunnel Token 或 Connector Secret 写入 Obsidian 知识库正文。

知识库只记录：

- Token 在哪里生成
- Token 用于哪一层
- 如何更新 / 撤销
- 出错时如何重新配置

不要记录实际值。

如果 ChatGPT Connector 页面要求 Token，应先确认该字段属于：

- OpenAI Tunnel / Connector 身份认证
- 还是 Obsidian Local REST API API Key

不要凭字段名猜测，也不要把 Obsidian API Key 随意填入不匹配的认证字段。

## 安全处理

调试阶段曾把明文 API Key 放在：

```text
C:\OpenAI-Tunnel\api-key.txt
```

在 Codex 调试结束后，可以撤销 sandbox 对该文件的读取权限，例如：

```powershell
icacls "C:\OpenAI-Tunnel\api-key.txt" /remove "W05\codexsandboxoffline"
```

原则：

- 调试时尽量缩小密钥文件读取权限
- 不要把 Key 写入 Markdown / Git / Slack / Confluence
- 不要在 ChatGPT 对话中要求回显明文 Secret
- Token 泄露后应立即 revoke / rotate

## 日常启动顺序

建议固定：

```text
1. 启动 Obsidian
2. 确认 Obsidian Local REST API 可用
3. 启动 tunnel-client
4. 检查 /healthz
5. 检查 /readyz
6. 在 ChatGPT 测试 Mosman Obsidian MCP Tunnel
7. 再开始知识库读写
```

## 常见故障判断

### 1. healthz 不通

优先检查 Tunnel Client 是否正在运行。

### 2. healthz live，但 readyz 不 ready

Tunnel 进程存在，但后端 MCP / Obsidian API 可能未成功连接。

优先检查：

- Obsidian 是否打开
- Local REST API 插件 / MCP Server 是否运行
- API Key 是否正确
- Profile `obsidian-http` 配置是否正确

### 3. OAuth 检查失败但 readyz = ready

大概率是静态 OAuth 检查与 API-key 模式不兼容。

以实际 MCP session 和 readyz 状态为主。

### 4. ChatGPT 连接器测试失败

依次确认：

```text
Obsidian → Local API → Tunnel → OpenAI control plane → ChatGPT Connector
```

不要一次修改所有配置，应逐层检查。

## 目前已经验证的 Vault 能力

通过当前 ChatGPT 连接器，可以调用 Obsidian Vault API 进行：

- List files / directories
- Read Markdown
- Write Markdown
- Append
- Patch sections
- Move / Copy / Delete
- Search Vault
- Read tags
- 获取当前 active file
- 打开指定文件

这意味着 Obsidian 可以作为 Mosman Interiors 的内部知识底座，而 ChatGPT 可以作为知识的读取、总结、更新和 Workflow 接口。

## 推荐知识库定位

建议把 Obsidian 定位成：

```text
Mosman Internal Knowledge Layer
```

适合长期沉淀：

- Odoo SOP
- CRM / Project Workflow
- Automation Rules
- MCP / Connector 配置
- GPT Skills
- Plugin 说明
- Troubleshooting
- 会议决策
- 测试记录

而不是只当作普通笔记软件。

## 后续建议

- [ ] 建立 `Odoo/` 知识目录
- [ ] 建立 `MCP/` 或 `Infrastructure/` 目录
- [ ] 建立 `Skills/` 统一规范
- [ ] 将常用自动化代码整理成独立笔记
- [ ] 给重要 SOP 加 `status / owner / updated` frontmatter
- [ ] 为 Tunnel 建一个简短的启动 / 故障处理 Checklist
- [ ] 后续考虑把 ChatGPT Skill 文档直接关联到 Vault 中对应 SOP

## 一句话总结

> ChatGPT 通过 OpenAI Tunnel 连接本地 Obsidian MCP / REST API，使 Obsidian Vault 成为可由 GPT 安全读取和维护的公司内部知识库，同时 Vault 数据仍保存在本地。