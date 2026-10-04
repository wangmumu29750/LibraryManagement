# 图书管理系统开发宪法（Constitution）

> 本文件是**约束 Agent 行为的最高规则**，在生成任何规格、UML、代码或测试之前生效。
> 生成依据：`specs/00-project-brief.md`、`specs/01-clarifying-questions.md`（含人类回答 Q1–Q29）。
> 适用范围：本项目全部 specs、UML、后端代码、Agent 提示词（`SKILL.md`）、测试与文档。

---

## 第 0 条 本项目的固定技术前提（不可协商）

| 项 | 约定 |
|---|---|
| 语言与框架 | Python 3.10+ / FastAPI / SQLAlchemy / Pydantic |
| 数据库 | SQLite（文件位于 `backend/app/library.db`，删除可重新初始化） |
| 服务端口 | **8001**（不是 8000） |
| 测试 | pytest + httpx（`TestClient`），接口级集成测试 |
| UML | **Mermaid 为主**，PlantUML 为辅；所有图必须是**可纳入 Git 的文本文件** |
| Agent 架构 | `orchestrator-agent`（意图识别 + 路由 + 聚合）→ 委派 `circulation-agent`（流通执行）或 `use_skill`（编排层直接参考 skill） |
| 测试账号 | `zhangsan/123456`、`lisi/123456`、`admin/admin123` |

> 除非人类书面变更本文件，否则 Agent 不得提议"改用 Java / Spring Boot""换 MySQL""改端口"等重构方案。

---

## 1. Specs 优先原则

任何代码实现前，必须先完成需求、用例、领域模型、架构、数据库、API、测试和任务拆解文档。
**Agent 不得在对应 specs 尚未冻结前编写该功能的实现代码。**

## 2. Agent 起草、人类确认原则

Agent 可以生成规格初稿，但**所有规格必须经过学生人工审查后**才能作为实现依据。
Agent 每次生成后必须明确说明：改了哪些文件、为什么改、哪些内容是它的推测。

## 3. UML-as-Spec 原则

用例图、类图、包图、顺序图必须使用 PlantUML 或 Mermaid **文本格式**保存到 `specs/` 目录，纳入 Git 管理。
**禁止只把图贴在文档里而不留可版本管理的源文件**；禁止使用无法 diff 的二进制图片作为唯一载体。

## 4. Baseline 原则

Specs 人工审查通过后必须建立 baseline（`git tag experiment1-specs-baseline-v1` / `experiment2-specs-baseline-v1`）。
**实现阶段 Agent 不得擅自修改 specs。** 若发现 specs 与实现冲突：

1. 先停止编码，输出影响分析；
2. 由人类决定改 specs 还是改代码；
3. **严禁为了让测试通过而反向修改 specs 迁就代码。**

## 5. 分层架构原则

系统必须采用分层架构，至少包括：**表现层（Controller）→ 应用服务层（Service）→ 领域层（Domain/Policy）→ 基础设施层（Repository/持久化）**，另设测试层。

- Controller 只做参数校验、调用 Service、封装响应；
- Service 组织用例流程；
- 领域层承载实体与策略对象；
- 基础设施层负责持久化；
- **依赖方向只能自上而下**，下层不得反向依赖上层。

## 6. MVC 与业务规则集中原则

Web/接口层遵循 MVC 或等价分离思想，**不得把业务规则写进 Controller**。
借阅数量、借阅期限、超期罚款、续借条件、评分范围等规则，必须集中在 **Policy / Service / 领域对象**中实现。

## 7. 权限原则

读者、图书管理员、系统管理员权限必须区分并在 Service 层校验。
第一阶段不实现真实认证（无 Token、密码不加密），接口通过 `user_id` 显式表示操作人；
越权或违反角色约束必须返回明确的业务错误，不得静默放行。

## 8. 接口信封与错误码原则

所有业务接口统一返回 `{code, message, data}`：

- `code = 200`：成功；
- `code = 400`：业务错误，`message` 原样透传给用户（如"管理员不能借书""评分必须在 1-5 之间"）；
- `code = 500`：系统异常；
- 资源不存在沿用基座现状返回 `404`（已知偏差 D-02）。

登录接口现阶段返回 `{success, message, data}`（已知偏差 D-01），在 `14-api-spec.md` 中显式标注，不得静默变更。

## 9. 命名一致原则（穿过 API 边界的名字）

凡是穿过 API 边界的标识符——`user_id`、`book_id`、`record_id`、`due_date`、`borrow_date`、`status`、`code`、`message`、`data`、`average_rating`、`total`、`reviews`、接口路径等——
**后端代码、API 规范、Agent 的 `SKILL.md` 提示词三处必须一字不差。**
仅描述流程或面向用户的话术（"聚合""登录上下文""清晰结构化"）用中文自然语言即可，不受此约束。

## 10. Agent 分层挂接原则

- 借书 / 还书 / **续借** / 预约 / 查借阅记录 → 由 `orchestrator-agent` **委派给 `circulation-agent`** 执行；
- 找书 / 搜索 / 登录 / **图书评论与评分** → 编排层 **`use_skill`** 方式调用对应 skill；
- `orchestrator-agent` 自身**不得直接调用 REST API**，只做意图识别、路由分发与结果聚合。

> 挂错层视为架构错误。新增能力时，Agent 必须先判断它属于"流通执行"还是"编排参考"，再决定挂接位置。

## 11. 测试闭环原则

核心用例必须有自动化测试，至少覆盖：
借书成功、超量借书失败、有超期书借书失败、还书成功、超期罚款计算、**续借成功**、**续借已还图书失败**、预约排队、**评分越界被拒**、权限不足被拒。
测试失败时，**先解释原因再做修复**，禁止直接改测试或改 specs 让测试变绿。

## 12. Git 审查原则

Agent 修改任何文件后，学生必须查看 `git diff` 再决定是否提交。
一次只做一个任务，一个任务一次提交；commit message 使用**中文**，并体现"按任务逐步开发"（如 `完成 TASK-007 续借接口`）。

## 13. 变更范围受控原则

每次任务开始前，人类明确"允许改 / 不许改哪些文件"。Agent 不得越界修改：
- 不得在未授权时修改 `specs/` 下已冻结的文档；
- 不得修改基座目录（`作业/`），实现必须在复制出的项目目录中进行；
- **不得大范围重写文件**，优先使用最小增量修改。

## 14. 密钥与隐私原则

- API Key 不得写入 Prompt、不得写入任何被提交的文件；
- `.gitignore` 必须包含 `.env`、`.env.*`、`__pycache__/`、`*.db`、`*.pyc`；
- 日志与错误信息中不得输出密钥、完整 Token 或用户密码。

## 15. AI 使用记录原则

每次使用 Agent 生成 specs、UML、代码或测试，都必须在 `specs/19-ai-usage-log.md` 中登记：
日期、阶段、Agent、任务与 Prompt 摘要、产出文件、人工审查情况、对应提交。
**未登记的生成行为视为未发生，不得计入交付物。**

## 16. 范围纪律原则

Project Brief 第 8 节"暂不实现"以及澄清问题 Q29 的裁剪决定具有约束力：
真实支付与缴款、条码扫描硬件、复杂全文检索、多校区调拨、微信/统一身份认证、消息通知、报表统计、图书推荐、Web 前端，**均不得纳入实现范围**。
Agent 若认为必须新增功能，应先提出影响分析并由人类批准，不得自行扩展。

---

## 附：Agent 标准工作流

```
Plan（先出计划，说明将改动哪些文件）
  → Confirm（人类确认"允许改 / 不许改"的范围）
    → Implement（最小增量实现）
      → Test（运行 pytest，接口 curl 验证）
        → Review（git diff 人工审查）
          → Commit（中文 commit message + 登记 ai-usage-log）
```

违反上述任一原则的输出，学生应要求 Agent 重做，不得直接采纳。
