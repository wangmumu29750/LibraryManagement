# AI 使用记录（AI Usage Log）

> 原则（来自 `constitution.md` 待生成，先行约定）：**每次使用 Agent 生成规格、UML、代码或测试，都必须记录在本文件中**，并配套一次 `git commit`，便于人工审查与答辩复盘。
> 记录字段：编号 / 日期 / 实验阶段 / Agent / 任务与 Prompt 摘要 / 产出文件 / 人工审查情况 / 对应提交。

---

## 记录表

| # | 日期 | 阶段 | Agent | 任务与 Prompt 摘要 | 产出文件 | 人工审查 | 提交 |
|---|---|---|---|---|---|---|---|
| 1 | 2026-10-04 | 实验一 · 步骤0（创建项目） | CodeBuddy（Hy4） | 执行指导书第四节：创建 `library-management-ai` 项目与 `specs/` 目录，创建 21 个规格占位文件，`git init` 并首次提交 | `specs/*.md`、`specs/*.puml`（21 个空文件） | 待审：文件清单与指导书一致（21/21） | `5fb6d4a` |
| 2 | 2026-10-04 | 实验一 · 步骤1（Project Brief） | CodeBuddy（Hy4） | 按指导书第五节模板写入项目背景、目标、用户、业务、读者类型、借出物类型、技术要求、暂不实现 | `specs/00-project-brief.md` | 待审：已主动补入"续借图书""图书评论与评分"两项业务，与任务卡两个扩展功能对齐 | `c2574b6` |
| 3 | 2026-10-04 | 实验一 · 步骤2（澄清问题） | CodeBuddy（Hy4） | 阅读 `00-project-brief.md`，按类别提出澄清问题，覆盖参与者/用例/业务规则/借阅规则/罚款规则/预约规则/权限/数据/架构/测试/教学复杂度，并预留"人类回答"区 | `specs/01-clarifying-questions.md` | 已自查：**首版 50 条超出指导书"20–30 个"限制** | `fc3e331` |
| 4 | 2026-10-04 | 实验一 · 步骤2（修订） | CodeBuddy（Hy4） | 人工审查发现问题数量超标，精简合并为 29 条，保持类别全覆盖，并新增"评论与评分"问题组 | `specs/01-clarifying-questions.md` | 已自查：29 条，符合 20–30 要求；⭐ 高优先级 19 条已标注 | `0ffd975` |
| 5 | 2026-10-04 | 实验一 · 步骤3（回答澄清问题） | CodeBuddy（Hy4） | 按"以基座代码现状为准"的原则逐条回答 Q1–Q29；已通读基座 `models.py` / `routes.py` 以保证答案与实现一致 | `specs/01-clarifying-questions.md`（人类回答区） | 待审：答案中标注了 3 处"基座现状与规格的已知偏差"，见下方"偏差登记" | 本次提交 |
| 6 | 2026-10-04 | 实验一 · 过程记录 | CodeBuddy（Hy4） | 补建本 AI 使用记录文件，并登记第 1–5 条历史使用记录 | `specs/19-ai-usage-log.md` | 待审 | 本次提交 |

---

## 偏差登记（规格 vs 基座实现）

> 记录在回答澄清问题时发现的"文档口径与现有代码不一致"之处，后续在 `02-requirements.md` / `14-api-spec.md` 中必须显式说明，避免实现阶段踩坑。

| 编号 | 偏差 | 基座现状 | 处理决定 |
|---|---|---|---|
| D-01 | 登录接口响应结构 | `/api/login` 返回 `{success, message, data}`，未使用统一信封 `{code, message, data}` | 第一阶段保持不动，在 `14-api-spec.md` 中显式标注例外 |
| D-02 | 资源不存在的状态码 | 部分接口返回 `404`（如 `/api/users/{id}`、`/api/books/{id}`） | 约定：业务/权限拒绝用 `400`，资源不存在沿用 `404`，系统异常 `500` |
| D-03 | 借阅期限与罚款规则 | 借期统一 30 天、罚款统一 0.5 元/天，未按读者类型/借出物类型差异化 | 差异化规则写入 `borrow_policies` / `fine_rules` 表设计，本期代码沿用基座单一规则 |
| D-04 | 预约的存储方式 | 预约复用 `borrow_records` 表，`status = "reserved"`，无独立 `reservations` 表 | specs 中建模独立 `reservations` 表；实现沿用基座，不做迁移 |
| D-05 | 馆藏副本建模 | `books` 表用 `total_copies` / `available_copies` 计数，无单副本明细 | 概念模型保留 `book_titles` + `library_items`，实现沿用简化，specs 中写明简化理由 |

---

## 待补充（后续每次生成内容时追加）

- 步骤4：`constitution.md` 生成记录
- 步骤5：`02-requirements.md` 生成记录
- 步骤6：`03-use-cases.md` 与 `04-use-case-model.puml` 生成记录
- 步骤7：`05-domain-model.md` 与 `06-domain-class-diagram.puml` 生成记录
- 步骤8：`07-architecture.md` 与 `08-package-diagram.puml` 生成记录
- 步骤10：`13-database-design.md` 生成记录
- 步骤13：`17-risk-analysis.md`、`18-review-checklist.md` 生成记录
- 实验二：`09/10/11/12/14/15/16` 各文档生成记录
- 编码阶段：续借、评论评分实现与测试记录
