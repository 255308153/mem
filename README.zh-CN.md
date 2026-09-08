<div align="center">

<img src="docs/assets/branding/hms-hero.png" alt="Holographic Memory System" width="94%">

### 面向长期推理的结构化记忆系统

<table>
  <tr>
    <td valign="middle"><strong>ShadowWeave Team</strong></td>
    <td width="74" align="center" valign="middle">
      <img src="docs/assets/branding/shadowweave-mark.png" alt="ShadowWeave" width="62">
    </td>
  </tr>
</table>

<a href="https://arxiv.org/"><img src="https://img.shields.io/badge/arXiv-coming_soon-B31B1B?style=flat-square&logo=arxiv&logoColor=white" alt="arXiv: coming soon"></a>
<img src="https://img.shields.io/badge/status-active-145DA0?style=flat-square" alt="Project status: active">

[English](README.md) · [中文](README.zh-CN.md)

</div>

---

## 项目简介

**Holographic Memory System（HMS）** 是面向 AI 应用的结构化长期记忆层。
它能够保留会话和文档、抽取可持久化事实、连接相关实体与事件，并在后续模型
调用时召回相关上下文。

HMS 适用于需要跨 Session 记忆、但不希望每次都把完整历史塞入 Prompt 的应用。

## 快速开始

使用全家桶本地包（`core/local-suite`）以内嵌 PostgreSQL 驱动 HMS：

```python
from hms import HMSEmbedded

client = HMSEmbedded(
    profile="myapp",
    llm_provider="openai",
    llm_api_key="your-api-key",
)

# 立即使用，无需手动管理服务
client.retain(bank_id="alice", content="Alice loves AI")
results = client.recall(bank_id="alice", query="What does Alice like?")
```

也可以使用 `start_server()` / `HMSClient` 显式管理服务，或用 Python SDK
（`interface/sdk/python`）直接调用 API。

## 可选的图片与视频记忆

dataplane 提供显式启用的 `openai_multimodal` 文件 parser。图片在本地校验并
规范化；视频在本地解码为有界且确定性的抽帧集合。视觉描述会被渲染为带证据的
canonical Markdown，然后进入既有 document、chunk、embedding、link 与 recall
主链。原始视频不会发送给描述 provider。

该能力默认关闭，当前 media runtime 支持矩阵限定为 PostgreSQL；Oracle 上的普通
HMS 能力不受影响，但未验证的 Oracle 多模态组合会 fail closed。真实 provider
质量需要由部署方另行验收，默认不标记为已验证。详见
[多模态运维指南](docs/multimodal_memory.md)和
[系统架构说明](docs/system_architecture_and_multimodal.md)。

## 记忆流程

```text
Retain
  -> 解析来源内容
  -> 抽取结构化记忆
  -> 解析实体与关系
  -> 存储事实、原始片段和来源信息

Recall
  -> 分析查询
  -> 执行语义、词法、图和时间检索
  -> 融合并重排证据
  -> 返回可追溯的记忆上下文
```

HMS 会为抽取后的记忆保留来源和时间元数据，方便应用检查召回内容来自哪里、
何时被观察到。

## 目录结构

```text
.
├── core/
│   ├── dataplane/     # HMS API 服务器（retain / recall 引擎）
│   ├── daemon/        # embedding worker
│   └── local-suite/   # 全家桶包（内嵌 PostgreSQL）
├── docs/
├── interface/
│   └── sdk/python/    # Python 客户端
├── scripts/
├── .env.example
├── README.md
└── README.zh-CN.md
```

## 环境配置

创建本地环境文件：

```bash
cp .env.example .env
```

配置 PostgreSQL、核心模型、Retain 模型和 Embedding Provider。不要提交填写后的
`.env` 文件。

## 核心配置

| 角色 | Provider | Model | Base URL | API Key |
| --- | --- | --- | --- | --- |
| 核心记忆推理 | `HMS_API_LLM_PROVIDER` | `HMS_API_LLM_MODEL` | `HMS_API_LLM_BASE_URL` | `HMS_API_LLM_API_KEY` |
| Retain 抽取 | `HMS_API_RETAIN_LLM_PROVIDER` | `HMS_API_RETAIN_LLM_MODEL` | `HMS_API_RETAIN_LLM_BASE_URL` | `HMS_API_RETAIN_LLM_API_KEY` |
| Embedding | `HMS_API_EMBEDDINGS_PROVIDER` | `HMS_API_EMBEDDINGS_OPENAI_MODEL` | `HMS_API_EMBEDDINGS_OPENAI_BASE_URL` | `HMS_API_EMBEDDINGS_OPENAI_API_KEY` |

核心模型与 Retain 模型可以使用同一个 OpenAI-compatible endpoint，Embedding
也可以单独使用其他 Provider 或本地模型。

## 安全说明

- 不要把 `.env`、私钥、Token 和真实凭证提交到 Git。
- 内部 API 与对外 API 使用不同密钥。
- 按用户或组织隔离 Tenant 与 Bank。
- 对外部署前检查 Gateway 的配额与限流配置。

## License

参见 [LICENSE](LICENSE)。
