# 领域模型（Domain Model）

| 项 | 内容 |
|---|---|
| 文档编号 | `05-domain-model.md` |
| 版本 | v1.0（实验一 · 步骤8） |
| 状态 | **待人工审查** |
| 生成依据 | `02-requirements.md`（FR / BR）、`03-use-cases.md`、`constitution.md` |
| 下游文档 | `06-domain-class-diagram.puml`（领域类图）、`07-architecture.md`、`13-database-design.md` |

> 建模原则（宪法第 5、6 条）：领域类来自**业务概念**，不是数据库表的机械翻译；业务规则集中在**领域对象与策略对象**中，不放 Controller。
> 字段命名与 API 边界保持一致（`user_id`、`book_id`、`record_id`、`due_date`、`average_rating` …）。

---

## 1. 领域对象总览（按构造型分类）

| 构造型 | 领域对象 | 说明 |
|---|---|---|
| **实体（Entity）** | `Reader`（抽象）、`StudentReader`、`TeacherReader` | 有唯一标识、有生命周期 |
| | `BorrowCard` | 借阅证，与读者一一绑定 |
| | `Librarian`、`SystemAdmin` | 业务代理者与维护者 |
| | `BookTitle` | 图书标题（书目）信息 |
| | `LibraryItem`（抽象）、`Book`、`Magazine`、`Thesis` | 馆藏资源与借出物类型 |
| | `Loan` | 借阅记录，承载借出/归还/续借行为 |
| | `Reservation` | 预约，支持排队 |
| | `FineRecord` | 罚款记录 |
| | `Review` | 图书评论与评分（本次新增） |
| **值对象（Value Object）** | `CardNo` | 借阅证号，自带格式与唯一性语义 |
| | `Barcode` | 馆藏条码 |
| | `ISBN` | 国际标准书号 |
| | `Money` | 金额（元），支持四则运算与上限截断 |
| | `Rating` | 评分，构造时校验 1–5 |
| | `DateRange` | 借期区间（起止日期），提供超期判定 |
| **策略 / 规则对象（Policy）** | `BorrowPolicy` | 读者类型 → 借阅数量、期限、**续借次数与天数** |
| | `FineRule` | 借出物类型 → 日罚款费率、上限 |
| **领域服务（Domain Service）** | `CirculationService` | 借书 / 还书 / **续借** 的流程编排与事务边界 |
| | `ReservationService` | 预约排队、有效期失效与递补 |
| | `CatalogService` | 图书检索 |
| | `ReviewService` | 评论保存（一人一书一条）与平均分计算 |

---

## 2. 核心领域类详解

### 2.1 Reader（读者，抽象实体）

| 项 | 内容 |
|---|---|
| **职责** | 表示图书馆服务对象的身份与借阅资格 |
| **关键属性** | `user_id: int`、`username: str`、`password: str`、`reader_type: ReaderType`（枚举：UNDERGRADUATE / POSTGRADUATE / DOCTORAL / SPECIALTY / TEACHER）、`max_borrow: int`、`email: str`、`created_at: datetime` |
| **关键行为** | `is_eligible_to_borrow()`、`can_borrow_more(current: int) -> bool`、`has_overdue(loans) -> bool`、`get_policy() -> BorrowPolicy` |
| **约束** | `username` 全局唯一；读者类型决定其适用的 `BorrowPolicy` |
| **关系** | 1 对 1 持有 `BorrowCard`；1 对多关联 `Loan`、`Reservation`、`Review` |
| **子类** | `StudentReader`（学生：本科/研究生/博士/专科）、`TeacherReader`（教师） |

> 实现说明（Q2）：**类图与领域模型中保留继承关系**，数据库映射采用"单表 + `reader_type` 字段"，与基座 `users` 表一致（D-03 同类简化）。

### 2.2 BorrowCard（借阅证，实体）

| 项 | 内容 |
|---|---|
| **职责** | 证明读者的借阅资格 |
| **关键属性** | `card_id: int`、`card_no: CardNo`、`reader_id: int`、`issue_date: date`、`expiry_date: date`、`status: str`（valid / expired / lost） |
| **关键行为** | `is_valid() -> bool`、`renew()`（续期） |
| **约束** | `card_no` 格式 `CARD + 年份 + 6 位序号` 且全局唯一；**一人一证**（BR-001）；过期不可借书 |
| **关系** | 属于唯一一个 `Reader`；被 `Loan`、`Reservation` 校验引用 |
| **编码范围** | 本期**仅建模**（FR-002 / FR-003） |

### 2.3 Librarian / SystemAdmin（管理员，实体）

| 项 | 内容 |
|---|---|
| **职责** | `Librarian` 代理办理借书、还书、续借，可查询任意读者借阅信息；`SystemAdmin` 负责借阅证、图书、副本、管理员与规则的维护 |
| **关键属性** | `admin_id: int`、`username: str`、`password: str`、`role: str`（librarian / admin） |
| **关键行为** | `borrow_for(reader, item)`、`return_for(loan)`、`renew_for(loan)`（Librarian）；`issue_card(reader)`、`maintain_rule(policy)`（SystemAdmin） |
| **约束** | `username` 唯一；**管理员本人不能借书**（BR-003） |
| **关系** | 与 `Loan` 存在"经手"关联（本期 `loans.librarian_id` 可空，不强制记录） |

### 2.4 BookTitle（图书标题，实体）

| 项 | 内容 |
|---|---|
| **职责** | 描述书目层面的信息，是预约与评论的对象 |
| **关键属性** | `title_id: int`、`title: str`、`author: str`、`isbn: ISBN`、`publisher: str`、`published_year: int`、`category: str`、`location: str`、`price: Money`、`description: str`、`is_active: bool` |
| **关键行为** | `available_count() -> int`、`is_reservable() -> bool`、`average_rating() -> float`、`add_item(item)`、`remove_item(item)` |
| **约束** | `isbn` 全局唯一；删除为**逻辑删除**（`is_active = false`） |
| **关系** | 1 对多组合 `LibraryItem`；被 `Reservation` 与 `Review` 引用 |
| **映射** | 对应基座 `books` 表（含 `total_copies` / `available_copies` 计数，见 D-05） |

### 2.5 LibraryItem（馆藏资源，抽象实体）

| 项 | 内容 |
|---|---|
| **职责** | 可被借出的具体资源，承载借出物类型差异化的罚款规则 |
| **关键属性** | `item_id: int`、`barcode: Barcode`、`title_id: int`、`item_type: ItemType`（枚举：CN_BOOK / FOREIGN_BOOK / CN_MAGAZINE / FOREIGN_MAGAZINE / THESIS）、`status: str`（available / borrowed / reserved / lost）、`location: str` |
| **关键行为** | `is_available() -> bool`、`mark_borrowed()`、`mark_available()`、`get_fine_rule() -> FineRule` |
| **约束** | `barcode` 全局唯一；处于 `borrowed` 的副本不可删除 |
| **子类** | `Book`（图书）、`Magazine`（杂志）、`Thesis`（论文）—— 各子类的 `item_type` 决定适用的 `FineRule` |
| **映射** | 基座简化：不建单副本明细行，用 `books.total_copies` / `available_copies` 计数（D-05） |

### 2.6 Loan（借阅记录，实体）⭐ 承载借出 / 归还 / 续借

| 项 | 内容 |
|---|---|
| **职责** | 记录一次借出行为，管理其生命周期：在借 → 续借 → 归还 / 超期 |
| **关键属性** | `record_id: int`、`user_id: int`、`book_id: int`、`borrow_date: datetime`、`due_date: datetime`、`return_date: datetime?`、`status: LoanStatus`（borrowing / returned / overdue / reserved）、`is_overdue: bool`、`renew_count: int` |
| **关键行为** | `can_renew(policy: BorrowPolicy, has_reservation: bool) -> (bool, str)`、`renew(policy) -> datetime`（返回新的 `due_date`）、`return_item() -> (days_overdue: int, fine: Money)`、`is_overdue_now() -> bool` |
| **约束** | 仅 `status = borrowing` 可续借；`renew_count < policy.renew_limit`；`now <= due_date`（**方案 A：超期不可续**）；图书未被他人预约（BR-009） |
| **关系** | 属于一个 `Reader`；指向一个 `BookTitle`（基座）或 `LibraryItem`；归还时产生 0 或 1 条 `FineRecord` |
| **映射** | 对应基座 `borrow_records` 表；续借需新增 `renew_count` 字段 |

> `Loan.can_renew()` 的失败原因字符串将直接作为接口 `message` 返回，保证 AC-017-2 / 5 / 6 / 7 的文案一致。

### 2.7 Reservation（预约，实体）

| 项 | 内容 |
|---|---|
| **职责** | 表达读者对某书目的排队请求 |
| **关键属性** | `reservation_id: int`、`user_id: int`、`title_id: int`、`reserved_at: datetime`、`expiry_date: datetime`、`queue_position: int`、`status: str`（active / fulfilled / expired / cancelled） |
| **关键行为** | `is_active() -> bool`、`expire()`、`fulfill()`、`position_in(queue) -> int` |
| **约束** | 同一读者对同一书目不得重复预约；**仅在可借数为 0 时可预约**；按 `reserved_at` FIFO 排队；有效期 7 天（BR-007 / BR-008） |
| **关系** | 属于一个 `Reader`，指向一个 `BookTitle` |
| **映射** | 基座复用 `borrow_records` 表以 `status = reserved` 表示（D-04）；领域模型保留独立实体 |

### 2.8 FineRecord（罚款记录，实体）

| 项 | 内容 |
|---|---|
| **职责** | 记录一次超期产生的罚款 |
| **关键属性** | `fine_id: int`、`record_id: int`、`user_id: int`、`days_overdue: int`、`amount: Money`、`created_at: datetime`、`paid: bool` |
| **关键行为** | `pay()`、`is_paid() -> bool` |
| **约束** | `amount` 由 `FineRule` 计算且不超过上限；本期**不实现缴款流程** |
| **关系** | 由 `Loan` 归还时产生；属于一个 `Reader` |
| **编码范围** | 本期仅建模（OPEN-02：是否落库待定） |

### 2.9 Review（图书评论与评分，实体）✨ 本次新增

| 项 | 内容 |
|---|---|
| **职责** | 表达读者对某本书的主观评价（评分 + 文本） |
| **关键属性** | `review_id: int`、`book_id: int`、`user_id: int`、`rating: Rating`、`content: str?`、`created_at: datetime`、`updated_at: datetime` |
| **关键行为** | `update(rating, content)`（覆盖更新）、`is_by(user_id) -> bool` |
| **约束** | `rating` 取值**整数 1–5**（`Rating` 值对象构造时校验，越界抛领域异常）；**(user_id, book_id) 唯一**（一人一书一条，BR-010） |
| **关系** | 属于一个 `Reader`，指向一个 `BookTitle` |
| **映射** | 本期**新增 `reviews` 表** |

---

## 3. 值对象

| 值对象 | 属性 | 不变式 | 用途 |
|---|---|---|---|
| `CardNo` | `value: str` | 匹配 `^CARD\d{4}\d{6}$` | 借阅证号 |
| `Barcode` | `value: str` | 非空、全局唯一 | 馆藏副本标识 |
| `ISBN` | `value: str` | 非空、全局唯一 | 书目标识 |
| `Money` | `amount: Decimal` | `amount >= 0`，保留 2 位小数 | 罚款金额、图书定价 |
| `Rating` | `value: int` | `1 <= value <= 5`，越界抛 `InvalidRatingError` | 评分校验集中在领域层 |
| `DateRange` | `start: date`、`end: date` | `start <= end` | 借期区间，提供 `days_overdue(now)` |

> `Rating` 的校验放在值对象里，是为了满足"业务规则不得写在 Controller"（宪法第 6 条）；后端接口只需捕获 `InvalidRatingError` 并转换为 `code=400`。

---

## 4. 策略 / 规则对象

### BorrowPolicy（借阅策略）

| 项 | 内容 |
|---|---|
| **职责** | 按读者类型定义借阅数量、期限与续借参数 |
| **关键属性** | `policy_id: int`、`reader_type: ReaderType`、`max_borrow: int`、`borrow_days: int`、`renew_limit: int`、`renew_days: int` |
| **关键行为** | `check_quota(current: int) -> bool`、`compute_due_date(from: date) -> date`、`can_renew(renew_count: int) -> bool` |
| **取值（BR-002 / BR-009）** | 本科 5 / 30 天；研究生 10 / 60 天；博士 15 / 90 天；专科 5 / 30 天；教师 20 / 90 天；**续借上限 1 次、每次 +30 天** |

### FineRule（罚款规则）

| 项 | 内容 |
|---|---|
| **职责** | 按借出物类型定义超期罚款费率与上限 |
| **关键属性** | `rule_id: int`、`item_type: ItemType`、`daily_rate: Money`、`upper_limit: Money?` |
| **关键行为** | `compute(days: int, price: Money?) -> Money` |
| **取值（BR-006）** | 中文图书 0.5 元/天；外文图书 1.0；中文杂志 0.2；外文杂志 0.5；论文 2.0；上限为定价 2 倍（无定价则 100 元） |

> 设计模式的落点：**Strategy**（不同读者类型的 `BorrowPolicy`、不同借出物类型的 `FineRule`）、**Repository**（持久化领域对象）、**Service Layer**（`CirculationService` 组织用例流程）、**DTO**（Pydantic 隔离接口层与领域层）。

---

## 5. 领域服务

| 服务 | 职责 | 关键方法 | 事务边界 |
|---|---|---|---|
| `CirculationService` | 借书 / 还书 / **续借** 的流程编排 | `borrow(user_id, book_id)`、`return(record_id)`、`renew(record_id)` | 每个方法一个事务；失败回滚（AC-011-8 / AC-012-3） |
| `ReservationService` | 预约排队、失效与递补 | `reserve(user_id, book_id)`、`expire_overdue()`、`next_in_queue(title_id)` | 预约创建为事务 |
| `CatalogService` | 检索与图书详情 | `search(keyword, category, author, location, page)` | 只读 |
| `ReviewService` | 评论保存与平均分 | `write_review(user_id, book_id, rating, content)`、`list_reviews(book_id)` | 保存为事务；平均分由**服务计算**（BR-011） |

---

## 6. 领域关系矩阵

| | Reader | BorrowCard | BookTitle | LibraryItem | Loan | Reservation | FineRecord | Review |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Reader** | — | 1 : 1 | | | 1 : N | 1 : N | 1 : N | 1 : N |
| **BorrowCard** | 1 : 1 | — | | | | | | |
| **BookTitle** | | | — | 1 : N（组合） | 1 : N | 1 : N | | 1 : N |
| **LibraryItem** | | | N : 1 | — | | | | |
| **Loan** | N : 1 | | N : 1 | | — | | 1 : 0..1 | |
| **Reservation** | N : 1 | | N : 1 | | | — | | |
| **FineRecord** | N : 1 | | | | N : 1 | | — | |
| **Review** | N : 1 | | N : 1 | | | | | — |

**继承关系**

```
Reader ──▷ StudentReader（本科 / 研究生 / 博士 / 专科）
       └──▷ TeacherReader

LibraryItem ──▷ Book
            ├──▷ Magazine
            └──▷ Thesis
```

---

## 7. 领域不变量（Invariants）

| 编号 | 不变量 | 保障位置 |
|---|---|---|
| INV-01 | 一个 Reader 至多持有一张有效 BorrowCard | `BorrowCard` / 唯一约束 |
| INV-02 | 同一读者的在借数量不超过其 `BorrowPolicy.max_borrow` | `CirculationService.borrow` |
| INV-03 | 存在超期未还时不得新借 | `CirculationService.borrow`（BR-004） |
| INV-04 | `Loan` 一旦 `returned` 不可再续借或再归还 | `Loan.renew` / `Loan.return_item` |
| INV-05 | `Loan.renew_count <= BorrowPolicy.renew_limit` | `Loan.can_renew`（BR-009） |
| INV-06 | 已超期的 `Loan` 不可续借（方案 A） | `Loan.can_renew`（AC-017-7） |
| INV-07 | 图书 `available_copies` 与在借 `Loan` 数量始终一致 | 事务内同步更新 |
| INV-08 | 同一 (Reader, BookTitle) 至多一条 `Review` | 联合唯一约束 + `ReviewService` |
| INV-09 | `Rating` 恒在 1–5 之间 | `Rating` 值对象 |
| INV-10 | 预约仅在可借数为 0 时创建，且按时间有序 | `ReservationService`（BR-007） |

---

## 8. 目标模型 ↔ 基座实现映射

| 领域对象 | 目标持久化 | 基座现状 | 差异 |
|---|---|---|---|
| `Reader` | `readers` | `users`（`role` 区分 reader/admin） | 表名与字段简化，行为一致 |
| `BookTitle` | `book_titles` | `books` | 一致 |
| `LibraryItem` | `library_items` | 无独立表，用计数列 | D-05，教学复杂度简化 |
| `Loan` | `loans` | `borrow_records` | 续借需**新增 `renew_count`** |
| `Reservation` | `reservations` | 复用 `borrow_records`（`status = reserved`） | D-04 |
| `BorrowCard` | `borrow_cards` | 无 | 本期仅建模 |
| `FineRule` / `FineRecord` | `fine_rules` / `fine_records` | 无（罚款按 0.5 元/天硬编码） | D-03 |
| `Review` | `reviews` | 无 | **本期新增** |
| `BorrowPolicy` | `borrow_policies` | 无（`users.max_borrow`） | 本期以常量表达 |

> 映射策略：**领域模型表达完整业务概念，数据库映射允许按教学复杂度简化**；所有简化已在偏差登记 D-01～D-06 中留痕，实现阶段不得以此为由删改规格。

---

## 9. 人工审查重点

| 指导书审查项 | 结论 |
|---|---|
| 1. 领域类是否来自业务，而非数据库表的机械翻译？ | ✅ `BorrowCard`、`BorrowPolicy`、`FineRule`、`Reservation` 均为业务概念，`LibraryItem` 与 `BookTitle` 已分离 |
| 2. 是否体现不同读者类型？ | ✅ `Reader → StudentReader / TeacherReader` + `reader_type` 枚举 |
| 3. 是否体现不同借出物类型？ | ✅ `LibraryItem → Book / Magazine / Thesis` + `item_type` 枚举 |
| 4. 借阅规则是否抽象为 BorrowPolicy？ | ✅ 含数量、期限、续借次数与天数 |
| 5. 罚款规则是否抽象为 FineRule？ | ✅ 含日费率与上限，按 `item_type` 取用 |
| 6. Loan 是否能表达借出与归还状态？ | ✅ `status`（borrowing / returned / overdue）+ `return_date` + `renew_count` |
| 7. Reservation 是否支持排队？ | ✅ `queue_position` + `reserved_at` FIFO + 7 天有效期 |
| 补充：两个新增功能是否建模？ | ✅ `Loan.renew()`（续借）与 `Review` + `Rating` + `ReviewService`（评论评分） |
