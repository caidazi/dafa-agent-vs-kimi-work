# Dafa vs Kimi · Side-by-Side 评测结果（2026-10-12）

本目录为约 **200 条**真实用户问题样本上，**Dafa** 与 **Kimi** 的 side-by-side 对比结果。

| 文件 | 内容 |
|---|---|
| `Fair200-v3_Dual-LLM-Judge_Side-by-Side.xlsx` | 双模型 LLM 盲评（量化策略含工程审计综合结论） |
| `Fair200-v3_Jev-Judge_Side-by-Side.xlsx` | Jev 盲评 |
| `judge-prompt.md` | 两套裁判共用的主评测 Prompt |

每个 Excel 含：`结论总览`（胜率摘要）+ `side_by_side`（全量明细）。  
隐私：已删除外链 URL 类字段。

---

## 1. 评测对象与输入

- **对比双方**：同一用户问题上的 Dafa 回答 vs Kimi 回答
- **送入裁判的研究问题**：优先带扩展上下文的完整问题；无扩展时等于原始问题
- **匿名**：回答以 A/B 呈现，裁判不知道产品身份

---

## 2. 双模型裁判（Dual LLM Judge）

对应文件：`Fair200-v3_Dual-LLM-Judge_Side-by-Side.xlsx`

### 2.1 方法

1. **盲评**：两份回答匿名为 A/B。
2. **双裁判**：DeepSeek、GLM。
3. **正序 + 位置互换**：每个裁判各评两次（A/B 对调），降低位置偏差。
4. **聚合规则**：
   - 两模型结论不一致 → **平局（tie）**
   - 同一模型正序与互换冲突 → 该模型票不计入稳定胜负
   - 仅「结构」维分出胜负、其余三维均为 tie → 总结果强制 **tie**
5. **量化策略额外工程审计**（与文字对比独立）：是否真正完成策略保存 / 回测等可核验交付；与文字结论冲突时保守处理。

### 2.2 明细字段（`side_by_side`）

| 字段 | 含义 |
|---|---|
| `conversation_id` | 样本 ID |
| `scenario` | 业务场景 |
| `winner` | 综合胜负（dafa / kimi / tie） |
| `reason` | 胜出原因摘要 |
| `query` | 用户问题 |
| `dafa_answer` / `kimi_answer` | 双方回答 |
| `deepseek_reason` / `glm_reason` | 两名裁判的理由 |

### 2.3 胜率口径

```text
加权胜率（soft） = (胜 + 0.5 × 平) / 可评测数量
```

相对 Dafa 胜率落在 `[0.47, 0.53]` 时，视为总体持平。

---

## 3. Jev 裁判（Jev Judge）

对应文件：`Fair200-v3_Jev-Judge_Side-by-Side.xlsx`

### 3.1 方法

1. **模型**：Jev（决策 / choice 接口）。
2. **Prompt**：与双模型裁判使用**同一份**主评测规则（见 `judge-prompt.md`）。
3. **结构化收取**：不写自由文本理由；附带 5 个 choice（总体 + 四维），选项为 A / B / tie。
4. **正序 + 位置互换**：不一致记 tie；仅结构维分胜负则 tie。
5. **量化策略**：叠加工程证据后的综合胜负。

### 3.2 明细字段（`side_by_side`）

| 字段 | 含义 |
|---|---|
| `conversation_id` | 样本 ID |
| `scenario` | 业务场景 |
| `winner` | 综合胜负（dafa / kimi / tie） |
| `query` | 用户问题 |
| `context` | 扩展上下文（持仓等；无则为空） |
| `dafa_answer` / `kimi_answer` | 双方回答 |
| `confidence` | 裁判置信（正序与互换的较低值） |
| `checklist` | 四维选择结果 |

---

## 4. 共用评测维度与 Prompt

两套裁判都按以下四维比较，再给总体 winner：

1. **论点覆盖**：是否对准必需关键点，是否完成用户要求的交付物  
2. **论据质量**：可核对事实、时点口径、量化是否站得住  
3. **平衡性**：必要风险 / 边界是否具备  
4. **结构**：结论能否快速定位（不是排版竞赛）

核心原则（摘要）：

- 身份无关；忽略篇幅长短，看信息密度与论证质量  
- **完成交付优先于**只给方法论 / 伪代码  
- 产品引导话术（订阅确认、保存策略、下载完整研报等）**不得**单独作为胜负依据  
- 文风 / Markdown 表格 / 厚度差异在核心信息等价时应判 **tie**

**完整 Prompt**：见 [`judge-prompt.md`](./judge-prompt.md)。

占位符：

```text
研究问题：{query}
用户上下文：{context}
回答A：{answer_a}
回答B：{answer_b}
```
