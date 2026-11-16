# Month 01 - Week 04：用 Git 管理真实项目 + 月底闭卷验收

> **核心目标**：将前三周的 Git 能力迁移到真实产品项目中，跑通“产品想法 → Issue 拆解 → Branch 开发 → PR 审查 → Merge 交付”的完整闭环，并通过首月 90 分钟闭卷硬验收。
> **本周边界**：只搭产品工程骨架与文档草案，**绝不提前实现**登录、数据库、部署等后续月份功能。

---

## 一、 本周核心三问（快速对齐）

### 1. 本周应该做什么？
- **Day 1（补遗留 + 建日志）**：把 `git-collaboration-lab` 推送到 GitHub；盘点首月 KPI 缺口；建立 `engineering-learning-log` 仓库并补齐 ≥3 条 `bug-diary.md`。**（已完成）**
- **Day 2（建产品正式 Repo）**：搭建年度产品仓库骨架（`README.md`、`docs/`、`src/`、`tests/`、`.gitignore`），编写 `product-scope.md` 和 `roadmap.md` 并走完本周 PR #1。
- **Day 3（任务拆解）**：在产品仓库创建 ≥6 个边界清晰的 `Issues`（涵盖 MVP 范围、理论体系、规则库格式、核心流程、使用边界、用户访谈），并关联至 `MVP v0.1` Milestone。
- **Day 4（标准 Workflow 实战）**：脱离 AI 辅助，挑选 Issue #3（规则库数据格式草案），独立走完 `Branch → 3 个有意义 Commits → Push → PR → 自审 → Merge → Pull`（本周 PR #2）。
- **Day 5（复测 + 预演）**：完成 7 天间隔知识复测；体检两个核心仓库；在练习库中闭卷预演完整流程，刻意制造并解决 1 次 Merge Conflict（本周 PR #3），演练 1 次远程 `revert`。
- **周末（月度硬验收）**：在【考核-第1月】窗口完成 90 分钟闭卷实操 + 口头答辩（目标 ≥ 80 分）。

### 2. 容易卡在哪？（高频避坑）
- **范围蔓延**：看到产品仓库建好了，忍不住开始写代码或陷入产品细节纠结（严格执行 70 分钟时间盒，不确定直接写“待验证”）。
- **PR 合并方式选错**：合并时必须选 **`Create a merge commit`**，不要选 Squash（否则会吞掉 3 个 commit 的演进证据，且本地 `git branch -d` 会提示未完全合并）。
- **错误恢复选错命令**：已 `push` 到远程 `main` 的错误提交，必须用 **`git revert`** 撤销后再 `push`，严禁使用 `reset --hard` 改写已共享历史。
- **终端环境混用**：同一个仓库固定使用一种 Bash 环境（Git Bash 或 WSL），不要在 PowerShell 和 Bash 之间反复横跳。
- **报障慌乱**：遇到报错不看状态直接瞎改（每天开工前必须默写**出错处理 6 步**：`停 → git status → 必要时 diff/log → 判断 Working/Staging/Local/Remote 四层 → 最小影响操作 → 再验证`）。

### 3. 本周绝对重点是什么？
- **工程交付闭环**：理解 README、Issue、Milestone、PR 如何把产品规划转化为可追溯的工程资产。
- **独立肌肉记忆**：Day 4 起核心流程严禁问 AI“下一步怎么做”，只能查 `git --help` 和 GitHub 官方中文文档。
- **可核验证据链**：所有月度 KPI 必须沉淀为真实的 GitHub URL 或 Commit Hash。

---

## 二、 每日执行打卡清单

### Day 1（11-07）：遗留补齐 + 建立 `engineering-learning-log`
- [ ] **第 0 步**：将 `git-collaboration-lab` 关联远程并 `push` 至 GitHub；统计历史仓库 Commit / PR / Conflict 数量。
- [ ] **每日默写**：闭卷默写“出错处理 6 步”。
- [ ] **建仓与文档**：创建 `engineering-learning-log`，完善 `README.md`、`docs/learning-principles.md` 及 `month-01/week-01~04.md`。
- [ ] **错误复盘**：创建 `bug-diary.md`，按“现象 → 假设 → 证据 → 根因 → 修复 → 预防”记录 ≥3 条真实踩坑（注明所在层）。
- [ ] **提交推送**：形成 ≥2 个有意义 commits 并 `push` 到 GitHub。

### Day 2（11-17）：年度产品正式 Repo 骨架
- [ ] **建仓防泄密**：创建产品私有仓库，配置 `.gitignore`（含 `.env`、`.venv/`、`__pycache__/`、`*.pyc`）及目录骨架，提交至 `main`。
- [ ] **完善 README**：回答产品定位、目标用户、核心痛点、核心价值、用户流程、MVP 边界、Roadmap 共 7 个核心问题。
- [ ] **文档 PR #1**：新建分支编写 `docs/product-scope.md` 与 `docs/roadmap.md`，发起 PR 并合并回 `main`。

### Day 3（11-18）：Issue 拆解 + Milestone 规划
- [ ] **创建 Issues**：创建 ≥6 个产品 Issues（包含 Background / Scope / Acceptance Criteria / Out of Scope）。
- [ ] **挂载里程碑**：创建 `MVP v0.1` Milestone，将属于首版范围的 Issues 归入其中。

### Day 4（11-19 ~ 11-20）：完整标准 Workflow（无 AI 辅助）
- [ ] **同步与建分支**：从干净的 `main` 拉出分支 `docs/rule-schema`（对应 Issue #3）。
- [ ] **3 次有意义提交**：
  - Commit 1：建立 `docs/rule-schema.md` 字段结构定义。
  - Commit 2：补充 2 条带出处的示例规则。
  - Commit 3：补充“输出如何追溯到规则”的说明并自检验收标准。
- [ ] **PR 与审查（PR #2）**：Push 分支 → 创建 PR（正文写 `Closes #3`）→ 在 `Files changed` 留下自审评论 → 选择 `Create a merge commit` 合并。
- [ ] **本地清理**：切回 `main` → `git pull` → `git branch -d` 删除本地分支 → `git fetch --prune` 清理远程跟踪引用。

### 缓冲日（11-20）
- [ ] 补齐 Day 1–4 遗留项；若已全部完成则休息蓄力。

### Day 5（11-21）：间隔复测 + 仓库体检 + 月考预演
- [ ] **7 天间隔复测**：闭卷回答 `restore/reset` 区别、`reflog` 有效期、停止跟踪文件 3 步、`amend` 与错分支迁移、密钥泄露处置 5 步。
- [ ] **双库体检**：检查 `engineering-learning-log` 与产品 Repo 的状态、提交图、分支清理及 GitHub 关联情况。
- [ ] **实战预演（PR #3）**：从空目录闭卷演练完整流程，刻意制造并解决 1 次 `Merge Conflict`，并在独立练习库演练 1 次远程 `revert` + `push`。

### 周末（11-22）：首月 90 分钟闭卷硬验收
- [ ] 在【考核-第1月】窗口完成 90 分钟闭卷实操（初始化 → 基础控制 → 分支与 3 次提交 → PR 全流程 → 远程 `revert`）与口头答辩，提交输出与链接给教练评分。

---

## 三、 首月硬验收 KPI 与证据链登记

| 考核指标 | 首月最低要求 | 本周最终实际数据 | 可核验证据链接 / Commit Hash |
| :--- | :--- | :--- | :--- |
| **Repository 数量** | ≥ 3 个 | | `learning-log`: <br> `产品 Repo`: <br> `collaboration-lab`: |
| **Commit 累计** | 20 – 25 个 | | （各仓库真实提交自然累计） |
| **Pull Request** | 3 – 5 个 | | PR #1 (docs): <br> PR #2 (Issue #3): <br> PR #3 (预演): |
| **Issue 数量** | ≥ 6 个 | | 产品 Repo Issues 列表 URL: |
| **Milestone** | ≥ 1 个 (`MVP v0.1`) | | Milestone URL: |
| **Merge Conflict** | ≥ 2 次 | | 解决记录 / PR 链接: |
| **Bug Diary** | ≥ 3 条 | | `bug-diary.md` 链接: |
| **远程 Revert 恢复** | ≥ 1 次 | | Revert Commit Hash / Repo: |
| **月底闭卷总分** | ≥ 80 分（独立完成） | | 考核成绩与链接: |

---

## 四、 本周复盘与卡点记录

- **本周最大卡点与根因**：
  - 卡点：Commit的`type`写不对：
  type 是给这次提交贴的分类标签，回答的是"这次改动属于哪一类工作"。 它不描述你做了什么，只说明这是哪一类活。
```text 
  第 1 月只需要熟练用好前 4 个：docs、chore、feat、fix。
  判断方法：按顺序问 3 个问题
① 只改了文档内容吗？（.md 说明、记录、手册）
   └─ 是 → docs
② 改的是产品功能吗？
   ├─ 增加了用户能用的新能力 → feat
   ├─ 修好了原本出错的行为 → fix
   └─ 行为不变、只改内部结构 → refactor
③ 都不是？（配置、目录整理、改名、依赖、清理）
   └─ chore 
``` 
  - 根因：**缺少规范化学习**

- **闭卷预演/考试中暴露的薄弱环节**：
  - 暂无
- **首月结束后，我真正掌握的工程习惯**：
  - 暂无