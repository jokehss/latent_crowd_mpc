# AI 科研代码 Git 工作流模板

> 适用项目：`latent_crowd_mpc`（拥挤人群导航 + latent world model + MPC）
> 工作模式：WSL (Codex 写代码) → GitHub (同步中心) → PowerShell (Claude Code 审查)

---

## 总体流程

```
从 main 开分支
    ↓
Codex 在 WSL 写代码
    ↓
本地测试
    ↓
commit
    ↓
push 到 GitHub
    ↓
创建 Pull Request
    ↓
Claude Code 在 PowerShell 审查
    ↓
GitHub merge 到 main
    ↓
删除临时分支
    ↓
WSL / PowerShell 都回到 main 并 pull 最新代码
```

---

## 0. 每次开始前：确认 main 是干净的

### WSL / Codex 端

```bash
cd ~/latent_crowd_mpc
git switch main
git pull
git status
```

你希望看到：

```
On branch main
Your branch is up to date with 'origin/main'.
nothing to commit, working tree clean
```

**含义：**

- 你现在在 main 分支
- 本地 main 和 GitHub main 一致
- 没有未提交改动
- 可以安全开新分支

---

## 1. 从 main 开一个新分支

按照任务类型命名：

```bash
git switch -c setup/ci
```

或者：

```bash
git switch -c feat/world-model-interface
git switch -c fix/collision-metric
git switch -c exp/rssm-smoke-train
git switch -c docs/experiment-protocol
```

### 命名规则

| 前缀 | 用途 | 示例 |
|------|------|------|
| `setup/...` | 项目配置、CI、工作流 | `setup/project-skeleton`, `setup/ci` |
| `feat/...` | 新功能 | `feat/world-model-interface`, `feat/rssm-dynamics` |
| `fix/...` | 修 bug | `fix/collision-metric` |
| `exp/...` | 实验脚本、实验配置 | `exp/rssm-smoke-train` |
| `docs/...` | 文档 | `docs/experiment-protocol` |
| `refactor/...` | 重构 | `refactor/mpc-planner-interface` |

例如你下一步 CI 就应该叫：

```bash
git switch -c setup/ci
```

---

## 2. 启动 Codex 写代码

在 WSL 当前分支里：

```bash
codex
```

然后给 Codex 明确任务，例如：

```
Read AGENTS.md.

Add a minimal GitHub Actions CI workflow.

Only modify:
- .github/workflows/ci.yml
- .python-version
- README.md if needed

Do not add algorithms.
Do not add training code.
Keep the change minimal.
```

> **原则：** Codex 只能在当前任务分支上写代码，不要让它直接改 main。

---

## 3. Codex 写完后，本地检查

先看改了哪些文件：

```bash
git status
git diff --stat
```

如果想看具体内容：

```bash
git diff
```

> 如果改动太多或不符合任务范围，先不要提交。

---

## 4. 本地运行测试

当前项目至少跑：

```bash
python -m pytest -q
```

如果刚改了依赖，可以先装：

```bash
pip install -e ".[dev]"
python -m pytest -q
```

目标是看到类似：

```
1 passed
```

以后测试多了，可能是：

```
12 passed
```

---

## 5. 提交 commit

先 add：

```bash
git add .
```

然后 commit：

```bash
git commit -m "Add CI workflow"
```

commit message 建议简短清楚：

- `Initialize project skeleton`
- `Add CI workflow`
- `Add world model interface`
- `Fix collision metric`
- `Add RSSM smoke config`

如果只想提交某几个文件，不要用 `git add .`，可以指定：

```bash
git add .github/workflows/ci.yml .python-version README.md
git commit -m "Add CI workflow"
```

---

## 6. Push 到 GitHub

第一次 push 新分支时：

```bash
git push -u origin setup/ci
```

如果分支名是别的，就改掉：

```bash
git push -u origin feat/world-model-interface
```

> `-u` 的意思是建立本地分支和 GitHub 远程分支的跟踪关系。

以后同一个分支再次 push，可以直接：

```bash
git push
```

---

## 7. GitHub 创建 PR

打开仓库：

```
https://github.com/jokehss/latent_crowd_mpc
```

GitHub 一般会提示：

> `setup/ci had recent pushes`
> **Compare & pull request**

点击它。

**确认：**

```
base: main
compare: setup/ci
```

> 含义：我要把 setup/ci 合并进 main
>
> **注意：** base 一般永远选 main，compare 选你的任务分支。

**PR 标题示例：**

> Add CI workflow

**PR 描述示例：**

> Add a minimal GitHub Actions workflow that installs the package and runs pytest on PRs and pushes to main.
>
> No algorithms, datasets, training code, or evaluation logic are introduced.

然后点击：

> **Create pull request**

---

## 8. PowerShell / Claude Code 审查 PR

### PowerShell 端拉取分支

```bash
cd D:\claude_work\latent_crowd_mpc-review
git fetch origin
git switch -c setup/ci --track origin/setup/ci
```

如果提示分支已经存在：

```bash
git switch setup/ci
git pull
```

### 启动 Claude Code

```bash
claude
```

给 Claude 输入：

```
Read CLAUDE.md.

Review the current branch setup/ci as a pull request against main.

Do not modify files.

Focus on:
1. Whether the change is minimal and correct.
2. Whether it improves reproducibility.
3. Whether it introduces unnecessary complexity.
4. Whether tests pass.
5. Whether it is safe to merge.

Output:
- Verdict: APPROVE or REQUEST_CHANGES
- Critical Issues
- Research Risks
- Required Fixes
```

---

## 9. 如果 Claude APPROVE

回到 GitHub PR 页面，点击：

> **Merge pull request**

然后点击：

> **Confirm merge**

合并后，如果 GitHub 提供按钮：

> **Delete branch**

点击删除远程分支。

---

## 10. 如果 Claude REQUEST_CHANGES

回到 WSL 的同一个分支继续改：

```bash
cd ~/latent_crowd_mpc
git switch setup/ci
```

让 Codex 修：

```bash
codex
```

修完后：

```bash
git status
git diff --stat
python -m pytest -q
git add .
git commit -m "Address review comments"
git push
```

> **注意：** 这里不用重新创建 PR。原来的 PR 会自动更新。

然后 PowerShell 再审查：

```bash
cd D:\claude_work\latent_crowd_mpc-review
git switch setup/ci
git pull
claude
```

---

## 11. PR merge 后，WSL 清理

回到 WSL：

```bash
cd ~/latent_crowd_mpc
git switch main
git pull
git branch -d setup/ci
git fetch --prune
git status
```

如果删除本地分支时报错，但你确认 PR 已经 merge，可以强制删除：

```bash
git branch -D setup/ci
```

---

## 12. PR merge 后，PowerShell 清理

PowerShell：

```bash
cd D:\claude_work\latent_crowd_mpc-review
git switch main
git pull
git branch -d setup/ci
git fetch --prune
git status
```

如果需要强制删除：

```bash
git branch -D setup/ci
```

---

## 13. 每次任务的完整命令模板

### WSL / Codex

```bash
cd ~/latent_crowd_mpc

git switch main
git pull
git status

git switch -c setup/ci

codex

git status
git diff --stat
python -m pytest -q

git add .
git commit -m "Add CI workflow"
git push -u origin setup/ci
```

### GitHub

1. 打开仓库
2. 点击 **Compare & pull request**
3. 确认 base: **main**
4. 确认 compare: **setup/ci**
5. 写 PR 标题和描述
6. **Create pull request**

### PowerShell / Claude Code

```bash
cd D:\claude_work\latent_crowd_mpc-review

git fetch origin
git switch -c setup/ci --track origin/setup/ci

claude
```

Claude approve 后，GitHub merge。

### 合并后两边清理

**WSL：**

```bash
cd ~/latent_crowd_mpc

git switch main
git pull
git branch -d setup/ci
git fetch --prune
git status
```

**PowerShell：**

```bash
cd D:\claude_work\latent_crowd_mpc-review

git switch main
git pull
git branch -d setup/ci
git fetch --prune
git status
```

---

## 14. 最常用 Git 命令速查

| 命令 | 作用 |
|------|------|
| `git status` | 看当前有没有未提交改动 |
| `git branch` | 看本地分支 |
| `git branch -a` | 看本地分支 + 远程分支 |
| `git switch main` | 切换到 main |
| `git switch -c feat/name` | 从当前分支创建新分支 |
| `git pull` | 拉取当前分支的 GitHub 最新代码 |
| `git fetch origin` | 只更新远程信息，不改当前代码 |
| `git fetch --prune` | 清理已经被 GitHub 删除的远程分支引用 |
| `git add .` | 把所有修改加入暂存区 |
| `git commit -m "message"` | 提交一个本地版本 |
| `git push -u origin branch-name` | 第一次推送新分支 |
| `git push` | 后续推送当前分支 |
| `git branch -d branch-name` | 删除已经合并的本地分支 |
| `git branch -D branch-name` | 强制删除本地分支 |
| `git log --oneline --graph --decorate --all -n 30` | 命令行查看分支图 |

---

## 15. 简化版（贴到 README 或笔记用）

```
每个任务都从 main 开新分支。

WSL / Codex:
1. git switch main
2. git pull
3. git switch -c <branch>
4. codex
5. git status
6. git diff --stat
7. python -m pytest -q
8. git add .
9. git commit -m "<message>"
10. git push -u origin <branch>

GitHub:
1. Create Pull Request
2. base = main
3. compare = <branch>

PowerShell / Claude:
1. git fetch origin
2. git switch -c <branch> --track origin/<branch>
3. claude
4. Review only, do not modify files

After APPROVE:
1. Merge PR on GitHub
2. Delete remote branch

WSL cleanup:
1. git switch main
2. git pull
3. git branch -d <branch>
4. git fetch --prune

PowerShell cleanup:
1. git switch main
2. git pull
3. git branch -d <branch>
4. git fetch --prune
```

---

> 这就是你当前项目的标准 Git 工作流。后面我们加 CI、branch protection、Issue 模板、自动 Claude review，都是在这套基础上升级。
