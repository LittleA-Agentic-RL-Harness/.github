# LittleA Agentic RL Harness

> **Agentic RL Harnesses & AI-native apps by LittleA** · LittleA 的 Agentic RL Harness 与 AI 原生应用。
> 可审计、可复现、人在环内——Agent 的价值来自真实世界的效果与证据链，而不是演示。

中文 · [English Summary](#english-summary)

## 概述

LittleA Agentic RL Harness 组织围绕**「Agent 真实执行」**构建基础设施与产品：

- **Harness / 基础设施**：多设备 Android 真机 Harness、精确一次（exactly-once）的执行效果、证据链、提案式推理、实时设备控制台，以及多 Agent 框架与官方样例。
- **AI 原生应用**：把 Harness 能力落到真实场景——播客、拉新、对话人感、商业产品工程。
- **研究与运营**：递归自改进（RSI）baseline 与真人团队运营方法论（HOP）。

所有对外执行**带披露、经人工审批**；执行效果与证据可回溯。

## 项目版图

| 分类 | 仓库 | 说明 |
| --- | --- | --- |
| Harness / Infra | `littlea-agent-infra` | Canonical harness monorepo：Coreline、Gateway、device fleet、Inside Harvest、OmniHub、contracts、evidence |
| Harness / Infra | `littlea-android-agent-harness` | 可审计 Android Agentic RL Harness：多设备 ADB、GUI-Owl 提案式推理、精确一次效果、Device Console |
| Harness / Infra | `LittleAFamily` | 框架与官方样例：多 Agent 业务闭环、AgentPackage、WorkflowMap、LangGraph / Langfuse 观测 |
| AI 原生应用 | [`littlea-evocast`](https://github.com/LittleA-Agentic-RL-Harness/littlea-evocast)（公开） | AI-native 播客：论文 / 报告 / 文档 / 链接 → 每位听众专属的 3–8 分钟双主播节目，HarnessPro 自我改进 |
| AI 原生应用 | `littlea-resident` | 面向 C 端产品的通用 AI 拉新：发现求助者 → 起草带披露回复 → 人审 → 真机 / 桌面执行 → 归因 |
| AI 原生应用 | `LittleA_Persona_Speaking_Prototype` | 人感对话原型：方案实现与交互界面 |
| 产品工程 | `littlea-singapore` | LittleA Singapore：可信商户供需与人设匹配、券、模型预算、Android fleet、可审计证据 |
| 产品工程 | `LittleBFanDaNew` | 梵大营养服务产品：Agent Task Platform（Runtime / Task / Studio）与营养运营模块 |
| 产品工程 | `LittleBJiangZiYa` | 姜子牙：AI 诊所接待与管理工作台（统一后端 + Vue 3 管理端 + UniApp 接待端） |
| 研究与运营 | `LittleA-Father-Agent` | Father Agent RSI 初步 baseline（含构造数据） |
| 研究与运营 | `littlea-hop` | HOP（Human Operations Playbook）：真人团队的运营实践、SOP、案例与复盘 |

> 部分仓库当前为私有，组织成员可见。

## 技术栈

TypeScript / Next.js（Vercel）· Python（LangGraph · uv）· Android（ADB · 真机设备 fleet）· Node · MongoDB · PostgreSQL · GitHub Actions

<a name="english-summary"></a>

## English Summary

**LittleA Agentic RL Harness** — open-source agentic RL harnesses and AI-native apps. Multi-device Android harnesses with exactly-once effects and auditable evidence chains, a multi-agent framework with official samples and LangGraph/Langfuse observability, and AI-native apps built on top (EvoCast personalized podcasts, Resident growth loops with disclosure and human approval, persona conversations, and product engineering for real businesses). Research on recursive self-improvement baselines and human-operations playbooks completes the loop. Real-world effects, human approval gates, and evidence chains are first-class.

Some repositories are currently private and visible to organization members.
