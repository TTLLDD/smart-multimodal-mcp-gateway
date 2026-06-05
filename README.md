# Smart Multimodal MCP Gateway

[![License: AGPL-3.0](https://img.shields.io/badge/License-AGPL--3.0-blue.svg)](LICENSE)
[![Python 3.10+](https://img.shields.io/badge/Python-3.10%2B-green.svg)](https://www.python.org/)
[![MCP](https://img.shields.io/badge/MCP-Compatible-ff69b4.svg)](https://modelcontextprotocol.io/)

**一个本地运行的 BYOK MCP 网关，让 AI 工作台的高消耗任务走你自己的模型 API。**

Cursor、Windsurf、Copilot、Trae 这些闭源工作台都有额度限制，而且不允许你自由替换模型。这个网关通过 MCP 协议在你本地运行，把 PPT 生成、论文写作、代码审查、任务规划等重任务路由到你自己配置的模型（DeepSeek、通义千问、OpenAI、Anthropic 等），不消耗工作台额度。

Languages: [中文](#中文说明) | [English](#english)

---

## 中文说明

### 架构

```
┌──────────────────────────────────────────────┐
│              MCP 客户端                       │
│     Cursor / Windsurf / Claude Desktop       │
└─────────────────┬────────────────────────────┘
                  │ SSE (MCP 协议)
                  ▼
┌──────────────────────────────────────────────┐
│         Smart Multimodal MCP Gateway          │
│                                               │
│  ┌──────────┐  ┌───────────┐  ┌───────────┐  │
│  │smart_chat│  │task_exec  │  │create_ppt │  │
│  └─────┬────┘  └─────┬─────┘  └─────┬─────┘  │
│        │             │              │         │
│        └──────┬──────┴──────────────┘         │
│               ▼                               │
│  ┌────────────────────────────────────────┐   │
│  │            模型路由层                    │   │
│  │  auto_skill → ppt / paper / code / plan│   │
│  └─────────────────┬──────────────────────┘   │
└────────────────────┼──────────────────────────┘
                     │ API 调用
                     ▼
┌──────────────────────────────────────────────┐
│               模型 Provider                    │
│   DeepSeek / Qwen / OpenAI / Anthropic        │
└──────────────────────────────────────────────┘
```

### 痛点对照

| 你的痛点 | 这个项目怎么帮你 |
| --- | --- |
| 闭源工作台不能自由切 API | 通过 MCP 工具层接入本地 BYOK 模型 |
| 订阅、额度、积分压力大 | 把长文、PPT、论文等重活分流到 BYOK 模型 |
| 高 token 任务成本高 | 高消耗任务走自有 API，成本可控 |
| 多个模型切换麻烦 | 模型档案统一管理，自动路由 |
| MCP 工具太分散 | 一个 MCP Server 内置聊天、PPT、论文、规划、代码审查 |

### 它能解决什么

- **额度焦虑**：长内容生成、PPT、论文、规划等任务可转交给你的 BYOK 模型，不消耗工作台额度。
- **模型切换**：通过模型档案自动路由文本模型和多模态模型。
- **工具整合**：一个 MCP Server 内置 `auto_skill`、PPT、论文、任务规划、代码审查。
- **数据本地**：所有 Key、配置、调用记录都在本地，不经过第三方服务器。

### 它不能解决什么

- 不能让不支持 MCP 的客户端接入工具。
- 不能替代工作台本身的代码补全、上下文理解等核心能力。
- 不会绕过任何订阅、鉴权或模型限制。
- 它更像是在工作台旁边加一个本地 MCP 工具层，把重活交给你的自有 API。

### 工具列表

网关暴露 7 个 MCP 工具，分为三层：

**基础层**

| 工具 | 说明 |
| --- | --- |
| `smart_chat` | 多模态智能路由，支持文本和图片输入，自动匹配模型 |
| `task_executor` | 任务委托，把完整的写作/分析/总结任务交给 BYOK 模型 |

**应用层**

| 工具 | 说明 |
| --- | --- |
| `create_ppt` | 本地生成 PPTX 文件，输出到 `outputs/` |
| `toolbox` | 统一入口，支持 `action=chat \| task \| ppt` |

**技能层**

| 工具 | 说明 |
| --- | --- |
| `auto_skill` | 自动判断任务类型，路由到最合适的内置技能 |
| `list_skills` | 列出所有内置技能 |
| `run_skill` | 手动运行指定技能 |

### 内置技能

| 技能 | 触发关键词 | 说明 |
| --- | --- | --- |
| `ppt_writer` | PPT / slide / 演示文稿 | 生成幻灯片大纲和 PPTX 文件 |
| `paper_writer` | 论文 / 学术 / 润色 | 论文写作、段落润色 |
| `task_planner` | 计划 / 拆解 / 步骤 | 任务拆解和执行计划 |
| `code_reviewer` | 代码审查 / bug / 测试 | 代码审查、风险检测 |

`auto_skill` 会根据输入自动匹配，不需要手动选择。

### 快速开始

**1. 克隆并创建虚拟环境**

```bash
git clone https://github.com/TTLLDD/smart-multimodal-mcp-gateway.git
cd smart-multimodal-mcp-gateway
python -m venv .venv

# Linux / macOS
source .venv/bin/activate

# Windows
.\.venv\Scripts\activate
```

**2. 安装依赖**

```bash
pip install -r requirements.txt
```

**3. 配置 API Key**

```bash
# Linux / macOS
cp .env.example .env

# Windows
copy .env.example .env
```

编辑 `.env`，填入你的 API Key（DeepSeek、通义千问、OpenAI 等），或者启动后在网页配置页面中设置。

**4. 启动服务**

```bash
uvicorn main:app --host 127.0.0.1 --port 8010
```

- 配置页面：`http://localhost:8010/`
- MCP SSE 地址：`http://localhost:8010/sse`

**5. 接入 MCP 客户端**

在 Cursor、Windsurf、Claude Desktop 等客户端的 MCP 配置中添加：

```json
{
  "mcpServers": {
    "smart-multimodal-mcp-gateway": {
      "transport": "sse",
      "url": "http://localhost:8010/sse"
    }
  }
}
```

如果客户端不识别 `transport` 字段，可以用 `type` 替代：

```json
{
  "mcpServers": {
    "smart-multimodal-mcp-gateway": {
      "type": "sse",
      "url": "http://localhost:8010/sse"
    }
  }
}
```

### 使用示例

**自动路由任务**

在工作台中直接输入：

> 帮我做一份 8 页关于 AI 本地化工具链的 PPT

`auto_skill` 会自动识别为 PPT 任务，路由到 `ppt_writer`，模型生成结构化内容后渲染为 PPTX 文件。

**手动运行技能**

> 用 paper_writer 帮我写一段关于多模态 API 网关的论文引言

**多模态聊天**

发送一张架构图并提问：

> 帮我分析这张架构图的设计优缺点

`smart_chat` 会自动路由到视觉模型（如 qwen-vl-max 或 gpt-4o）。

### 安全说明

- API Key 只保存在本地 `.env` 和 `models.json` 中
- **不要**提交 `.env` 或 `models.json`（已在 `.gitignore` 中排除）
- 默认不记录完整请求体和 API Key
- Base64 图片原样转发，不压缩、不改写

### 技术栈

- **Python 3.10+** + **FastAPI**
- **MCP SDK** (FastMCP)
- **SSE** 协议传输
- **httpx** 异步 HTTP 调用
- **python-pptx** PPTX 渲染

---

## English

A local BYOK MCP gateway that routes high-consumption tasks from AI workbenches to your own model APIs.

Closed-source workbenches like Cursor, Windsurf, Copilot, and Trae impose quota limits and don't allow you to freely swap models. This gateway runs locally via the MCP protocol, routing heavy tasks — PPT generation, paper writing, code review, task planning — to your own configured models (DeepSeek, Qwen, OpenAI, Anthropic, etc.) without consuming your workbench quota.

### What It Solves

- **Quota anxiety**: offload long-form generation, PPT, papers, and planning to your own BYOK models.
- **Model switching**: route text and multimodal tasks through local model profiles.
- **Tool consolidation**: one MCP server exposes chat, PPT, paper, planning, and code review.
- **Data locality**: all keys, configs, and logs stay on your machine.

### What It Does Not Do

- Cannot add MCP support to clients that don't support MCP.
- Cannot replace your workbench's built-in code completion or context understanding.
- Does not bypass subscriptions, authentication, or model restrictions.

### Quick Start

```bash
git clone https://github.com/TTLLDD/smart-multimodal-mcp-gateway.git
cd smart-multimodal-mcp-gateway
python -m venv .venv && source .venv/bin/activate  # Windows: .\.venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env  # Edit with your API keys
uvicorn main:app --host 127.0.0.1 --port 8010
```

MCP endpoint: `http://localhost:8010/sse`

```json
{
  "mcpServers": {
    "smart-multimodal-mcp-gateway": {
      "transport": "sse",
      "url": "http://localhost:8010/sse"
    }
  }
}
```

### Tools

| Tool | Description |
| --- | --- |
| `smart_chat` | Multimodal routing (text + images) |
| `task_executor` | Delegate full tasks to your BYOK model |
| `create_ppt` | Generate local PPTX files |
| `toolbox` | Unified entry: `chat`, `task`, `ppt` |
| `auto_skill` | Auto-route to the best built-in skill |
| `list_skills` | List available skills |
| `run_skill` | Run a specific skill |

### Built-in Skills

| Skill | Description |
| --- | --- |
| `ppt_writer` | Slide deck generation |
| `paper_writer` | Academic writing and polishing |
| `task_planner` | Task breakdown and execution plans |
| `code_reviewer` | Code review, risks, and test gaps |

### Tech Stack

Python 3.10+ · FastAPI · MCP SDK (FastMCP) · SSE · httpx · python-pptx

### Safety

- API keys stored locally in `.env` / `models.json` — never committed.
- Full request bodies and keys are not logged by default.
- Base64 images forwarded unchanged.

---

## License

This project is licensed under the [AGPL-3.0](LICENSE) license.

## Star History

If this project helps you, consider giving it a star!
