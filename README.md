# karpathy-llm-wiki



> Hermes Agent skill · v

## 功能

1|---
2|name: karpathy-llm-wiki
3|description: 基于 Andrej Karpathy LLM Wiki 规范的知识库维护技能。支持多 vault（理想之地/AIGC-KB双库），对 wiki 做体检（lint）、消化新资料（ingest）、全文检索（query）。覆盖 Profile 系统健康检查（SOUL↔wiki对齐+frontmatter审计+memory审查）。触发词：「lint一下wiki」「查一下wiki里关于XXX」「给wiki做个全身体检」「在AIGC-KB里搜XXX」「自检」「系统健康检查」。
4|version: "2.3"
5|author: hermes-agent
6|---
7|
8|# Karpathy LLM Wiki 维护技能（多 vault 版）
9|
10|基于 [Karpathy LLM Wiki 模式](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) 构建的 wiki 维护系统。
11|
12|## 核心理念
13|
14|传统 RAG：每次问题从原始文档重新检索，LLM 每次"重新发现"知识。  
15|LLM Wiki：知识被编译一次，之后**积累复利**——交叉引用、矛盾标记、跨文档综合都提前做好。
16|
17|### 销售驱动视角（理想之地专属）
18|
19|wiki 以**促进销售**为核心目标，按四维驱动体系组织：
20|- 政策驱动（房票/学区/人才）→ 降低购房门槛
21|- 宣传驱动（舆情/品牌/媒体）→ 建立销售信任
22|- 活动驱动（圈层/商业/产品）→ 吸引到访转化
23|- 管理驱动（工程/物业/团队）→ 保障承诺兑现
24|

## 安装

此技能通过 Hermes Agent 管理。在 Hermes 配置中启用即可：

```bash
hermes skill enable karpathy-llm-wiki
```

## 依赖

- Python 3.10+
- Hermes Agent

## 许可证

MIT
