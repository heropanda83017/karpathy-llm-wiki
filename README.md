# karpathy-llm-wiki

> Hermes Agent skill · 基于 Andrej Karpathy LLM Wiki 规范的知识库维护工具

---

## 功能

多 Vault Wiki 维护系统，提供知识库的全生命周期管理。

### 核心能力

| 功能 | 命令 | 说明 |
|:-----|:------|:------|
| **Wiki 体检** | `lint一下wiki` | 检查链接完整性、frontmatter 有效性 |
| **资料消化** | `ingest` | 将新资料结构化注入 wiki |
| **全文检索** | `query` | 语义搜索 + 关键词融合 |
| **自动上下文** | `wiki_brain` | 启动时自动注入相关上下文 |
| **系统健康** | `自检` | 全面系统健康检查 |

### 支持的多 Vault

| Vault | 路径 | 用途 |
|:------|:------|:------|
| AIGC-KB | `E:/AIGC-KB/wiki-AIGC-KB/` | 主知识库 |
| 理想之地 | (项目专用) | 项目知识库 |

### 健康检查覆盖

- SOUL.md ↔ wiki 对齐
- frontmatter 审计（标题/标签/日期完整性）
- memory 审查（一致性检查）
- 双向链接完整性

---

## 使用方法

```bash
# 在 Hermes Agent 中：
# "lint一下wiki"           → 检查 wiki 健康
# "查一下wiki里关于XX"     → 全文检索
# "给wiki做个全身体检"     → 完整健康检查
# "在AIGC-KB里搜XXX"       → 指定 vault 搜索
```

---

## 架构

```
用户查询
  ↓
wiki_brain context router
  ↓
┌─ FTS5 关键词搜索 ─┐
├─ 语义向量搜索 ────┤  →  RRF 融合  →  输出
└─ 缓存加速 ────────┘
```

---

## 依赖

| 依赖 | 用途 |
|:-----|:------|
| Python 3.14+ | 运行环境 |
| FTS5 | 全文检索引擎 |
| pathlib | 文件操作 |

---

## 相关项目

- [kms-engine](https://github.com/heropanda83017/kms-engine) — 知识管理系统
- [book-note-maker](https://github.com/heropanda83017/book-note-maker) — 读书笔记
- [meeting-minutes](https://github.com/heropanda83017/meeting-minutes) — 公文排版
- [investment-engine](https://github.com/heropanda83017/investment-engine) — A 股量化投资
