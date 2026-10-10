# 接口交互与 API：LLM 应用的对外接口设计

## 一、概念引入

传统后端 API 通常"给定输入，确定性计算，一次性返回结果"。而 LLM 应用的接口完全不同：模型是**生成式**的，同样的 Prompt 每次输出都可能不同；输出是**流式**的，需要像打字机一样逐 token 推给用户；每次调用都要按 Token 计费，**成本敏感**；并且接口背后是真金白银的第三方模型调用，需要面对超时、限流、重试、鉴权等一系列工程问题。这篇笔记把 LLM 应用接口设计中最核心的五个话题讲透：结构化输出、Function Calling、流式输出（SSE）、上下文管理、错误处理与鉴权。

## 二、LLM 应用接口设计的特殊性

| 对比维度 | 传统 API | LLM API |
| --- | --- | --- |
| 输出性质 | 确定性、可严格断言 | 生成式、非确定性（同输入不同输出） |
| 输出格式 | 强类型字段（JSON/XML schema） | 自然语言为主，格式脆弱 |
| 响应方式 | 一次性同步返回 | 通常流式逐 token 返回（体验必须） |
| 时延 | 毫秒级可预期 | 秒级且波动大 |
| 成本 | 固定（硬件/带宽） | 随 Token 数线性增长，流式也计费 |
| 依赖 | 数据库/业务逻辑 | 外部第三方模型服务（有配额、有故障） |
| 错误特征 | 状态码齐全 | 有 429 限流、超时、内容审核拦截等特殊错误 |

设计 LLM 接口的核心思路：**把非确定性控制起来**（用结构化输出），**把长时延藏起来**（用流式），**把成本和故障管起来**（用缓存、重试、熔断、配额）。

## 三、结构化输出：让模型输出可被程序稳定消费

### 3.1 为什么需要结构化输出

模型输出是自由文本，下游程序（字段提取、入库、触发工作流）如果直接解析自然语言，会因格式漂移而崩溃：少一个引号、多一个换行、把数字写成"三"，都会让 `json.loads` 失败。因此必须把模型的输出"约束"成固定的 JSON 结构，保证下游解析稳定、可自动化、可验收。

### 3.2 实现途径与对比

| 途径 | 原理 | 可靠性 | 适用场景 |
| --- | --- | --- | --- |
| **约束 Prompt** | 在 Prompt 里"请只输出 JSON 并满足以下 schema"，配合正则兜底解析 | 低-中，仍会漂移 | 快速原型、非关键路径 |
| **JSON Schema / response_format** | 在 API 请求里声明 `response_format={"type":"json_object"}` 或提供完整 JSON Schema | 中-高（模型端软约束） | OpenAI/通义等主流 API 的标准做法 |
| **Function Calling / 工具模式** | 把"输出一个结构体"声明成一个函数，模型返回参数 JSON | 高，输出即参数 | 需要同时执行动作的场景（下一步会细讲） |
| **开源受限解码（Constrained Decoding）** | 在推理时逐 token 用有限状态机过滤非法 token（如 `outlines`、vLLM `guided_*`、Llama.cpp grammar） | 最高（硬约束，保证合法语法） | 本地自部署模型、对格式零容忍的生产场景 |

以 OpenAI 兼容接口为例的响应式 JSON：

```python
resp = client.chat.completions.create(
    model="gpt-4o",
    response_format={"type": "json_object"},
    messages=[{"role": "user", "content": "返回 {"name":..., "score":...} 格式的课程评价"}],
)
data = json.loads(resp.choices[0].message.content)
```

## 四、Function Calling（函数调用）调用流程详解

Function Calling 的本质：把模型输出约束为"对我的某个函数的一次调用声明"，由我们的代码真正执行函数，再把结果回填给模型继续推理。典型流程如下：

### 4.1 消息时序

```
[客户端/主机代码]                          [模型服务]
      │ 1. 发带 tools 声明的 messages        │
      │   (tools=[{name,params schema,desc}])│
      │ ────────────────────────────────────► │
      │ 2. 返回 tool_calls=                 │
      │   [{id,function:{name,arguments}}]  │
      │ ◄─────────────────────────────────── │
（代码收到参数，真实执行本地/远端函数）
      │ 3. 回填：追加一条 role=tool 消息     │
      │   (tool_call_id 与第2步对应)         │
      │ ────────────────────────────────────► │
      │ 4. 模型依据工具结果继续生成最终答复   │
      │ ◄─────────────────────────────────── │
```

### 4.2 最小可跑代码（OpenAI 风格 / 兼容接口）

```python
import json
from helper import get_openai_client  # 你的客户端初始化

def get_weather(city: str) -> str:
    return f"北京当前 22℃，多云。"

tools = [{
    "type": "function",
    "function": {
        "name": "get_weather",
        "description": "查询指定城市的当前天气",
        "parameters": {
            "type": "object",
            "properties": {"city": {"type": "string"}},
            "required": ["city"],
        },
    },
}]

messages = [{"role": "user", "content": "北京天气怎么样？"}]
resp = get_openai_client().chat.completions.create(
    model="gpt-4o", messages=messages, tools=tools
)

msg = resp.choices[0].message
if msg.tool_calls:
    for tc in msg.tool_calls:
        args = json.loads(tc.function.arguments)
        result = get_weather(args["city"])
        # 关键：回填 tool 角色消息，再让模型基于结果作答
        messages.append(msg)
        messages.append({
            "role": "tool",
            "tool_call_id": tc.id,
            "content": result,
        })
    final = get_openai_client().chat.completions.create(
        model="gpt-4o", messages=messages, tools=tools
    )
    print(final.choices[0].message.content)
```

注意点：`tool_call_id` 必须与模型返回的 id 一一对应；`messages.append(msg)` 里要保留原始 `tool_calls` 字段，多个 tool_call 可以并行执行后一次性回填。

## 五、流式输出：SSE 原理与 EventSource

### 5.1 SSE 工作原理

SSE（Server-Sent Events）是 HTML5 标准，基于普通 HTTP，服务端把响应 Content-Type 设为 `text/event-stream`，然后持续发送形如 `data: {...}\n\n` 的事件块。它与 WebSocket 的区别：SSE 是**单向**（服务端→客户端）的，但天然支持 HTTP、自动重连、简单可靠，正好匹配"模型逐 token 推给前端"的场景。

一份 SSE 流长这样：

```text
HTTP/1.1 200 OK
Content-Type: text/event-stream

data: {"delta":"你"}

data: {"delta":"好"}

data: [DONE]
```

### 5.2 前端 EventSource 解析

```javascript
const es = new EventSource("/api/chat/stream");
es.onmessage = (ev) => {
  if (ev.data === "[DONE]") { es.close(); return; }
  const obj = JSON.parse(ev.data);
  document.getElementById("out").textContent += obj.delta;
};
es.onerror = () => { /* 网络中断会自动重连，这里可加提示 */ };
```

## 六、代码实践：FastAPI 后端流式输出 + EventSource

### 6.1 后端（FastAPI + StreamingResponse）

```python
# app.py
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
from openai import OpenAI
import asyncio

app = FastAPI()
client = OpenAI()  # 使用 OPENAI_API_KEY 环境变量

async def token_stream(prompt: str):
    stream = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}],
        stream=True,   # 开启流式
    )
    for chunk in stream:
        delta = chunk.choices[0].delta
        if delta and delta.content:
            tok = delta.content
            # 每个 token 作为一条 SSE 事件
            yield f"data: {__import__('json').dumps({'delta': tok}, ensure_ascii=False)}\n\n"
    yield "data: [DONE]\n\n"

@app.post("/api/chat/stream")
async def chat(payload: dict):
    return StreamingResponse(
        token_stream(payload.get("prompt", "")),
        media_type="text/event-stream",
    )
```

### 6.2 前端（原生 EventSource + fetch 传参）

```html
<!-- index.html -->
<textarea></textarea>
<button onclick="ask()">发送</button>
<div></div>
<script>
async function ask() {
  // EventSource 只能 GET，因此用 fetch 先建任务或改用 POST+SSE：这里用 GET 简化演示
  const prompt = document.getElementById("q").value;
  const es = new EventSource(`/api/chat/stream?prompt=${encodeURIComponent(prompt)}`);
  document.getElementById("out").textContent = "";
  es.onmessage = (ev) => {
    if (ev.data === "[DONE]") { es.close(); return; }
    document.getElementById("out").textContent += JSON.parse(ev.data).delta;
  };
}
</script>
```

运行：`pip install fastapi uvicorn openai` 后执行 `uvicorn app:app --port 8000`，浏览器打开 `http://localhost:8000`，即可看到逐字打字的对话效果。若想用 POST，常规做法是后端把消息 push 进一个 uuid 任务队列 + EventSource 订阅该队列。

## 七、长时间/多轮上下文管理

- **窗口（Window）**：模型有硬性上下文上限（如 128K），超限会直接报错（如 `maximum context length exceeded`）。在设计接口时做输入长度校验与降级。
- **截断（Truncation）**：简单粗暴丢弃最旧的轮次，缺点是丢信息。优先丢弃 `role=system` 之外的中间轮，保留 system 与最新消息。
- **摘要（Summarization）**：当历史超过阈值，先用模型把旧对话浓缩成一段摘要替换掉，再拼接新对话，兼顾长度与信息量。
- **缓存复用（Caching）**：对"长 system + 稳定的工具定义 + 历史前缀"做 prompt cache（例如 Anthropic、DeepSeek、OpenAI 都有相关计费折扣），命中缓存可大幅降低延迟与成本。实现上把不变的公共部分放在消息最前面、按"前缀"组织。

## 八、错误处理：超时、限流、重试与熔断

| 错误类型 | 症状 | 处理策略 |
| --- | --- | --- |
| **超时（Timeout/TimeoutError，如 524）** | 请求挂起 | 设合理的读超时（如 60s+ 流式更长），对"流式"用逐 chunk 超时，避免整连接超时误杀 |
| **限流 429（RateLimit）** | HTTP 429 | 结构化解偶：记录 `Retry-After` 头，指数退避重试 |
| **5xx / 网络** | 500/502/503 | 指数退避重试，重试次数 3 次上限 |
| **熔断（Circuit Breaker）** | 上游持续故障 | 快速失败（fail fast）而不是持续打爆上游；半开状态探测恢复 |

指数退避的最小实现：

```python
import time, random

def call_with_backoff(fn, max_retries=3):
    delay = 1.0
    for i in range(max_retries):
        try:
            return fn()
        except Exception as e:
            if i == max_retries - 1:
                raise
            delay = min(delay * 2, 30)
            time.sleep(delay + random.uniform(0, 1))  # 加抖动防惊群
```

注意：**429 重试只对幂等操作安全**；如果工具调用有副作用（下单、发消息），重试前先确认是否已执行，否则可能重复扣费/重复下单。

## 九、接口隔离与鉴权

- **API Key / Bearer**：对外接口一律走 HTTP 头 `Authorization: Bearer <api_key>`，不要把模型厂商的 key 直接暴露给前端；由后端持有"主 key"、对下游签发"受限 key"。
- **成本配额（Quota）**：在后端对每个账号/租户做 Token 与金额额度控制，超限返回 402/429；为不同用户限制最大并发与最大单次 Token。
- **内容审核（Content Moderation）**：请求与响应双侧过审（模型端原生审核 + 后端关键词/分类模型兜底），敏感内容返回 400 或替换安全文案。
- **运维侧**：统一网关做限流（令牌桶/漏桶）、日志审计（记录每次调用的用户、token 数、成本）、灰度发布。

## 十、常见坑与最佳实践汇总

- **坑：让前端直接调模型接口** → 密钥泄露、成本失控、无审计。必须经后端代理。
- **坑：`json.loads` 直接解析模型输出** → 加入"重试修复 JSON"或用结构化输出/受限解码。
- **坑：流式只按整响应判断超时** → 逐 token/chunk 超时才是流式的正确姿势。
- **坑：无脑重试非幂等调用** → 同一请求加幂等键（idempotency key）。
- **坑：上下文无管理直接塞满** → 用窗口/截断/摘要/缓存四位一体，优先前缀缓存省成本。
- **坑：并发保护缺失** → 限流、熔断、Signal 与背压（Backpressure）一起上。
- **最佳实践清单**：接口返回结构统一（status/code/data）；日志全链路 trace_id；成本按产品维度拆分报表；模型与提示词版本化，升级可回滚；内部基准评测（Golden Set）防止"升级劣化"。

## 十一、小结

LLM 应用接口的本质矛盾是"模型的非确定性、流式长时延、按 Token 计费"与"程序需要稳定、快速、低成本的输出"之间的张力。解决之道不外乎：用结构化输出/Function Calling 约束结果，用 SSE + 流式体验藏住时延，用窗口/截断/摘要/缓存管理上下文，用超时/退避重试/熔断消化上游不稳定，用鉴权/配额/审核锁住安全与成本。把这五件事做扎实，LLM 接口就能从"玩具"变成"可靠的生产级服务"。
