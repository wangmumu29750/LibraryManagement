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
| 10 | 2026-10-04 | 实验一 · 步骤5（需求规格） | CodeBuddy（Hy4） | 依据 `00-project-brief.md`、`01-clarifying-questions.md`（Q1–Q29）、`constitution.md` 生成需求规格：项目目标、5 类参与者与权限矩阵、FR-000～FR-019（含新增 FR-017 续借、FR-018/019 评论评分）、NFR-001～010、BR-001～014、每条需求配 AC-xxx 验收标准、本期编码范围划分、4 项待确认问题 | `specs/02-requirements.md` | 待审：重点核对 OPEN-01 与"仅建模"需求是否影响验收 | `20f8a4d` |
| 11 | 2026-10-04 | 实验一 · 步骤5（需求修订） | CodeBuddy（Hy4） | 学生确认 OPEN-01 采用**方案 A（超期不可续借）**：修订 BR-009 为定稿、新增 AC-017-7（超期续借被拒）、在 FR-017 补写"演示路径"约束 | `specs/02-requirements.md` | 已确认：后续用例、顺序图、测试计划均按方案 A 编写 | 本次提交 |
| 12 | 2026-10-04 | 实验一 · 步骤6（用例文本） | CodeBuddy（Hy4） | 依据 `02-requirements.md` 生成用例文本：5 类参与者、22 个用例总览、include/extend 关系表、6 个重点用例完整事件流（办理借书、办理还书、计算超期罚款、办理续借、预约图书、办理借阅证）、其余用例简表、需求追溯表 | `specs/03-use-cases.md` | 待审：重点核对 UC-104 续借事件流与 UC-101 五环节完整性 | `bf38358` |
| 13 | 2026-10-04 | 实验一 · 步骤7（用例图） | CodeBuddy（Hy4） | 依据 `03-use-cases.md` 生成 PlantUML 用例图：5 个参与者（Student/Teacher 泛化自 Reader）、22 个用例、5 组 include、2 组 extend、新增功能备注 | `specs/04-use-case-model.puml` | 待审：本机已装 Java 1.8，需 `plantuml.jar` + Graphviz 或 VS Code PlantUML 插件才能渲染预览 | `49f811a` |
| 14 | 2026-10-04 | 实验一 · 步骤8（领域模型） | CodeBuddy（Hy4） | 依据 `02-requirements.md`、`03-use-cases.md` 生成领域模型：按构造型分类（10 个实体 / 6 个值对象 / 2 个策略对象 / 4 个领域服务）、核心类详解（职责/属性/行为/约束/关系）、继承关系、关系矩阵、10 条领域不变量、目标模型与基座映射表 | `specs/05-domain-model.md` | **已审查通过（2026-10-04）**：实体来自业务概念、续借与评论均已建模、映射简化已留痕 | `37647b4` |
| 15 | 2026-10-04 | 实验一 · 步骤9（领域类图） | CodeBuddy（Hy4） | 依据 `05-domain-model.md` 生成 PlantUML 领域类图：4 个枚举、10 个实体（含两组继承）、2 个策略对象、6 个值对象，表达关联/组合/依赖与多重度，并附续借与评论规则说明 | `specs/06-domain-class-diagram.puml` | **已审查通过（2026-10-04）**：实体继承与策略关系清晰，未混入 Controller / Repository / DTO | `170d17d` |
| 16 | 2026-10-04 | 实验一 · 需求决策（OPEN-02 / OPEN-03） | CodeBuddy（Hy4） | 学生采纳建议：**OPEN-02 选方案 B**（罚款不落库，`fine_records` 仅建模并标注"本期不建表"）；**OPEN-03 保持只建模**（借阅证保留在文档中，实验报告写明"按教学复杂度裁剪，仅完成规格建模"） | `specs/02-requirements.md` | 已确认：后续 `13-database-design.md` 按"预留表"方式编写 | 本次提交 |
| 17 | 2026-10-04 | 实验一 · 步骤10（架构设计） | CodeBuddy（Hy4） | 依据 `02-requirements.md`、`05-domain-model.md`、`constitution.md` 生成架构设计：分层架构总览（Mermaid）、五层职责表、8 个模块划分、MVC 映射、依赖方向、权限策略、异常处理与错误码映射、事务边界、业务规则落点表、8 种设计模式、Agent 接入层挂接规则、目标目录结构 | `specs/07-architecture.md` | 待审：重点核对"Controller 不得直接依赖 Repository"与两个新增功能的挂接位置 | 本次提交 |

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
