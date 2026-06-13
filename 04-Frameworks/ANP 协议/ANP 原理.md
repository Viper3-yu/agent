---
title: ANP 协议（Agent Network Protocol）
aliases: [ANP, Agent Network Protocol]
tags:
  - protocol
  - topic/agent
  - status/reviewed
created: 2026-06-12
updated: 2026-06-13
spec_url: https://arxiv.org/html/2505.02279v1
maintainer: 社区驱动（agent-network-protocol 组织）
transport: HTTP / DIDComm
---

# ANP 协议（Agent Network Protocol）

> 社区驱动的开放协议，专注**跨组织、去中心化** Agent 网络中的发现与安全协作。基于 W3C DID 和 JSON-LD 标准构建。

## 协议概览

| 项目 | 详情 |
|------|------|
| 全称 | Agent Network Protocol (ANP) |
| 提出者 | 社区驱动（agent-network-protocol） |
| 规范链接 | [arXiv 论文](https://arxiv.org/html/2505.02279v1) |
| 传输层 | HTTP / DIDComm |
| 消息格式 | JSON-LD |
| 方向 | 水平（Agent↔Agent，跨组织） |

## 设计动机

MCP 和 A2A 解决了"组织内部"的 Agent 协作问题。但当 Agent 需要：

- **跨组织**：A 公司 Agent 调 B 公司 Agent 付费能力
- **去中心化**：没有中央目录服务，Agent 通过网络自主发现
- **强身份**：用加密学身份（DID）而非 OAuth 账户

MCP/A2A 就力不从心了。ANP 在 arXiv 2505.02279 提出，定位是 **Agent 互联网的 HTTP**。

## 核心架构

```
┌───────────────┐  DID 解析   ┌───────────────┐
│  Agent A      │ ──────────> │  Agent B      │
│  (公司甲)      │   JSON-LD   │  (公司乙)      │
│               │ <────────── │               │
│  DID: did:web:│   DIDComm   │  DID:did:web: │
│  a.example    │   加密握手   │  b.example    │
└───────────────┘             └───────────────┘
       ▲                             ▲
       │  ┌─────────────────────┐    │
       └─│ 区块链 / DID Registry │────┘
          └─────────────────────┘
```

三层结构：

1. **身份层**：每个 Agent 有 DID（去中心化标识符）
2. **描述层**：JSON-LD 描述 Agent 能力和接口
3. **通信层**：DIDComm 加密消息交换

## 关键概念

- 基于去中心化标识符（DIDs, W3C 标准）
- JSON-LD 图谱描述能力（语义网标准）
- 跨组织 Agent 发现（无需中央注册中心）
- 开放网络场景下的安全协作
- 端到端加密（DIDComm 协议）

## 消息类型

ANP 在 HTTP 之上定义 RESTful 端点：

| 端点 | 方法 | 用途 |
|------|------|------|
| `/.well-known/did.json` | GET | Agent 的 DID 文档 |
| `/.well-known/anp.json` | GET | Agent 的能力描述符（JSON-LD） |
| `/anp/auth` | POST | 发起 DIDComm 握手 |
| `/anp/invoke` | POST | 调用 Agent 能力（需加密信封） |
| `/anp/subscribe` | WebSocket | 订阅 Agent 事件 |

### DID 文档示例

```json
{
  "id": "did:web:research.example.com",
  "verificationMethod": [{
    "id": "did:web:research.example.com#key-1",
    "type": "EcdsaSecp256k1VerificationKey2019",
    "publicKeyJwk": { "...": "..." }
  }],
  "service": [{
    "id": "did:web:research.example.com#anp",
    "type": "AgentNetworkProtocol",
    "serviceEndpoint": "https://research.example.com/anp"
  }]
}
```

### 能力描述符（JSON-LD）

```json
{
  "@context": "https://anp.org/schema/v1",
  "@type": "Agent",
  "name": "Medical Research Agent",
  "capabilities": [{
    "@type": "Capability",
    "name": "literature_search",
    "input": "MedicalScholarQuery",
    "output": "PaperList",
    "pricing": { "model": "per_call", "amount": "0.10 USD" }
  }]
}
```

## 交互流程

```
1. Agent A 已知 Agent B 的 DID（来自声誉系统/目录/手动配置）
2. Agent A → GET https://b.example/.well-known/did.json   （解析公钥）
3. Agent A → GET https://b.example/.well-known/anp.json   （发现能力）
4. Agent A → POST /anp/auth                               （DIDComm 握手）
5. Agent A ↔ Agent B 完成端到端加密通道建立
6. Agent A → POST /anp/invoke                             （加密调用）
7. Agent B → 加密返回结果
```

## 安全考量

- **零信任**：每个 Agent 都有独立加密身份，无中央授权中心
- **端到端加密**：DIDComm 协议确保中间节点无法窃听
- **可验证凭证（VC）**：Agent 能力可由第三方颁发凭证证明
- **声誉系统**：去中心化评分协议（如 ERC-8004 类提议）
- **防女巫攻击**：每个 DID 需质押或工作量证明
- **数据主权**：调用方可指定数据驻留区域（GDPR 合规）

## 与其他协议的关系

ANP 解决的是**跨组织、开放网络**中的 Agent 互操作，比 A2A 的范围更广：

| 维度 | A2A | ANP |
|------|-----|-----|
| 信任边界 | 组织内 | 跨组织 |
| 身份 | OAuth / API Key | DID / VC |
| 发现 | Agent Card + 中心目录 | DID + 去中心化 |
| 加密 | TLS（传输层） | DIDComm（端到端） |
| 计费 | 自定义 | 链上 / 智能合约 |
| 适用 | 企业内 Agent 协作 | Agent 互联网 / 市场 |

```
场景复杂度 →
  低  ────────────────────────────────────>  高
  │           │              │              │
组织内部    跨组织受限网络    开放 Agent 市场
  │           │              │              │
  └── MCP ────┴── A2A ───────┴───── ANP ────┘
```

## 采纳建议

适合跨组织 Agent 市场场景：
- 多家供应商提供付费 Agent 能力
- 需要可验证的信誉和审计
- 数据主权 / 合规要求严格

大多数项目用 [[../MCP 协议/MCP 原理|MCP]] + [[../A2A 协议/A2A 原理|A2A]] 即可。只有明确需要去中心化特性时才引入 ANP。

## 代码示例

```python
# ANP 客户端：用 DID 文档发现并调用远程 Agent
import httpx, json
from didcomm import pack_message, unpack_message

async def call_remote_agent(target_did: str, capability: str, params: dict):
    # 1. 解析 DID 文档
    domain = target_did.replace("did:web:", "")
    did_doc = (await httpx.AsyncClient().get(
        f"https://{domain}/.well-known/did.json"
    )).json()
    public_key = did_doc["verificationMethod"][0]["publicKeyJwk"]
    endpoint = did_doc["service"][0]["serviceEndpoint"]

    # 2. 获取能力描述符
    desc = (await httpx.AsyncClient().get(
        f"https://{domain}/.well-known/anp.json"
    )).json()

    # 3. DIDComm 加密握手（此处简化）
    encrypted = pack_message(
        plaintext={"capability": capability, "params": params},
        to=public_key,
    )

    # 4. 调用加密端点
    response = await httpx.AsyncClient().post(
        f"{endpoint}/anp/invoke",
        content=encrypted,
        headers={"Content-Type": "application/didcomm-encrypted+json"},
    )
    return unpack_message(response.json())
```

## 时间线

| 时间 | 事件 |
|------|------|
| 2025-05 | arXiv 论文 2505.02279 发布 |
| 2025-08 | 社区成立 agent-network-protocol 组织 |
| 2025-12 | v0.1 草案，参考实现 in Python |
| 2026-Q2 | v0.2 引入 ERC-8004 类链上声誉 |
| 2026-Q4 | v1.0 目标 |

## 参考资料

- [arXiv 论文](https://arxiv.org/html/2505.02279v1)
- [W3C DID 规范](https://www.w3.org/TR/did-core/)
- [DIDComm 协议](https://didcomm.org/)
- 对比协议：[[../MCP 协议/MCP 原理|MCP 原理]]、[[../A2A 协议/A2A 原理|A2A 原理]]