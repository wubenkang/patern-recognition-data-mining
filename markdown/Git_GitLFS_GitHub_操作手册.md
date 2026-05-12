# Git、Git LFS 与 GitHub 操作手册

## 适用场景

这份手册适合以下场景：

- 把本地文件夹初始化为 Git 仓库
- 把 Markdown、代码、笔记提交到 GitHub
- 用 Git LFS 管理 PDF、视频、压缩包等大文件
- 提交前检查，避免提交失败
- 提前 `git add` 或 `git commit` 后撤销
- 已经 push 到 GitHub 后安全回退

## 一句话理解 Git 工作区

Git 常见流程可以理解为 4 层：

```text
工作区 -> 暂存区 -> 本地仓库 -> 远程仓库 GitHub
```

对应命令：

```bash
git add      # 工作区 -> 暂存区
git commit   # 暂存区 -> 本地仓库
git push     # 本地仓库 -> 远程仓库
git pull     # 远程仓库 -> 本地仓库和工作区
```

Git LFS 的作用是：大文件不直接进入普通 Git 历史，而是由 LFS 存储真实文件，Git 仓库里保存一个指针文件。

## 常用目录示例

假设项目目录是：

```bash
/Users/wubenkang/Desktop/模式识别与数据挖掘
```

书籍目录是：

```bash
/Users/wubenkang/Desktop/模式识别与数据挖掘/books
```

如果 Git 仓库根目录是：

```bash
/Users/wubenkang/Desktop/模式识别与数据挖掘
```

那么 PDF 的 Git LFS 规则应写成：

```bash
books/*.pdf
```

如果 Git 仓库根目录就是：

```bash
/Users/wubenkang/Desktop/模式识别与数据挖掘/books
```

那么 PDF 的 Git LFS 规则应写成：

```bash
*.pdf
```

## 初始化 Git 仓库

进入项目根目录：

```bash
cd /Users/wubenkang/Desktop/模式识别与数据挖掘
```

初始化仓库：

```bash
git init
git branch -M main
```

检查当前状态：

```bash
git status
git branch --show-current
```

## 配置 Git 用户信息

查看当前配置：

```bash
git config --global user.name
git config --global user.email
```

设置用户名和邮箱：

```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
```

## Git LFS 基础配置

检查 Git LFS 是否安装：

```bash
git lfs version
```

如果未安装：

```bash
brew install git-lfs
```

启用 Git LFS：

```bash
git lfs install
```

跟踪 `books` 目录下的 PDF：

```bash
git lfs track "books/*.pdf"
```

跟踪所有 PDF：

```bash
git lfs track "*.pdf"
```

跟踪 `books` 目录及其子目录下的所有 PDF：

```bash
git lfs track "books/**/*.pdf"
```

常见大文件跟踪规则：

```bash
git lfs track "*.pdf"
git lfs track "*.zip"
git lfs track "*.mp4"
git lfs track "*.mov"
git lfs track "*.psd"
git lfs track "*.xlsx"
git lfs track "data/*.csv"
```

设置完 LFS 规则后，需要提交 `.gitattributes`：

```bash
git add .gitattributes
git commit -m "Track large files with Git LFS"
```

## `.gitattributes` 含义

示例：

```gitattributes
books/*.pdf filter=lfs diff=lfs merge=lfs -text
```

含义：

| 片段 | 含义 |
| --- | --- |
| `books/*.pdf` | 匹配仓库根目录下 `books` 文件夹里的 PDF |
| `filter=lfs` | 提交时交给 Git LFS 处理 |
| `diff=lfs` | 使用 LFS 的方式处理差异比较 |
| `merge=lfs` | 使用 LFS 的方式处理合并 |
| `-text` | 不当作文本处理，不做换行符转换 |

注意：如果写成 `bok/*.pdf`，但实际目录叫 `books`，则规则不会生效。

## 提交前检查命令

### 1. 检查当前位置和仓库状态

```bash
pwd
git status
git branch --show-current
git remote -v
```

### 2. 检查 Git LFS 状态

```bash
git lfs version
git lfs env
git lfs track
cat .gitattributes
```

### 3. 检查某个文件是否被 LFS 接管

```bash
git check-attr filter -- books/你的文件.pdf
```

如果输出类似下面这样，说明会走 Git LFS：

```text
books/你的文件.pdf: filter: lfs
```

### 4. 预览将要添加的文件

`-n` 表示 dry run，只预览，不真正添加。

```bash
git add -n books/*.md
git add -n books/*.pdf
```

### 5. 添加文件

```bash
git add .gitattributes
git add books/*.md
git add books/*.pdf
```

### 6. 检查暂存区

```bash
git status --short
git diff --cached --name-only
git diff --cached --stat
git diff --cached --check
```

### 7. 检查 LFS 暂存状态

```bash
git lfs status
```

### 8. 检查大文件

检查 `books` 目录下超过 50MB 的文件：

```bash
find books -type f -size +50M -print
```

这些文件如果要提交到 GitHub，建议先确认是否被 Git LFS 接管。

### 9. 提交

```bash
git commit -m "Add learning plans and books"
```

### 10. 推送前检查远程仓库

```bash
git remote -v
git ls-remote origin
```

如果使用 GitHub CLI：

```bash
gh auth status
gh repo view
```

### 11. 推送前 dry run

```bash
git push --dry-run origin main
git lfs push --dry-run origin main
```

### 12. 正式推送

```bash
git push origin main
```

## 标准提交流程

```bash
cd /Users/wubenkang/Desktop/模式识别与数据挖掘

git status
git lfs track
cat .gitattributes

git check-attr filter -- books/你的文件.pdf

git add .gitattributes
git add books/*.md
git add books/*.pdf

git diff --cached --stat
git lfs status

git commit -m "Add learning plans and books"

git push --dry-run origin main
git lfs push --dry-run origin main

git push origin main
```

## 创建 GitHub 仓库并推送

先确认 GitHub CLI 登录：

```bash
gh auth status
```

未登录则执行：

```bash
gh auth login
```

创建公开仓库并推送：

```bash
gh repo create big-data-learning-plans --public --source . --remote origin --push
```

创建私有仓库并推送：

```bash
gh repo create big-data-learning-plans --private --source . --remote origin --push
```

如果远程仓库已经存在：

```bash
git remote add origin https://github.com/你的用户名/big-data-learning-plans.git
git push -u origin main
```

## 上传大文件的完整 Shell 脚本

这个脚本适用于：仓库根目录是 `/Users/wubenkang/Desktop/模式识别与数据挖掘`，大文件在 `books/` 目录中。

```bash
#!/usr/bin/env bash
set -euo pipefail

REPO_DIR="/Users/wubenkang/Desktop/模式识别与数据挖掘"
LFS_PATTERN="books/*.pdf"
COMMIT_MSG="Add books and learning plans"
BRANCH="main"

cd "$REPO_DIR"

if ! command -v git >/dev/null 2>&1; then
  echo "Error: git 未安装"
  exit 1
fi

if ! git lfs version >/dev/null 2>&1; then
  echo "Error: Git LFS 未安装"
  echo "请先安装：brew install git-lfs"
  exit 1
fi

if [ ! -d ".git" ]; then
  git init
  git branch -M "$BRANCH"
fi

echo "仓库目录：$REPO_DIR"
echo "LFS 规则：$LFS_PATTERN"
echo

read -r -p "确认要设置 Git LFS、提交并推送吗？输入 yes 继续: " confirm
if [ "$confirm" != "yes" ]; then
  echo "已取消"
  exit 0
fi

git lfs install
git lfs track "$LFS_PATTERN"

git add .gitattributes
git add books/*.md
git add books/*.pdf

git status --short
git diff --cached --stat
git lfs status

git commit -m "$COMMIT_MSG" || echo "没有新的提交内容"

git push --dry-run origin "$BRANCH"
git lfs push --dry-run origin "$BRANCH"

read -r -p "dry run 已完成，确认正式 push？输入 yes 继续: " push_confirm
if [ "$push_confirm" != "yes" ]; then
  echo "已取消 push"
  exit 0
fi

git push origin "$BRANCH"

echo "完成"
```

## 只上传 Markdown 的脚本

如果只想上传学习计划 Markdown，不上传 PDF：

```bash
#!/usr/bin/env bash
set -euo pipefail

REPO_DIR="/Users/wubenkang/Desktop/模式识别与数据挖掘"
COMMIT_MSG="Add markdown learning plans"
BRANCH="main"

cd "$REPO_DIR"

git add books/*.md

git status --short
git diff --cached --stat

git commit -m "$COMMIT_MSG" || echo "没有新的提交内容"

git push --dry-run origin "$BRANCH"

read -r -p "确认正式 push？输入 yes 继续: " confirm
if [ "$confirm" != "yes" ]; then
  echo "已取消"
  exit 0
fi

git push origin "$BRANCH"
```

## 撤销操作速查

### 已经 `git add`，但还没 commit

撤销某个文件的暂存：

```bash
git restore --staged books/xxx.pdf
```

撤销全部暂存：

```bash
git restore --staged .
```

这不会删除文件，只是把文件从暂存区拿回工作区。

### 已经修改文件，但想丢弃工作区修改

谨慎使用：这会丢掉本地未提交的修改。

```bash
git restore books/xxx.md
```

丢弃全部工作区修改：

```bash
git restore .
```

### 已经 commit，但还没 push

撤销最近一次 commit，保留修改并保持暂存：

```bash
git reset --soft HEAD~1
```

撤销最近一次 commit，保留修改但取消暂存：

```bash
git reset HEAD~1
```

修改最近一次 commit message：

```bash
git commit --amend -m "新的提交信息"
```

### commit 里多加了某个文件，还没 push

```bash
git reset --soft HEAD~1
git restore --staged books/不想提交的文件.pdf
git commit -m "正确的提交信息"
```

### 文件已经被 Git 跟踪，但想保留本地文件、不再纳入 Git

```bash
git rm --cached books/xxx.pdf
```

整个目录：

```bash
git rm -r --cached books/
```

然后提交：

```bash
git commit -m "Stop tracking large files"
```

### 已经 push 到 GitHub，想安全撤销

推荐用 `revert`，它会生成一个反向提交，不改写历史。

```bash
git revert HEAD
git push origin main
```

撤销指定 commit：

```bash
git revert <commit_id>
git push origin main
```

### 已经 push，但确实要改历史

个人仓库并确认没人协作时才考虑：

```bash
git reset --soft HEAD~1
git push --force-with-lease origin main
```

`--force-with-lease` 比 `--force` 安全一些。

## Git LFS 撤销和调整

查看当前 LFS 规则：

```bash
git lfs track
cat .gitattributes
```

取消某个规则：

```bash
git lfs untrack "books/*.pdf"
```

提交规则变化：

```bash
git add .gitattributes
git commit -m "Update Git LFS tracking rules"
```

注意：`git lfs untrack` 只影响后续文件，不会自动改写已经提交的历史。

## 大文件误提交处理

### 情况 1：大文件已经 add，但还没 commit

```bash
git restore --staged books/xxx.pdf
git lfs track "books/*.pdf"
git add .gitattributes
git add books/xxx.pdf
git lfs status
```

### 情况 2：大文件已经 commit，但还没 push

```bash
git reset --soft HEAD~1
git restore --staged books/xxx.pdf
git lfs track "books/*.pdf"
git add .gitattributes
git add books/xxx.pdf
git commit -m "Add large file with Git LFS"
```

### 情况 3：大文件已经 push 到 GitHub 普通 Git 历史

优先做法：

```bash
git rm --cached books/xxx.pdf
git lfs track "books/*.pdf"
git add .gitattributes
git add books/xxx.pdf
git commit -m "Move large file to Git LFS"
git push origin main
```

如果 GitHub 已经拒绝 push，通常需要先改本地 commit，再重新提交。

如果大文件已经进入远程历史并且必须彻底清除，需要使用 `git filter-repo` 或 BFG 这类历史改写工具。这个操作风险较高，执行前建议先完整备份仓库。

## 常见错误与处理

### 错误：remote origin already exists

说明远程仓库已经配置过。

查看：

```bash
git remote -v
```

修改：

```bash
git remote set-url origin https://github.com/你的用户名/仓库名.git
```

### 错误：src refspec main does not match any

通常是还没有 commit，或当前分支不叫 `main`。

检查：

```bash
git status
git branch --show-current
```

修复：

```bash
git branch -M main
git add .
git commit -m "Initial commit"
git push -u origin main
```

### 错误：文件超过 GitHub 100MB 限制

说明大文件没有被 Git LFS 正确接管。

检查：

```bash
git check-attr filter -- books/xxx.pdf
git lfs status
```

修复：

```bash
git reset --soft HEAD~1
git restore --staged books/xxx.pdf
git lfs track "books/*.pdf"
git add .gitattributes
git add books/xxx.pdf
git commit -m "Add PDF with Git LFS"
```

### 错误：Authentication failed

检查 GitHub CLI 登录：

```bash
gh auth status
```

重新登录：

```bash
gh auth login
```

## 推荐日常工作流

### 每次提交前

```bash
git status
git lfs track
cat .gitattributes
git diff --cached --stat
git lfs status
```

### 每次提交后、push 前

```bash
git log --oneline -5
git push --dry-run origin main
git lfs push --dry-run origin main
```

### 正式 push

```bash
git push origin main
```

## 最常用命令总结

```bash
# 查看状态
git status

# 查看 LFS 规则
git lfs track
cat .gitattributes

# 跟踪 PDF
git lfs track "books/*.pdf"

# 添加文件
git add .gitattributes
git add books/*.md
git add books/*.pdf

# 检查暂存区
git diff --cached --stat
git lfs status

# 提交
git commit -m "Add learning plans and books"

# 推送前预演
git push --dry-run origin main
git lfs push --dry-run origin main

# 推送
git push origin main

# add 错了
git restore --staged .

# commit 错了但没 push
git reset --soft HEAD~1

# 已经 push 后安全撤销
git revert HEAD
git push origin main
```

