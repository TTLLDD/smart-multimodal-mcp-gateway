# Smart Multimodal MCP Gateway

Languages: [中文](#中文说明) | [English](#english)

## 中文说明

### 你的闭源 AI 工具不能自由切 API 吗？

你会有订阅焦虑、额度焦虑、积分焦虑吗？

你是不是已经有自己的 GLM、Mimo、Qwen、GPT-4o 或其他 OpenAI-compatible API Key，却被某些闭源 AI 客户端锁在固定订阅、固定模型、固定额度里？

你会不会觉得很多闭源 AI 工作台很好用，但每一次写长文、做 PPT、拆任务、改论文、总结资料，都不一定值得消耗昂贵的客户端内置额度？

**Smart Multimodal MCP Gateway** 就是为这个痛点做的：一个本地运行的 BYOK MCP SSE 网关。只要你的闭源 AI 客户端支持接入 MCP，它就可以把高消耗任务交给你自己配置的模型 API，而不是只能依赖客户端内置的模型渠道。

简单说：

```text
闭源 AI 客户端负责：入口、上下文、MCP 调度
本地网关负责：工具箱、技能路由、模型档案切换
你的 BYOK 模型负责：长文、PPT、论文、规划、代码审查等重活
```

你只需要在支持 MCP SSE 的客户端里导入一个 MCP Server，就可以得到一个本地工具箱：

- 客户端继续负责界面、项目入口和上下文
- 长内容生成交给你自己的文本模型
- 截图、架构图、图片理解交给你自己的多模态模型
- PPT、论文、任务规划、代码审查通过内置 skills 自动路由
- 多个 API 渠道统一放进模型档案，网页上切换和测试

它特别适合这些场景：

- 想保留闭源 AI 客户端的项目入口和对话体验
- 想减少内置模型在高消耗任务上的使用频率
- 想用自己的 API Key 承担 PPT、论文、任务规划、代码审查
- 想把多个模型档案统一成一个 MCP 工具箱
- 想把常用工作流做成可开源、可迁移的本地能力包

例如：Qoder 本身可以自定义模型；QoderWork 这类限制更强、不能自由切 BYOK/API 的闭源客户端，才更接近这个项目要解决的典型痛点。

本项目不隶属于任何闭源 AI 客户端、OpenAI、DeepSeek、阿里、智谱、小米或任何模型厂商，也不内置任何真实 API Key。

### 一分钟看懂

| 你的痛点 | 这个项目怎么帮你 |
| --- | --- |
| 闭源客户端不能自由切 API | 通过 MCP 工具层接入本地 BYOK 模型 |
| 订阅、额度、积分压力大 | 把长文、PPT、论文、规划等重活分流到 BYOK 模型 |
| 日常高 token 任务成本高 | 低价值或高消耗任务可走自有 API |
| 多个模型切换麻烦 | 用模型档案统一管理文本模型和多模态模型 |
| MCP 工具太分散 | 一个 MCP Server 内置聊天、PPT、论文、规划、代码审查 |
| 想开源给别人用 | 不内置密钥，配置示例、README、忽略规则都已准备好 |

### 它能解决什么

- **额度焦虑**：长内容生成、PPT、论文、规划等任务可转交给你的 BYOK 模型。
- **模型切换麻烦**：通过模型档案自动路由文本模型和多模态模型。
- **工具太分散**：一个 MCP server 内置 `auto_skill`、PPT、论文、任务规划、代码审查等能力。
- **开源迁移困难**：配置、README、MCP JSON 示例和 `.gitignore` 已整理好。

### 它不能解决什么

- 它不能让不支持 MCP 的客户端凭空接入工具。
- 它不是任何闭源客户端的官方 BYOK 替代方案。
- 它不会绕过任何订阅、鉴权或模型限制。
- 它更像是在闭源客户端旁边加一个本地 MCP 工具层，把重活交给你的自有 API。

### 能力

这个项目只需要导入一个 MCP Server，但内部提供一组工具：

- `auto_skill`：自动判断任务类型，并路由到最合适的内置 skill
- `toolbox`：统一入口，支持 `action=chat | task | ppt`
- `smart_chat`：文本/多模态聊天，自动路由到模型档案
- `task_executor`：把完整任务交给你的 BYOK 文本模型
- `create_ppt`：生成本地 `.pptx` 文件，输出到 `outputs/`
- `list_skills`：列出内置 skills
- `run_skill`：手动运行指定内置 skill

内置 skills：

- `ppt_writer`：PPT / 演示文稿
- `paper_writer`：论文写作、论文润色、学术段落
- `task_planner`：任务拆解、执行计划
- `code_reviewer`：代码审查、风险和测试缺口

### 安装

```powershell
python -m venv .venv
.\.venv\Scripts\activate
python -m pip install -r requirements.txt
copy .env.example .env
```

然后编辑 `.env`，或者启动服务后打开网页配置页。

### 运行

```powershell
python -m uvicorn main:app --host 127.0.0.1 --port 8010
```

配置页：

```text
http://localhost:8010/
```

MCP SSE 地址：

```text
http://localhost:8010/sse
```

### MCP 配置

优先使用：

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

如果客户端不认 `transport`，可尝试：

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

自动路由任务：

```json
{
  "task": "帮我做一份 8 页关于 AI 本地工具箱的 PPT",
  "pages": 8
}
```

手动运行 skill：

```json
{
  "skill": "paper_writer",
  "task": "帮我写一段关于多模态 API 网关的论文引言"
}
```

统一工具入口：

```json
{
  "action": "ppt",
  "topic": "AI 本地工具箱如何减少内置模型消耗",
  "pages": 8,
  "style": "简洁蓝白"
}
```

`auto_skill` 的内部路由规则：

- PPT / slide / 演示文稿 -> `ppt_writer`
- 论文 / 学术写作 / 润色 -> `paper_writer`
- 代码审查 / 找 bug / 测试风险 -> `code_reviewer`
- 计划 / 拆解 / 步骤 -> `task_planner`
- 其他任务 -> `task_executor`

### 安全说明

- API Key 只保存在本地 `.env` / `models.json`
- 不要提交 `.env` 或 `models.json`
- 默认不记录完整请求体和 API Key
- Base64 图片会原样转发，不压缩、不改写

## English

### Is Your Closed AI Client Locked To Its Own API?

Do subscriptions, quotas, or credit limits make you hesitate before asking for long outputs?

Do you already have GLM, Mimo, Qwen, GPT-4o, or another OpenAI-compatible API key, but your closed AI client does not let you freely switch to your own API?

Do some closed AI workbenches feel useful, but too costly for every long answer, slide deck, paper edit, task breakdown, or summary?

**Smart Multimodal MCP Gateway** is built for that pain point: a local BYOK MCP SSE gateway. If your closed AI client supports MCP, this gateway can route high-consumption tasks to your own model APIs instead of forcing everything through the client's built-in model channel.

In short:

```text
The closed AI client handles: entrypoint, context, MCP orchestration
The local gateway handles: tools, skill routing, model profile switching
Your BYOK models handle: long text, PPT, papers, planning, code review, and other heavy work
```

Import one MCP server into any MCP SSE-compatible client, and you get a local toolbox:

- keep the client UI, project entrypoint, and context
- send long-form generation to your own text model
- send screenshots, diagrams, and image understanding to your own multimodal model
- route PPT, paper writing, planning, and code review through bundled skills
- manage multiple API channels through local model profiles and a web UI

It is especially useful when you want to:

- keep a closed AI client's project and chat experience
- reduce built-in model usage on high-consumption tasks
- use your own API keys for PPT, paper writing, planning, and code review
- manage multiple model profiles through one MCP server
- package repeatable workflows as local, open-source-friendly skills

For example: Qoder itself can be configured with custom models; more restrictive clients such as QoderWork, where BYOK/API switching is limited, are closer to the typical pain point this project targets.

This project is not affiliated with any closed AI client, OpenAI, DeepSeek, Alibaba, Zhipu, Xiaomi, or any model provider. It does not include API keys.

### At A Glance

| Pain Point | How This Helps |
| --- | --- |
| Closed clients cannot freely switch APIs | Add a local BYOK model layer through MCP tools |
| Subscriptions, quotas, or credits feel tight | Offload long text, PPT, papers, and planning to BYOK models |
| Routine high-token work is costly | Route lower-value or high-consumption tasks to your own API |
| Switching models is annoying | Manage text and multimodal models through local profiles |
| MCP tools are scattered | One MCP server exposes chat, PPT, paper, planning, and code review |
| You want to open source it | No embedded keys; examples, README, and ignore rules are ready |

### What It Helps With

- **Quota anxiety**: offload long-form generation, PPT, paper writing, and planning to your own BYOK models.
- **Model switching**: route text and multimodal tasks through local model profiles.
- **Scattered tools**: one MCP server exposes `auto_skill`, PPT, paper writing, planning, and code review.
- **Open-source portability**: README, MCP config example, and `.gitignore` are included.

### What It Does Not Do

- It cannot add MCP support to clients that do not support MCP.
- It is not an official BYOK replacement for any closed AI client.
- It does not bypass subscriptions, authentication, or model restrictions.
- It adds a local MCP tool layer beside your closed client, while your own APIs do the heavy work.

### Features

You only need to import one MCP server. The server exposes a small local toolbox:

- `auto_skill`: automatically route a task to the best bundled skill
- `toolbox`: unified entrypoint, with `action=chat | task | ppt`
- `smart_chat`: route text/image chat to configured model profiles
- `task_executor`: delegate a complete text task to your BYOK model
- `create_ppt`: generate a local `.pptx` file under `outputs/`
- `list_skills`: list bundled local skills
- `run_skill`: run a bundled local skill

Bundled skills:

- `ppt_writer`: PPT / slide decks
- `paper_writer`: academic writing and polishing
- `task_planner`: task breakdown and execution plans
- `code_reviewer`: code review, risks, and missing tests

### Install

```powershell
python -m venv .venv
.\.venv\Scripts\activate
python -m pip install -r requirements.txt
copy .env.example .env
```

Then edit `.env`, or open the web config page after starting the service.

### Run

```powershell
python -m uvicorn main:app --host 127.0.0.1 --port 8010
```

Config page:

```text
http://localhost:8010/
```

MCP SSE URL:

```text
http://localhost:8010/sse
```

### MCP Config

Recommended:

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

Some clients use `type` instead of `transport`:

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

### Examples

Automatically route a task:

```json
{
  "task": "Create an 8-page PPT about how a local AI toolbox reduces built-in model usage",
  "pages": 8
}
```

Run a bundled skill:

```json
{
  "skill": "paper_writer",
  "task": "Write an introduction paragraph about a multimodal API gateway"
}
```

Use the unified toolbox:

```json
{
  "action": "ppt",
  "topic": "How a local AI toolbox reduces built-in model usage",
  "pages": 8,
  "style": "clean blue-white"
}
```

`auto_skill` routes internally:

- PPT / slide requests -> `ppt_writer`
- paper / academic writing -> `paper_writer`
- code review -> `code_reviewer`
- planning / task breakdown -> `task_planner`
- everything else -> `task_executor`

### Safety

- API keys are stored locally in `.env` / `models.json`
- Do not commit `.env` or `models.json`
- Full request bodies and API keys are not logged by default
- Base64 image payloads are forwarded unchanged
