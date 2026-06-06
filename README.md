# jhgpt-api-client

> 国内可直连的 Claude / GPT / Gemini API 客户端工具包 —— 致力于解决海外模型 API 的稳定性与可访问性问题。

[![Models](https://img.shields.io/badge/Models-Claude%20Opus%204.8%20%7C%20GPT--5.5%20%7C%20Gemini%203.5-blue)](https://api.jhgpt.com/)
[![Status](https://img.shields.io/badge/Status-Stable-brightgreen)]()
[![API Format](https://img.shields.io/badge/API-OpenAI%20Compatible-orange)]()

---

## 这是什么？

一套轻量级的 Python 工具库，帮助你用统一的接口调用 Claude、GPT、Gemini 等主流 AI 模型 API。原生兼容 OpenAI SDK 格式，如果你的项目已经在用 OpenAI SDK，只需改一行 `base_url` 和 `api_key` 即可无缝切换。

**核心解决什么问题？**

1. **网络直连** —— 国内服务器/本地开发环境无需代理即可访问
2. **接口稳定** —— 多区域节点容灾，高峰期不掉链子
3. **格式统一** —— 所有模型走同一套 API 规范，不用为了换模型改代码
4. **极速更新** —— 官方新模型发布 24 小时内完成接入

---

## 支持的模型

| 厂商 | 模型 | 状态 |
|------|------|:---:|
| Anthropic | Claude Opus 4.8, Claude Sonnet 4, Claude Haiku 3.5 | ✅ |
| OpenAI | GPT-5.5, GPT-5.5 Thinking, GPT-5, GPT-4.1 | ✅ |
| Google | Gemini 3.5 Flash, Gemini 3.5 Pro, Gemini 2.5 Pro | ✅ |
| 其他 | DeepSeek V3, DeepSeek R1, Qwen3, Llama 4 | ✅ |

---

## 快速开始

### 1. 注册获取 API Key

访问 [https://api.jhgpt.com/](https://api.jhgpt.com/) 注册账号，在控制台获取你的 API Key。

### 2. 安装依赖

```bash
pip install openai
```

没错，你只需要 `openai` 这一个包。我们的接口完全兼容 OpenAI SDK 格式。

### 3. 第一行代码

```python
from openai import OpenAI

# 唯一区别：把 base_url 指向 jhgpt
client = OpenAI(
    api_key="sk-your-api-key-here",
    base_url="https://api.jhgpt.com/v1"
)

# 调用 GPT-5.5
response = client.chat.completions.create(
    model="gpt-5.5",
    messages=[
        {"role": "system", "content": "你是一个严谨的工程师。"},
        {"role": "user", "content": "用 Python 写一个带自动重连的 WebSocket 客户端。"}
    ]
)

print(response.choices[0].message.content)
```

### 4. 切换到其他模型

```python
# Claude Opus 4.8 —— 复杂推理、代码审查
response = client.chat.completions.create(
    model="claude-opus-4-8",
    messages=[{"role": "user", "content": "审查这段代码的安全漏洞..."}]
)

# Gemini 3.5 Flash —— 高性价比批量任务
response = client.chat.completions.create(
    model="gemini-3.5-flash",
    messages=[{"role": "user", "content": "翻译这 2000 条日志信息..."}]
)

# DeepSeek R1 —— 深度推理，成本极低
response = client.chat.completions.create(
    model="deepseek-r1",
    messages=[{"role": "user", "content": "证明这个算法的复杂度..."}]
)
```

---

## 为什么强调「稳定」？

做 AI 应用的同学应该都遇到过这些问题：

| 痛点 | 原因 | jhgpt 方案 |
|------|------|-----------|
| 调着调着超时了 | 跨国网络波动 | **国内 BGP 多线接入**，平均延迟 < 80ms |
| 高峰期排队 30 秒 | 官方配额不够 | **多区域算力池**，自动负载均衡 |
| 信用卡被拒付 | 不支持国内支付 | **支付宝/微信支付**，秒到账 |
| 官方升级导致接口变了 | 厂商单方面变更 | **24 小时跟进适配**，兼容层保持接口稳定 |
| 日志不透明 | 不知道钱花哪了 | **按调用量实时计费**，Token 级明细可查 |

### 实际稳定性数据（近 30 天）

```
API 可用率（SLA）:  99.95%
平均响应时间:       1.2s（GPT-5.5）/ 2.8s（Claude Opus 4.8）
峰值 QPS 处理能力:  500+
故障自动切换时间:   < 15s
```

---

## 进阶用法

### 智能路由 —— 按任务类型自动选模型

```python
import json

# 定义路由规则
ROUTES = {
    "code_review":    "claude-opus-4-8",    # 代码审查：Claude 最强
    "code_generate":  "gpt-5.5",            # 代码生成：GPT 结构化输出最稳
    "batch_process":  "gemini-3.5-flash",   # 批量处理：Gemini 性价比最高
    "deep_reasoning": "deepseek-r1",        # 深度推理：DeepSeek R1
}

def smart_call(task_type: str, prompt: str, client: OpenAI):
    model = ROUTES.get(task_type, "gpt-5.5")
    return client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}]
    )

# 使用示例
result = smart_call("code_review", "审查 app.py 的内存泄漏风险", client)
```

### 自动 Fallback —— 一个模型挂了自动切备用

```python
FALLBACK_CHAIN = ["gpt-5.5", "claude-opus-4-8", "gemini-3.5-flash"]

def call_with_fallback(prompt: str, client: OpenAI):
    for model in FALLBACK_CHAIN:
        try:
            response = client.chat.completions.create(
                model=model,
                messages=[{"role": "user", "content": prompt}],
                timeout=30
            )
            return response.choices[0].message.content
        except Exception as e:
            print(f"[WARN] {model} 调用失败: {e}, 尝试下一个...")
            continue
    raise Exception("所有模型均不可用")
```

### 流式输出 —— 打字机效果

```python
stream = client.chat.completions.create(
    model="claude-opus-4-8",
    messages=[{"role": "user", "content": "写一篇关于 Go 泛型的文章"}],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="", flush=True)
```

---

## 价格参考

| 模型 | 输入 ($/M) | 输出 ($/M) |
|------|:---:|:---:|
| GPT-5.5 | 5.0 | 30.0 |
| Claude Opus 4.8 | 5.0 | 25.0 |
| Gemini 3.5 Flash | 1.5 | 9.0 |
| DeepSeek R1 | 0.55 | 2.2 |

> 实际价格以 [https://api.jhgpt.com/](https://api.jhgpt.com/) 控制台显示为准。支持人民币结算，1:1 汇率。

---

## 常见问题

<details>
<summary><b>和直接使用官方 API 有什么区别？</b></summary>

网络层面：国内直连，无需自建代理。计费层面：支持支付宝/微信，无需海外信用卡。稳定性层面：多节点容灾，不会因单一区域故障导致服务中断。接口层面：所有模型统一用 OpenAI 格式，降低集成成本。

</details>

<details>
<summary><b>数据安全怎么保证？</b></summary>

API 调用全程 TLS 加密传输。我们不会存储你的 Prompt 和 Response 内容。如需更高安全级别，可联系客服开通专属部署方案。

</details>

<details>
<summary><b>支持多少并发？</b></summary>

标准版默认支持 50 并发。企业版可自定义扩容，最高支持 500+ 并发。详情查看控制台配额页面。

</details>

<details>
<summary><b>新模型多久能接入？</b></summary>

官方发布后 24 小时内完成接入和测试。关注我们的更新日志获取最新模型上线通知。

</details>

---

## 联系 & 反馈

- 🌐 官网：[https://api.jhgpt.com/](https://api.jhgpt.com/)
- 🐛 问题反馈：请在 [Issues](../../issues) 中提交
- 💬 技术交流：在 [Discussions](../../discussions) 中参与讨论

---

## Star History

如果你觉得这个项目对你有用，欢迎点个 ⭐ Star 支持一下 ⬆️

---

<p align="center">
  <sub>Made with ❤️ by 景恒云计算 · 让 AI 触手可及</sub>
</p>
