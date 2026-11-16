# Bug Diary

> 格式：现象 → 预期 vs 实际 → 假设 → 证据 → 根因 → 修复 → 验证 → 预防
> 每条必须注明 [所在层] ：Working / Staging / Local Repo / Remote

---

## BUG-001| Readme 文件名大小写与Git跟踪记录不一致

| 项目 | 内容 |
|---|---|
| 日期 | 2026-11-13 |
| 仓库 | engineering-learning-log（远程：shop） |
| 环境 | Windows 10/11 + Git Bash |
| 所在层 | **Staging（索引中记录的路径名）**，并随 commit 进入 Local Repo / Remote |
| 严重程度 | 低：Windows 上不影响使用；在大小写敏感的系统（Linux / macOS）上可能导致路径错误 |
| 耗时 | 约 40 分钟（含一次走错方向） |

### 1.问题现象
磁盘文件为`README.md`，Git跟踪的却是`readme.md`,`git status`显示`clean`,但在大小写敏感的系统中拉取或构建会报错。
### 2.预期 vs 实际
-预期：磁盘文件名，Git 跟踪的文件名、Github显示的文件三者一致，都是'README.md'
-实际:磁盘是'README.md',Git和Github显示是'readme.md',而且Git认为“没有变化”

### 3.假设（排查前列出，逐个验证）
| # | 假设 | 结论 |
|---|---|---|
| H1 | 改名后忘记 add / commit | ❌ status 是 clean，说明 Git 认为没有待提交的变化 |
| H2 | GitHub 网页有缓存，显示的是旧名字 | ❌ 本地 `git ls-files` 也是小写，问题不在 Remote |
| H3 | Git 没有感知到"只改大小写"的改名 | ✅ 见证据 |

### 4.证据
```bash
$ ls
README.md bug-diary.md ···

$ git ls-files | grep -i readme
readme.md

$ git config core.ignorecase
true
```
结论：Working 和Staging 里的文件名大小写不一致，但因为`ignorecase=ture`，Git把两者视为同一个文件

### 5.根本原因
-Windows文件系统默认**不区分大小写**:`readme.md` 和 `README.md`被视为同一个文件
Git for Windows 在init/clone时会自动设置，默认的`core.ignorecase`为`ture`,以配适这种文件系统,导致忽略了文件名大小写的变更。
**一句话：只改大小写的改名，必须通过Git来做，不能在资源管理器里直接改。**

### 6.解决方案
**走过的弯路（不要照做）：**
一开始我执行了 `git config core.ignorecase false`，想让 Git"识别大小写"。
这是错的：在 Windows 上手动改这个值，以后再改文件名大小写时，Git 会同时跟踪大小写两份记录，造成重复。
Git 官方文档也说明这个值由 Git 自动设置，不应手动修改。已于 2026-11-16 改回 `true`。

**正确修复（两步改名，绕开"不区分大小写"）：**
```bash
git status                          # 确认工作区干净
git mv readme.md tmp-readme.md      # 第 1 步：先改成完全不同的名字（Staging + Working 同步改）
git mv tmp-readme.md README.md      # 第 2 步：再改成目标名字
git status                          # 应显示 renamed: readme.md -> README.md
git commit -m "chore: 统一 README 文件名大小写"   # Staging → Local Repo
git push                            # Local Repo → Remote
```

### 7. 验证
```bash
$ git ls-files | grep -i readme
README.md                           # ✅ 索引已是大写

$ git config core.ignorecase
true                                # ✅ 配置已恢复默认

$ git log --oneline -1 --stat
90f2f88 ... readme.md => README.md  # ✅ Git 记录为改名，不是删除再新建
```
GitHub 网页刷新后显示 `README.md` ✅

### 8. 预防
- [ ] **改文件名一律用 `git mv`**，不在资源管理器里直接改
- [ ] 只改大小写时，用"先改成临时名、再改成目标名"的两步法
- [ ] **不手动修改 `core.ignorecase`**
- [ ] 改名后用 `git ls-files` 核对一遍，不只看资源管理器

### 9. 一句话教训
**status 显示 clean，不等于没有问题；它只说明"Git 认为"没有变化。** 要判断问题在哪一层，就要分别查看每一层：`ls` 看 Working，`git ls-files` 看 Staging，GitHub 网页看 Remote