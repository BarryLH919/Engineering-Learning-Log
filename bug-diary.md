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

## BUG-002| .gitignore _pycache__/少了一个下划线，以后 __pycache__/ 不会被忽略

| 项目 | 内容 |
|---|---|
| 日期 | 2026-11-17 |
| 仓库 | SCMP（远程：Intelligent-Engineering-Management） |
| 环境 | Windows 10/11 + Git Bash |
| 所在层 | **Working**，并随 commit 进入 Local Repo / Remote |
| 严重程度 | 高：错误导致__pycache__文件的忽略不生效，可能导致信息泄露、Git错误跟踪等问题 |
| 耗时 | 约 30 分钟 |


### 1.问题现象
.gitignore 里写成了 `_pycache__/`，少了一个下划线，导致以后 `__pycache__/` 不会被忽略。

### 2.预期 vs 实际
-预期：让git忽略`__pycache__/`文件
-实际: 由于拼写错误，导致git只忽略了`_pycache__/`，导致 `__pycache__/` 不会被忽略

### 3.假设（排查前列出，逐个验证）
| # | 假设 | 结论 |
|---|---|---|
| H1 | 修改.gitignore后忘记 add / commit | ❌ status 是 clean，说明 Git 认为没有待提交的变化 |
| H2 | 在.gitignore中__pycache__/拼写出现错误 | ✅ 见证据 _pycache__/ |

### 4.证据
```bash
$ git check-ignore -v src/__pycache__/x.pyc

没有输出

true
```
结论：.gitignore文件中为`_pycache__/`,与`__pycache__/`不一致，所以git忽略错误

### 5.根本原因
-写完忽略规则后没有验证。规则写出，git不会报错，只有验证才能发现！！！
**一句话：忽略规则写完，一定要验证，避免信息泄露等重大问题出现。**

### 6.解决方案
**正确修复：**
```bash
git status                          # 确认工作区干净
vim .gitigonre                      # 第 1 步：进入编辑器修改 _pycache__/错误并保存
git status                          # 应显示 modified: .gitignore
git add .gitignore                  # 应无任何报错返回
git commit -m "chore(.gitignore): pycache忽略格式输入错误修正"   # Staging → Local Repo
git push                            # Local Repo → Remote
```

### 7. 验证
```bash
$ git check-ignore -v src/__pycache__/x.pyc
.gitignore:3:__pycache__/                      # ✅ 忽略生效

$ git log --oneline -1 --stat
ecec6df ... chore(.gitignore): pycache忽略格式输入错误修正  # ✅ Git 记录正确符合修改要求
```


### 8. 预防
- [ ] **修改一定要验收、核查、保存**
- [ ] **Push后去GitHub上再次检查是否存在错误，若有则及时修改**

### 9. 一句话教训
**错误的格式拼写，不仅仅是一次"小问题"，可能会对后续开发产生巨大的影响，也容易让其他开发成员误解你的意思，造成更大的损失**

## BUG-003| 对已推送的Commit执行Amend导致历史分叉与冗余Merge

| 项目 | 内容 |
|---|---|
| 日期 | 2026-11-17 |
| 仓库 | SCMP（远程：Intelligent-Engineering-Management）|
| 环境 | Windows 10/11 + Git Bash |
| 所在层 | **Local Repo/Remote** |
| 严重程度 | 中：版本历史中会出现`同名且平行`的冗余节点，让其余开发者通过git log查看项目进度时产生误解等 |
| 耗时 | 约 20 分钟 |

### 1.问题现象
Git提交历史中连续出现了两个提交信息完全相同的`commit`（chore(src/tests): 新增.gitkeep文件），紧接着产生一个多余的merge branch合并提交记录。
### 2.预期 vs 实际
-预期：tests/.gitkeep和src/.gitkeep合并在同一个干净的提交记录中（线性历史），远程与本地指针完全一致，无多余的Merge节点。
-实际:远程保留了仅含tests/.gitkeep的旧提交，本地生成了包含两个.gitkeep的新提交，随后因推送别拒出发了pull合并，导致两个“同名但不同哈希数”的提交在历史中并存。

### 3.假设（排查前列出，逐个验证）
| # | 假设 | 结论 |
|---|---|---|
| H1 | 中途修改远程仓库名称，导致本地与远程跟踪分支同步异常 | ❌ git reflog完整还原了指针移动轨迹 |
| H2 | 根据git reflog移动轨迹 | ✅ 见证据 |

### 4.证据
```bash
git reflog
1a4f3da HEAD@{12:05}: commit:          chore(src/tests): 新增.gitkeep文件   ← 初始提交（仅含 tests/.gitkeep）并已 push
c78e86d HEAD@{12:07}: commit (amend):  chore(src/tests): 新增.gitkeep文件   ← 补加 src/.gitkeep 后 amend，生成新哈希
1e9a295 HEAD@{12:09}: pull origin main: Merge made by the 'ort' strategy.  ← push 被拒后执行 pull，产生自动合并
```
结论：根据reflog指针轨迹显示，初始提交已经push，之后补加新的commit,同时amend生成新哈希，push被拒后执行pull产生自动合并。

### 5.根本原因
违背了`不可重写已推送公共历史`的原则。`git commit --amend` 的本质不是在原节点上修改，而是基于父节点创建一个全新的Commit对象来替换本地的`HEDA`；由于原节点已经推送到`Remote`本地`Local Repo`与`Remote`的历史在同一节点下各自向前走了一步，形成了分叉；随后`git push`被拒，执行`git pull`将远程强行合入本地的哈希，最终留下**两个同名节点和合并节点**。
**一句话：已经Remote的Commit，不能通过amend取代，amend是基于原节点创建新的Commit**

### 6.解决方案

**正确修复（两步改名，绕开"不区分大小写"）：**
```bash
git status                          # 确认工作区干净
git reset --soft 1a4f3da~1          # 第 1 步：在本地重置到分叉前的公共父节点，保留工作区文件
git commit -m"chore(src/tests): 新增.gitkeep文件"    # 第 2 步：重新统一提交
git push --force-with=lease origin main             # Local Repo → Remote
```

### 7. 验证
```bash
$ git log --oneline --graph -n 5    # ✅ 确认提交历史呈一条直线，仅存在唯一一个提交记录，且无merge 记录
$ git status                        # ✅ 确认输出Your branch is up to date with origin/main 且工作区干净
```

### 8. 预防
- [ ] **推送前先核查**：执行`git commit`前养成运行`git status`和`git diff --cached`的习惯，确认所有相关目录都已进入暂存区。
- [ ] **严禁amend**已推送的节点，执行`amend`前先用`git status`确认状态，禁止已经push到Remote的commit执行amend,绝不重写已经推送到云仓库的节点。


### 9. 一句话教训
**推送前先核查，已推送到云的节点不得重写** 
