# MCP（Model Context Protocol）模型上下文协议

## 一、概念引入：MCP 是什么

MCP（Model Context Protocol，模型上下文协议）是由 Anthropic 于 2024 年 11 月开源的一个开放协议，目标是为"大模型应用与外部数据源、工具"之间定义一个**统一、标准化的接入方式**。可以把 MCP 理解为"大模型世界的 USB-C 接口"：过去每个数据源/工具都要为每个大模型单独适配，而现在只要符合 MCP 协议，任何一个 MCP Server 都能被任何一个支持 MCP 的客户端（Claude Desktop、各种 IDE、LangChain 等）即插即用。

MCP 采用客户端-服务器架构，其中 Host（宿主程序，如 Claude Desktop）是用户与大模型交互的载体，Host 内嵌一个或多个 MCP Client，每个 Client 与一个 MCP Server 建立一条连接，Server 通过协议对外暴露该数据源/工具的能力。

## 二、为什么需要 MCP：它解决了什么问题

核心痛点之一叫 **"N×M 集成问题"**：假设有 N 个模型/框架（Claude、GPT、LangChain、自研 Agent），M 个工具/数据源（GitHub、数据库、Notion、Slack），如果两两对接就需要 N×M 条集成链路，且每一条都是定制开发、难以维护。而引入一个统一协议 M 之后，只需要 N 个模型适配 M、M 个工具也适配 M，总共 N+M 条链路，数量级大幅下降。

MCP 解决的第二个痛点是**上下文增强的碎片化**。大模型要发挥作用必然需要"上下文"，而上下文包括记忆、工具、知识、工作流等。此前每个应用各自发明一套工具调用格式与鉴权方式，导致工具碎片化。MCP 把"工具（Tools）"、"资源（Resources）"、"提示（Prompts）"三大能力统一封装，让上下文增强有标准可循。

## 三、整体架构讲透：三层模型与两种传输

### 3.1 三层模型：Host / Client / Server

```
┌───────────────────────────── 宿主程序 Host ─────────────────────────────┐
│   e.g. Claude Desktop / IDE / 自研应用                                    │
│                                                                          │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐                                │
│  │ MCP     │   │ MCP     │   │ MCP     │   Client 持有会话与会话能力      │
│  │ Client 1│   │ Client 2│   │ Client 3│                                │
│  └────┬────┘   └────┬────┘   └────┬────┘                                │
└───────┼─────────────┼─────────────┼─────────────────────────────────────┘
        │ stdio       │ HTTP/SSE    │ HTTP/SSE
        ▼             ▼             ▼
   ┌──────────┐  ┌──────────┐  ┌──────────┐
   │ MCP      │  │ MCP      │  │ MCP      │   Server 暴露：Tools /
   │ Server A │  │ Server B │  │ Server C │   Resources / Prompts
   └──────────┘  └──────────┘  └──────────┘
        │             │             │
      文件系统        GitHub       数据库（各自封装真正的外部能力）
```

- **Host（宿主）**：用户直接打交道的程序，负责承载 LLM 会话、渲染用户界面、管理多个 Client 的连接生命周期。
- **Client（客户端）**：Host 内与单个 Server 维持连接的工具人，负责发请求、收响应、决定是否调用工具。一个 Host 可有多个 Client。
- **Server（服务端）**：暴露能力的一方。它隔离了"外部系统"与"大模型"：Server 内部自行处理鉴权、API 调用、数据拉取，对外只暴露协议层面的标准接口。

### 3.2 两种传输 Transport

- **stdio**：Client 以子进程方式启动 Server，通过标准输入/输出交换 JSON-RPC 消息。零配置、本地安全，适合本地文件/工具类的 Server。Claude Desktop 调用本地 Server 默认走 stdio。
- **HTTP/SSE（Streamable HTTP）**：Client 通过 HTTP 请求远端的 Server，支持服务端通过 SSE（Server-Sent Events）持续推送流式消息，也支持 HTTP 轮询。适合部署在远程服务器、须鉴权的场景。2025 年 3 月协议修订后用"Streamable HTTP"取代了早期拆分式 SSE 传输。

### 3.3 基于 JSON-RPC 的消息模型

MCP 的消息全部采用 JSON-RPC 2.0 规范，支持三种消息类型：

- **Request（请求）**：`{"jsonrpc":"2.0","id":1,"method":"...","params":{...}}`，必须有 id 对应。
- **Response（响应）**：`{"jsonrpc":"2.0","id":1,"result":{...}}`，与请求 id 配对。
- **Notification（通知）**：`{"jsonrpc":"2.0","method":"notifications/...","params":{...}}`，无 id，单向通知，如工具列表变更通知 `notifications/tools/list_changed`。

协议还区分了**Client→Server** 方向的消息（如 `initialize`、`tools/call`）与 **Server→Client** 方向的消息（如 `logging/message`、`resources/updated`）。首次握手时双方交换能力声明（`initialize` 中带上 protocolVersion、capabilities），版本不兼容则协商降级。

## 四、三大能力原语：Tools / Resources / Prompts

| 原语 | 语义 | 方向 | 典型用途 |
| --- | --- | --- | --- |
| **Tools（工具）** | 可执行的函数，模型自主决定何时调用 | 通常 Client→Server：`tools/list` 列出、`tools/call` 调用 | 执行搜索、读写数据库、发邮件、调用代码解释器 |
| **Resources（资源）** | 只读的数据来源，由客户端/用户主动读取并注入上下文 | Server→Client：`resources/list`、`resources/read` | 读取文件、数据库表结构、日志、文档快照 |
| **Prompts（提示模板）** | 可复用的提示词模板，含参数占位，客户端可获取回填 | Client→Server 或 Server 内置：`prompts/list`、`prompts/get` | MCP 命令 echo、复杂工作流模板、标准话术 |

- **Tools 示例**：MCP Server 暴露一个 `search_course` 工具，模型在对话中说"帮我查一下《围棋教程》"，先经 `tools/list` 拿到工具定义（名称、参数 schema、描述），再发 `tools/call` 带参数，Server 查询数据库返回结果，模型基于结果组织自然语言回答。
- **Resources 示例**：Server 把 SQLite 数据库的 schema 暴露为资源，客户端 `resources/read` 拉回来注入上下文，模型提问时就知道有哪些表。
- **Prompts 示例**：Server 定义 `echo` 提示模板（配合官方示例 `mcp servers echo` 或 Python 示例 server），客户端 `prompts/get` 拿到模板后回填参数。典型应用是"生成某个标准文档"的工作流模板。

## 五、一次标准工具调用流程（消息时序）

以 Claude Desktop + 一个"天气查询" MCP Server 为例：

```
[Host/Client]                          [MCP Server]
      │  1. initialize (能力协商)            │
      │ ───────────────────────────────────► │
      │  2. initialize_result               │
      │ ◄─────────────────────────────────── │
      │  3. notifications/initialized       │
      │ ───────────────────────────────────► │
      │  4. tools/list                      │
      │ ───────────────────────────────────► │
      │  5. tools/list_result (返回工具schema)│
      │ ◄─────────────────────────────────── │
（模型决定调用 search_weather("北京")）
      │  6. tools/call {search_weather,北京} │
      │ ───────────────────────────────────► │
      │  7. tools/call_result {content, 温度..}│
      │ ◄─────────────────────────────────── │
（模型把结构化结果转成自然语言回复给用户 + 引用来源）
```

关键点：第 6~7 步之间，**真正的外部调用（天气 API）完全发生在 Server 内部**，Client 与模型只看到协议约定的输入输出，因此 Server 就可以自由选择数据获取方式，实现即插即用。

## 六、快速上手：用 Python 官方 SDK 写最小 MCP Server

### 6.1 依赖安装

```bash
pip install "mcp>=1.0.0"      # Anthropic 官方 Python SDK
pip install "mcp[cli]"        # 可选：附带 mcp 命令行调试工具
```

### 6.2 一个完整的最小 Server：提供一个工具 + 一个资源

```python
# server.py
from mcp.server import Server
from mcp.types import Tool, TextContent

app = Server("demo-server")

# ---------- 工具：加法 ----------
@app.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "my_add":
        a = float(arguments["a"])
        b = float(arguments["b"])
        return [TextContent(type="text", text=f"结果={a+b}")]

# ---------- 资源：一段只读文本 ----------
@app.list_resources()
async def list_resources():
    from mcp.types import Resource
    return [
        Resource(uri="file://README", name="README 资源", mimeType="text/plain")
    ]

@app.read_resource()
async def read_resource(uri: str):
    from mcp.types import TextResourceContents
    return [TextResourceContents(uri=uri, mimeType="text/plain", text="这是一个演示资源。")]
```

### 6.3 用官方 CLI 调试

```bash
# 用 mcp 命令以 SSE（HTTP/JSONRPC）方式启动
mcp run server.py

# 开发调试客户端，直接在当前终端交互调用工具
mcp dev server.py
```

### 6.4 在 Claude Desktop 中配置

编辑 `claude_desktop_config.json`（Windows 在 `%APPDATA%\Claude\claude_desktop_config.json`）：

```json
{
  "mcpServers": {
    "demo": {
      "command": "python",
      "args": ["C:/path/to/server.py"]
    }
  }
}
```

重启 Claude Desktop 后，在 MCP 面板看到 `demo` Server 及其工具即可在对话中直接使用：模型会自行决定何时调用 `my_add`。其它支持 MCP 的客户端（如 Cursor、VS Code、Continue 等）配置方式类似，只是配置文件位置与字段略有差异。

## 七、与主流框架的集成方式

- **LangChain / LlamaIndex**：两者都有 MCP 适配器。LangChain 通过 `langchain-mcp-adapters` 把 MCP Server 的 Tools 转换为 LangChain Tool/`ChatOpenAI` 可用的 `create_openai_tools_agent`；LlamaIndex 通过 `llama-index-integrations-mcp` 的 `McpToolSpec` 导入工具。
- **各类 Agent 框架**：AutoGen、CrewAI、OpenAI Agents SDK 等都提供了 MCP 客户端接入能力（如 `mcp` 的 `ClientSession` 配合 `stdio_client`），可以把任意 MCP 工具挂进自家 Agent 的工具箱。
- **自研应用**：直接用官方 SDK 的异步客户端 `BaseMCPClient`/`ClientSession` 连接 server，便可在你自己的推理循环中枚举并调用任意标准工具。

## 八、当前生态现状

- **官方组织**：Anthropic 维护的核心协议仓库 `modelcontextprotocol/modelcontextprotocol`（协议规范与中文文档）以及 Python (`modelcontextprotocol/python-sdk`) 与 TypeScript (`modelcontextprotocol/typescript-sdk`) 两大官方 SDK。
- **协议版本**：目前迭代到 `2025-06-18` 附近，支持 Streamable HTTP Transport、OAuth 2.0 鉴权、采样（sampling）、日志等；各 SDK 以 `initialize` 中的 protocolVersion 字段做版本协商。
- **官方 servers 仓库**：`modelcontextprotocol/servers` 汇集了大量常用官方/社区 Server，如 Filesystem（文件系统）、GitHub、Postgres、Brave Search、SQLite、Fetch（网页抓取）、Memory（记忆服务）等，可直接配置使用。
- **社区与工具链**：Inspector（可视化调试工具）集成在 SDK 中；`mcp.run`、Smithery 等第三方注册中心让 Server 可一键发现与安装。

## 九、局限性与安全考虑

1. **工具权限放大风险（重点）**：MCP 把"接入成本"降到了极低，意味着用户在对话里不经意的一句话就可能触发 Server 执行"写文件、发邮件、改配置"等真实操作——这实质上是**把权限委托给了模型**。模型一旦被越狱/注入恶意提示，就能借机调用权限过大的工具。务必对每个 Server 坚持**最小权限原则**：只给必要的能力、关键操作加确认二次弹窗、在独立低权限账号/容器里跑高危 Server。
2. **信息泄露**：模型可能把不该吐的（token、个人数据）当成工具入参发送，或经 Server 读到的敏感内容泄露给用户提示词注入方。需要对资源 URI 做白名单，对回传内容做敏感信息过滤。
3. **资源/连接管理**：同时挂几十个 Server 会占用本地资源与模型上下文长度（每个工具 schema 都要进上下文），需按需裁剪。
4. **缺少统一治理**：协议本身不强制做审计、配额与限流，这些需要在 Host 层自己实现。

## 十、展望

MCP 正从"工具接入协议"演进为大模型应用与外部世界的"通用上下文总线"：Anthropic 与 OpenAI 之间关于工具协议（MCP vs 2025 年推出的 Agentic Commerce Protocol/A2A）存在竞争，但从 Tool Use 到跨 AI 的 Agent 互操作（A2A 用于智能体之间通信），整个行业在朝"开放、标准、最小可信权限"的方向收敛。作为学习者，掌握 MCP 等于掌握了未来"模型即插即用"基础设施的钥匙，建议优先熟读协议核心五种原语（Tools、Resources、Prompts、Sampling 与 Roots）并动手写一个 Server 体验完整链路。