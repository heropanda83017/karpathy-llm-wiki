# Profile 系统健康检查流水线

> 适用场景：Hermes profile 自检 — 确保 SOUL.md、wiki、memory、config 一致性
> 触发词：「自检」「系统健康检查」「优化 SOUL memory wiki」

## 五步流水线

```
Step 1: SOUL.md ↔ wiki SCHEMA.md 对齐检查
  ├── 提取 SOUL.md 中声称的 wiki 目录结构
  ├── 对比实际 wiki 目录（os.listdir / walk）
  ├── 不一致 → 以 SCHEMA.md 为准更新 SOUL.md
  └── 常见问题：旧版目录名残留（如 01-fundamentals vs 01-theory）

Step 2: wiki 体检（vault_status + wiki_lint）
  ├── vault_status: 确认路径存在、页面数、index.md/log.md 完整
  ├── wiki_lint: 扫描孤立页面和过时页面
  ├── AIGC-KB 特殊处理：孤立页面不修复（学习笔记 vault 特性）
  └── 过时页面：bump frontmatter 中的 updated 日期

Step 3: Frontmatter 覆盖率审计
  ├── 扫描所有 .md 文件的 frontmatter 字段
  ├── 检查：title / type / updated（或 created）
  ├── 缺失项 → 批量补充
  └── 目标：100% title + date 覆盖

Step 4: Memory 审查
  ├── 检查 memory 条目数
  ├── 至少应有：环境约定 + vault 特性说明
  ├── 去重：不保留过时/重复条目
  └── 补充：关键偏好和环境信息

Step 5: Config + Log 收尾
  ├── 确认 config.yaml 关键字段（model/provider）
  ├── 追加 log.md 维护记录
  └── 输出汇总报告（before/after 对照表）
```

## Frontmatter 批量修复模式

```python
import os, re

def ensure_frontmatter_date(content, rel_path):
    """为缺失 date 字段的 frontmatter 补充 updated"""
    if not content.startswith('---'):
        return content, "no_frontmatter"
    
    fm_match = re.match(r'^(---\n)(.*?)(\n---)', content, re.DOTALL)
    if not fm_match:
        return content, "bad_fm"
    
    before_fm = fm_match.group(1)
    fm_body = fm_match.group(2)
    after_fm = fm_match.group(3)
    rest = content[fm_match.end():]
    
    has_date = bool(re.search(r'^(updated|created|date):', fm_body, re.MULTILINE))
    if has_date:
        return content, "has_date"
    
    # 补充 updated 行
    if fm_body.strip():
        new_fm = before_fm + fm_body.rstrip() + '\nupdated: 2026-05-23' + after_fm
    else:
        new_fm = before_fm + 'updated: 2026-05-23' + after_fm
    
    return new_fm + rest, "added_date"

# 遍历 wiki 目录执行
wiki_dir = r"E:\AIGC-KB\wiki"
for root, dirs, files in os.walk(wiki_dir):
    if '.obsidian' in root:
        continue
    for f in files:
        if not f.endswith('.md'):
            continue
        fp = os.path.join(root, f)
        with open(fp, 'r', encoding='utf-8') as fh:
            content = fh.read()
        new_content, status = ensure_frontmatter_date(content, fp)
        if status == "added_date":
            with open(fp, 'w', encoding='utf-8') as fh:
                fh.write(new_content)
```

## 过时页面批量 Bump

```python
stale_pages = ['path/to/page1.md', 'path/to/page2.md', ...]
for rel in stale_pages:
    fp = os.path.join(wiki_dir, rel.replace('/', os.sep))
    with open(fp, 'r', encoding='utf-8') as f:
        content = f.read()
    new_content = re.sub(
        r'(updated|created):\s*OLD_DATE',
        r'\1: NEW_DATE',
        content
    )
    if new_content != content:
        with open(fp, 'w', encoding='utf-8') as f:
            f.write(new_content)
```

## SOUL.md 架构升级模式：节段重写

当角色能力框架发生结构性升级（如从"领域罗列"重构为"三支柱能力栈"），需要对 SOUL.md 的整个顶层节段进行重写，而非逐行修补。

### 何时用节段重写 vs 逐行修补

| 场景 | 方法 | 示例 |
|------|------|------|
| 目录名不匹配 | 逐行替换 | 01-fundamentals → 01-theory |
| 章节归位错误 | 移动标记 | ## → # （数据源脱离默认行为） |
| 能力框架整体升级 | **节段重写** | "知识覆盖范围" → "三支柱能力架构" |

### 节段重写执行模式

```python
# 1. 定位节段边界：当前节段的 # 标题 → 下一个同级 # 标题
lines = soul.split('\n')
start_idx = next(i for i, l in enumerate(lines) if l.strip() == '# 知识覆盖范围')
end_idx = next((i for i in range(start_idx+1, len(lines))
                if lines[i].startswith('# ') and not lines[i].startswith('## ')), len(lines))

# 2. 构造新节段内容（完整的 markdown）
new_section = """# 三支柱能力架构
...（完整新内容）...
"""

# 3. 拼接：前段 + 新节段 + 后段
new_soul = soul[:start_idx] + new_section + '\n' + soul[end_idx:]

# 4. 落盘 + 验证节段数
with open(soul_path, 'w', encoding='utf-8') as f:
    f.write(new_soul)
sections = re.findall(r'^#(?!#)\s+(.+)$', new_soul, re.MULTILINE)
```

### 三支柱框架模板

```
# 三支柱能力架构
本角色按**数据获取 → 分析研判 → 撰写输出**三层能力栈组织。

## 支柱一：数据获取能力
  - A股核心数据（Tushare/akshare）
  - 海外市场（yfinance）
  - 实时资讯与舆情（aihot/财联社/小红书/公众号）
  - 通用采集（web_search/web_extract/agent-reach）

## 支柱二：分析研判能力
  - 基本面分析（财务拆解/估值模型）
  - 量化分析（回测框架/技术指标/统计工具）
  - 可视化分析（matplotlib/plotly/数据看板）
  - AI技术分析（LLM原理/wiki检索/毛选框架）
  - 宏观经济研判

## 支柱三：撰写输出能力
  - 投资研究报告（个股/行业/周报模板）
  - 自动化报告（cronjob定时生成）
  - 专业文档（公文体/PPT/PDF/反AI写作）
  - 知识沉淀（学习笔记/实战记录→wiki入库）

## 领域知识库
  - AI学习（wiki路径/三元中心/分层架构）
  - 投资哲学（长期主义/安全边际/分散投资/逆向思维）
```

### 升级后验证清单

- [ ] 新节段包含所有旧节段的核心信息
- [ ] 顶级章节数未减少（至少保持原有数量）
- [ ] wiki 路径引用保持正确
- [ ] 投资哲学未丢失
- [ ] 旧节段中 `---` 分隔线在拼接处无残留

| 症状 | 根因 | 修复 |
|------|------|------|
| SOUL.md 目录结构 ≠ wiki 实际 | 角色搭建后 wiki 重构，SOUL 未同步 | 以 SCHEMA.md 为准更新 SOUL |
| index.md 笔记数错误（64→35） | 手动计数失误 | os.walk 统计 .md 文件数 |
| wiki_lint 报告大量孤立页 | AIGC-KB 学习笔记特性 | 不修复，仅记录 |
| wiki_lint 报告 6 页过时 | 重构后 frontmatter 日期未更新 | 批量 bump updated |
| 28 页缺 date 字段 | 早期页面未规范 frontmatter | 批量补充 updated |
| Memory 目录为空 | memory 工具存入 Hermes 内部 DB | 正常，用 memory 工具管理 |

## 2026-05-23 实战案例

见 `log.md` 中的 `2026-05-23 optimization` 条目。本次检查产出：
- SOUL.md 目录结构 5层→8层对齐
- index.md 全量重写（35篇内容笔记完整索引）
- Frontmatter: 0% date → 100% date（43/43）
- 过时页面: 6 → 0
- Memory: 1条 → 3条
