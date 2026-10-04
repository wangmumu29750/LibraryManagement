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
| 6 | 2026-10-04 | 实验一 · 过程记录 | CodeBuddy（Hy4） | 补建本 AI 使用记录文件，并登记第 1–5 条历史使用记录 | `specs/19-ai-usage-log.md` | 待审 | `36b061f` |
| 7 | 2026-10-04 | 实验一 · 步骤4（开发宪法） | CodeBuddy（Hy4） | 依据 `00-project-brief.md` 与 `01-clarifying-questions.md`（Q1–Q29 答案）生成开发宪法：约束 Agent 生成 specs/UML/代码/测试的行为，强调 UML-as-Spec、Baseline 冻结、分层架构与 MVC、业务规则不进 Controller、权限、响应信封、API 命名一致、Agent 分层挂接、测试闭环、Git 审查、密钥与 AI 使用记录 | `specs/constitution.md` | **已审查通过（2026-10-04）**：16 条原则 + 固定技术前提表 + 标准工作流全部采纳 | `9ac57af` |
| 8 | 2026-10-04 | 准备 · 行动顺序第1步（复制基座） | CodeBuddy（Hy4） | 将基座 `作业/` 复制到 `library-hw-system-1024005295/`（排除 `__pycache__`，共 31 个文件），新增 `.gitignore`（含 `.env`、`*.db`），`git init` 并提交基线。**未改动任何功能代码** | `library-hw-system-1024005295/`（独立仓库） | 已自查：与基座逐文件一致，仅多 `.gitignore` | `a709766`（代码仓库） |
| 9 | 2026-10-04 | 准备 · 行动顺序第2步（跑通基座） | CodeBuddy（Hy4） | 启动后端 `python main.py`，验证 `/api/health`、`/api/login`（3 个测试账号 + 错误密码）、`/api/borrow/check-quota/1`、`/api/borrow/records/1`、`/api/books` | 验证结论见"环境验证结论"小节；新发现偏差 D-06 | 已自查：接口全部可用，实测结论已登记 | `46da208` |
| 10 | 2026-10-04 | 实验一 · 步骤5（需求规格） | CodeBuddy（Hy4） | 依据 `00-project-brief.md`、`01-clarifying-questions.md`（Q1–Q29）、`constitution.md` 生成需求规格：项目目标、5 类参与者与权限矩阵、FR-000～FR-019（含新增 FR-017 续借、FR-018/019 评论评分）、NFR-001～010、BR-001～014、每条需求配 AC-xxx 验收标准、本期编码范围划分、4 项待确认问题 | `specs/02-requirements.md` | 待审：重点核对 OPEN-01（超期能否续借，与 D-06 冲突）与"仅建模"需求是否影响验收 | 本次提交 |

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
| D-06 | 种子数据全部超期（实测发现） | 系统当前日期为 2026-10，而种子借阅记录 `due_date` 为 2026-07-15 / 07-20 / 08-01，`zhangsan`、`lisi` 的在借记录**全部已超期**（`has_overdue = true`，`overdue_count = 2`） | 后果：借书会被"有超期未还"拦截；若续借规则为"超期不可续借"，续借演示也会失败。**建议演示路径：先还掉超期书 → 再借新书（due_date = 今天 + 30 天）→ 再续借** |

---

## 环境验证结论（2026-10-04 实测）

| 项 | 结果 |
|---|---|
| Python | 3.10.2 ✅ |
| 依赖 | fastapi 0.115.0、uvicorn 0.30.0、SQLAlchemy 2.0.49、pydantic 2.9.0 ✅ 均无需安装 |
| 服务启动 | `python main.py` 正常，监听 `http://0.0.0.0:8001` ✅ |
| `/api/health` | `{"status":"ok"}` ✅ |
| `/api/login` | `zhangsan/123456` → `user_id=1, role=reader, max_borrow=5` ✅；`lisi/123456` → `user_id=2` ✅；`admin/admin123` → `user_id=3, role=admin` ✅；错误密码 → `success=false, message=密码错误` ✅ |
| `/api/borrow/check-quota/1` | `current_borrowed=2, max_borrow=5, has_overdue=true, overdue_count=2` ⚠️ 见 D-06 |
| `/api/borrow/records/1` | 返回 2 条 `status=borrowing` 记录（`record_id=1`《三体》、《`record_id=2`深入理解计算机系统》） ✅ |
| `/api/books` | `total=7`，含《三体》等种子图书 ✅ |
| SQLite | `backend/app/library.db` 自动生成（49 KB）✅ |

> 控制台中的中文乱码（如 `ç»å½æå`）仅为 PowerShell GBK 显示问题，**接口返回本身是正确 UTF-8**，不影响使用。

---

## 待补充（后续每次生成内容时追加）

- 步骤5：`02-requirements.md` 生成记录
- 步骤6：`03-use-cases.md` 与 `04-use-case-model.puml` 生成记录
- 步骤7：`05-domain-model.md` 与 `06-domain-class-diagram.puml` 生成记录
- 步骤8：`07-architecture.md` 与 `08-package-diagram.puml` 生成记录
- 步骤10：`13-database-design.md` 生成记录
- 步骤13：`17-risk-analysis.md`、`18-review-checklist.md` 生成记录
- 实验二：`09/10/11/12/14/15/16` 各文档生成记录
- 编码阶段：续借、评论评分实现与测试记录
