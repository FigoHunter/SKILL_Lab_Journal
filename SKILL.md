---
name: experiment-journal
description: >-
  Maintain an experiment lab journal of detailed statistical analysis reports
  as interlinked HTML pages (cover + TOC in index.html, one full report page
  per experiment) under logs/lab_journal/. Three trigger scenarios: (1) the
  user asks to 记实验日记 / 整理实验 / journal the experiment, including
  reconstructing past experiments from agent transcripts; (2) a new batch of
  raw experiment results (results JSON, sweep output, eval logs) appears and
  needs full analysis — descriptive stats, per-seed tables, significance
  tests, plots; (3) an experiment cycle happened in conversation (exploration,
  questions raised, re-verification, decisions, execution) and the process
  itself must be recorded, not just final numbers.
---

# Experiment Journal（实验分析报告集）

日记 = `logs/lab_journal/` 下一组互链 HTML：封面目录 + **每个实验周期一份完整
分析报告页**。核心要求：报告要详细到用户不需要逐句追问——统计、图表、过程
叙事都主动做全。

## 目录结构

```
logs/lab_journal/
├── index.html                    # 封面 + 目录（每份报告一行）
├── style.css
├── r2026-07-01_mapped-new.html   # 一份报告一页，命名 rYYYY-MM-DD_slug.html
├── assets/                       # 图（相对路径引用，禁止 base64 内嵌）
└── analysis/                     # 报告用的统计/绘图脚本，留档可复跑
```

一份报告 = 一个完整实验周期（问题 → 探索 → 跑 → 分析 → 决策）。同一周期内
相关的多组运行合成一份报告，不拆碎；不同实验各成一页，避免单文件过大。
初始化（仅当目录不存在）：从 `templates/` 拷 `index.html`、`style.css`，建空
`assets/`、`analysis/`。

## 三个触发场景

**场景 A：从 transcript 回溯整理。** 用户要求补记过去的实验时，检索 agent
transcripts（转录目录见系统提示）+ 结果文件时间戳 + git log，重建时间线。
先搜关键词定位，再读命中行附近的小窗口，不要通读。转录里的数字一律要用
结果文件原件复核，对不上以文件为准并注明。

**场景 B：拿到一组原始结果。** 立即执行下面的统计分析规程 + 绘图，不等用户
逐项要。

**场景 C：对话中完成了一轮实验。** 除了数字，把过程写进报告的"过程记录"
章节：当时的疑问、走过的弯路（含错误假设和被推翻的解释）、再验证的手段、
最终方案与执行。弯路是重要内容，禁止只留"成功路径"。

## 统计分析规程（场景 B/C 强制）

写 `analysis/stats_<slug>.py` 对原始文件逐项算并留档，禁止只抄现成 mean 字段：

1. **数据清点**：n、缺失项、config 出处（训练数据/标签组/评估集确切路径）。
2. **描述统计**：每指标 mean±std(ddof=1)、min/max；**逐 seed 全表必须进报告**。
3. **配对比较**：同 seed 配对 paired t（n=5 |t|>2.776、n=10 |t|>2.262 为
   p<0.05）；报告效应量（均值差 + Cohen's dz）；显著与不显著都写，ns 只能
   说"均值领先、差异不显著"。
4. **稳健性**：leave-one-worst-out 重算 mean；std 异常大的行点名；配对逐
   seed 胜负计数（如 3/5）。
5. **趋势**（扫参数时）：单调性/平台区、最优点 vs 部署点差距是否超 1 std。
6. **跨指标一致性**：逐帧 vs 分段排名是否一致；不一致是发现，必须分析成因。
7. **结论纪律**：结论句都带数字；推测标注"推测，未验证"；写明"这组数据
   **不能**说明什么"。

## 绘图规程

每份报告至少 1 张图，用 `analysis/` 里的脚本生成 PNG 存 `assets/`
（`YYYY-MM-DD_名字.png`）：扫参数 → 折线+误差带并标部署点；方法对比 →
分组柱状图（关键指标 2–3 个）；配对比较 → 逐 seed 散点/连线。matplotlib
默认风格即可，坐标轴/图例/标注完整，图注写清"这张图说明什么"。

## 报告的固定章节（按序，缺一不可）

| 章节 | 内容 |
|---|---|
| 摘要 | 3–5 句：做了什么、最重要 2–3 个数字结论、决策 |
| 背景与动机 | 问题、为什么做、假设（叙述体，能独立读懂） |
| 过程记录 | 时间线：疑问 → 探索 → 弯路/被推翻的假设 → 再验证 → 方案 → 执行 |
| 实验设置 | 数据（路径+规模+标签组）、模型/超参、协议（固定什么/变什么）、seeds、可复制命令 |
| 代码与产物 | 脚本（相对路径+职责）、结果文件、analysis/ 脚本 |
| 原始结果 | 全量指标表（mean±std+best 标记）+ 逐 seed 全表 + 图 |
| 统计分析 | 配对检验表（t/dz/显著性）、稳健性、趋势、跨指标一致性 |
| 讨论 | 机制解释、与前序报告呼应、局限、"不能说明什么" |
| 结论与决策 | 带数字的结论清单；决策及理由 |
| 遗留问题 | 可执行待办；被后续解决后回填"→ 见 …"链接 |
| 相关报告 | 前因后果链接（没有写"无"） |

## 收尾

1. `index.html` 目录 `<!-- TOC -->` 后插行（时间倒序）：日期/标题/带数字的
   一句话结论；更新封面报告数与日期范围。
2. 本报告链接旧报告；旧报告被解决的遗留问题处回填链接。
3. 语言跟随用户（默认中文）；标题能独立读懂。

## 模板

- [templates/report.html](templates/report.html) — 报告页骨架
- [templates/index.html](templates/index.html) — 封面+目录
- [templates/style.css](templates/style.css) — 共享样式
