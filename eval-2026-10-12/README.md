# Fair200 v3 · Side-by-Side 评测发布包（2026-10-12）

本目录为 **Fair200 v3**（约 200 条样本外对话）上，**Dafa new-core** 与 **Kimi** 的对外 side-by-side 结果。

| 文件 | 内容 |
|---|---|
| `Fair200-v3_Dual-LLM-Judge_Side-by-Side.xlsx` | 双模型 LLM 盲评 + 量化策略工程审计后的综合 side-by-side |
| `Fair200-v3_Jev-Judge_Side-by-Side.xlsx` | OpenRouter Jev 盲评 side-by-side（全场景合并版） |
| `judge-prompt.md` | 两套裁判共用的主评测 Prompt 全文 |

隐私：已删除 `kimi_url` / `image_url` 类字段；单元格内若出现图片链接会替换为占位符。

---

## 1. 评测对象与输入

- **评测集**：`kimi_compare_fair200_20261009_v3`（Fair200 v3）
- **对比双方**：
  - **Dafa**：new-core 重跑回答（同一用户问题）
  - **Kimi**：同期固定回答
- **送入裁判的研究问题**：优先 `extend_query`（含持仓/策略等扩展上下文）；无扩展时等于 `query`
- **匿名**：回答以 A/B 呈现，裁判不知道产品身份

---

## 2. 双模型裁判（Dual LLM Judge）

对应文件：`Fair200-v3_Dual-LLM-Judge_Side-by-Side.xlsx`

### 2.1 方法

1. **盲评**：两份回答匿名为 A/B。
2. **双裁判**：
   - `doubao:deepseek-4.1-flash`
   - `doubao:glm-5.3-flash`
3. **正序 + 位置互换**：每个裁判各评两次（A/B 对调），降低位置偏差。
4. **聚合规则**：
   - 两模型结论不一致 → **平局（tie）**
   - 同一模型正序与互换冲突 → 该模型票作废（不作为稳定胜负）
   - 仅「结构」维分出胜负、其余三维均为 tie → 总结果强制 **tie**
5. **量化策略额外工程审计**（与文字 pairwise 独立）：
   - Dafa：是否真实落库 / 回测 / trace 完成
   - Kimi：回答中的代码与回测声明是否可核验（静态审计，不执行第三方代码）
   - 内容胜负与工程证据冲突时，保守降为平局或以「真实完成侧」为准（见工程有效胜负）

### 2.2 表结构（要点）

- `结论总览`：整体与分场景胜率摘要  
- `pairwise明细`：逐 case 的 `winner`、双模型理由、工程审计字段、双方原文等  

### 2.3 胜率口径

```text
加权胜率（soft） = (胜 + 0.5 × 平) / 可评测数量
```

默认 hold 带：相对 Dafa 胜率落在 `[0.47, 0.53]` 视为总体持平。

---

## 3. Jev 裁判（Jev Judge）

对应文件：`Fair200-v3_Jev-Judge_Side-by-Side.xlsx`

### 3.1 方法

1. **模型**：OpenRouter `~typesafe/jev-latest`（决策/choice 接口）。
2. **Prompt**：与双模型裁判使用**同一份**主评测规则（见 `judge-prompt.md`）。
3. **结构化收取**：Jev 不写自由 JSON `reason`；在 Prompt 原文之外附带 5 个 choice（总体 + 四维：论点覆盖 / 论据质量 / 平衡性 / 结构），选项为 A / B / tie。
4. **正序 + 位置互换**：不一致记 tie；仅结构维分胜负则 tie。
5. **量化策略**：默认叠加与双模型一致的工程证据 / 有效胜负字段（`effective_winner`、`engineering_*` 等）。
6. **与双模型对照**：表中含 `llm_winner` / `agree_llm_display`，便于看 Jev 与 Dual-LLM 是否一致。

### 3.2 表结构（要点）

| Sheet | 含义 |
|---|---|
| `00_总览` | 汇总说明与胜率 |
| `01_全部side_by_side` | 全量明细 |
| `02_kimi胜` / `03_dafa胜` / `04_平局` | 按展示胜负拆分 |
| `05_高置信_min0.7` | 低置信过滤后的子集 |
| `06_与LLM不一致` | 与双模型结论不一致的 case |
| `10_`～`19_` | 按业务场景（L1）拆分 |

---

## 4. 共用评测维度与 Prompt

两套裁判都按以下四维比较，再给总体 winner：

1. **论点覆盖**：是否对准必需关键点，是否完成用户要求的交付物  
2. **论据质量**：可核对事实、时点口径、量化是否站得住  
3. **平衡性**：必要风险/边界是否具备  
4. **结构**：结论能否快速定位（不是排版竞赛）

核心原则（摘要）：

- 身份无关；忽略篇幅长短，看信息密度与论证质量  
- **完成交付优先于**只给方法论 / 伪代码  
- 产品引导话术（订阅确认、保存策略、下载完整研报等）**不得**单独作为胜负依据  
- 文风 / Markdown 表格 / 厚度差异在核心信息等价时应判 **tie**

**完整 Prompt 正文**：见同目录 [`judge-prompt.md`](./judge-prompt.md)。

运行时占位：

```text
研究问题：{query}
用户上下文：{context}
回答A：{answer_a}
回答B：{answer_b}
```

---

## 5. 如何阅读结果列（速查）

| 字段 | 含义 |
|---|---|
| `winner` / `display_winner` | 对外展示用的综合胜负（dafa / kimi / tie） |
| `effective_winner` | 叠加工程审计后的有效胜负（Jev 表） |
| `jev_winner` | Jev 语义盲评胜负 |
| `llm_winner` | 双模型裁判胜负（对照） |
| `conf_min` / `conf_fwd` / `conf_swp` | Jev 置信（正序/互换） |
| `dafa_answer` / `kimi_answer` | 双方回答原文 |
| `engineering_winner` | 量化策略工程审计胜负 |

---

## 6. 复现脚本（仓库内）

```text
双模型：scripts/run_kimi_dafa_pairwise.py
Jev：    scripts/run_fair200_jev_pairwise.py
Prompt： scripts/run_kimi_dafa_pairwise.py → JUDGE_PROMPT
```

---

## 7. 版本说明

- 评测集版本：Fair200 **v3**（2026-10-09 构建）  
- 双模型结果固化：工程审计合并版（源文件日期约 2026-10-10 / 10-11）  
- Jev 合并完整结果：2026-10-12  
- 本发布目录命名：`eval-2026-10-12`
