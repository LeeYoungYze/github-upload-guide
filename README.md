# 从零把项目上传到 GitHub（Windows 完整教程）

适合第一次用 Git、手里有个能跑的本地项目（比如 `E:\MyPoints`），想把它传到 GitHub 上的人。
以 Windows + PowerShell 为例，命令在 CMD 里同样可用；图形界面方案见第五、六节。

> 只想快速跑一遍：直接跳到 [第二节「5 分钟速通」](#25-分钟速通)。

## 目录

- [一、一次性准备](#一一次性准备)
- [二、5 分钟速通](#二5-分钟速通)
- [三、四个概念，理解 Git 在干什么](#三四个概念理解-git-在干什么)
- [四、逐条解释那几条命令](#四逐条解释那几条命令)
- [五、用 GitHub CLI 一条命令搞定](#五用-github-cli-一条命令搞定)
- [六、不想碰命令行：图形界面方案](#六不想碰命令行图形界面方案)
- [七、.gitignore：决定什么不该上传](#七gitignore决定什么不该上传)
- [八、八个最常见的坑](#八八个最常见的坑)
- [九、日常循环与命令速查](#九日常循环与命令速查)
- [十、上传前检查清单](#十上传前检查清单)

---

## 一、一次性准备

### 1. 安装 Git

下载 <https://git-scm.com/download/win>，一路默认安装即可。装完打开 PowerShell 验证：

```powershell
git --version
# git version 2.55.0.windows.3
```

### 2. 注册 GitHub 账号

<https://github.com/signup>。用户名会出现在仓库地址里，起个好记的。

### 3. 告诉 Git 你是谁

每一次提交都会记录作者信息，所以必须先配置（只需一次）：

```powershell
git config --global user.name "你的名字或 GitHub 用户名"
git config --global user.email "你的邮箱@example.com"
```

查看是否配置成功：

```powershell
git config --global --list
```

### 4. 让 Git 能登录 GitHub

三个方案，选一个：

| 方案 | 难度 | 体验 |
| --- | --- | --- |
| GitHub CLI（`gh auth login`） | 简单 | 一次授权，之后全自动，还能直接建仓库 |
| Personal Access Token | 中等 | 每次 push 要粘一次长串 token |
| SSH 密钥 | 中等 | 配好后永久免密，偏极客 |

**推荐 GitHub CLI**：<https://cli.github.com/>，装完执行

```powershell
gh auth login
```

依次选：`GitHub.com` → `HTTPS` → `Login with a web browser`，屏幕上会给一个 8 位验证码，浏览器里粘进去授权即可。

**Personal Access Token 方案**：GitHub 头像 → `Settings` → 左侧最下 `Developer settings` → `Personal access tokens` → `Tokens (classic)` → `Generate new token`，勾选 `repo` 权限，生成后复制保存（只显示一次）。以后 push 提示输密码时，粘这个 token 当密码。

**SSH 方案**：

```powershell
ssh-keygen -t ed25519 -C "你的邮箱"
# 一路回车，默认存在 ~/.ssh/id_ed25519

Get-Content ~/.ssh/id_ed25519.pub
# 复制输出，粘到 GitHub → Settings → SSH and GPG keys → New SSH key
```

之后仓库地址用 `git@github.com:用户名/仓库名.git` 这种形式。

---

## 二、5 分钟速通

假设项目在 `E:\MyPoints`，要传成一个叫 `my-project` 的公开仓库。

### 路线 A：命令行 + 网页建仓

```powershell
cd E:\MyPoints
git init
git add -A
git commit -m "第一次提交"
```

然后浏览器打开 <https://github.com/new>，填仓库名 `my-project`，选 **Public**，**不要**勾选 "Add a README file"（会和本地冲突），点 `Create repository`。
页面会显示仓库地址，回到命令行：

```powershell
git remote add origin https://github.com/你的用户名/my-project.git
git branch -M main
git push -u origin main
```

刷新网页，文件就在了。

### 路线 B：装了 GitHub CLI，一条命令

```powershell
cd E:\MyPoints
git init
git add -A
git commit -m "第一次提交"
gh repo create my-project --public --source . --remote origin --push
```

`--source .` 用当前目录当仓库，`--public` 公开，`--push` 建完直接推。

---

## 三、四个概念，理解 Git 在干什么

| 概念 | 通俗解释 |
| --- | --- |
| 工作区 | 你平时编辑的那个文件夹 |
| 暂存区 | 「这次要打包的文件」的清单，`git add` 往里放 |
| 本地仓库 | `.git` 隐藏文件夹，存着所有历史快照，`git commit` 生成 |
| 远端仓库 | GitHub 上的那份，`git push` 上传、`git pull` 下载 |

一次上传的本质：**挑文件 → 打包 → 传到远端**，对应 `git add` → `git commit` → `git push`。

`.git` 文件夹是仓库的灵魂，删了历史就没了（文件还在，但全部变成「未跟踪」）。它不需要也不应该上传。

---

## 四、逐条解释那几条命令

### `git init`

把当前文件夹变成 Git 仓库，生成 `.git`。一个文件夹只需执行一次。

### `git add`

把文件放进暂存区。

```powershell
git add main.py              # 单个文件
git add src/ docs/           # 整个文件夹
git add -A                   # 所有新增、修改、删除
git add *.py                 # 按通配符
```

**先 `git status` 看一眼**，确认没有把 `.venv`、密码文件之类的加进去。

### `git commit`

把暂存区打包成一个快照，写一条说明。

```powershell
git commit -m "修复登录重试的 bug"
```

提交信息写清楚「做了什么」，别写 `update`、`1`。以后回滚、看历史全靠它。

### `git remote add origin ...`

告诉本地仓库「远端在哪」。`origin` 只是远端的默认代号，可以叫别的，但大家都用 origin。

```powershell
git remote add origin https://github.com/用户名/仓库名.git
git remote -v                # 查看已配置的远端
```

### `git branch -M main`

把默认分支从 `master` 改名成 `main`（GitHub 现在的默认叫法）。不改也能用，只是不太统一。

### `git push`

上传本地提交到远端。

```powershell
git push -u origin main      # 第一次：指定远端和分支，-u 记住对应关系
git push                     # 以后直接这样
```

---

## 五、用 GitHub CLI 一条命令搞定

`gh repo create` 把「网页建仓 + 加 remote + push」合并了：

```powershell
# 用当前目录当仓库，直接建好并推送
gh repo create my-project --public --source . --remote origin --push

# 建私有仓库
gh repo create my-project --private --source . --remote origin --push

# 指定描述
gh repo create my-project --public --source . --push `
  --description "一句话介绍你的项目"
```

其他常用：

```powershell
gh repo view --web           # 浏览器打开当前仓库
gh repo list                 # 列出你的仓库
gh repo delete 用户名/仓库名  # 删除仓库（不可恢复）
```

---

## 六、不想碰命令行：图形界面方案

### 方案 1：GitHub Desktop（对新手最友好）

1. 下载 <https://desktop.github.com/> 并登录
2. 菜单 `File` → `Add local repository`，选你的项目文件夹
   - 如果提示「这不是 Git 仓库」，点 `create a repository` 即可
3. 左侧会列出所有改动，勾选要提交的文件（默认全选）
4. 左下角填提交说明，点 `Commit to main`
5. 点上方 `Publish repository`，**取消勾选** "Keep this code private" 就是公开仓库

### 方案 2：PyCharm / IntelliJ

1. `VCS` → `Enable Version Control Integration` → 选 `Git`
2. `Git` → `GitHub` → `Share Project on GitHub`，登录后填仓库名即可建仓并推送
3. 日常：右上角 √ 是提交，↑ 是推送，↓ 是拉取

### 方案 3：网页直接拖拽（只适合少量小文件）

GitHub 仓库页 → `Add file` → `Upload files` → 把文件拖进去 → `Commit changes`。
不支持文件夹层级太多、超过 25 MB 的文件，**不适合整个项目**，应急用。

---

## 七、.gitignore：决定什么不该上传

在项目根目录新建 `.gitignore`，每行一个规则：

```gitignore
# Python 缓存
__pycache__/
*.py[cod]

# 虚拟环境（几百 MB，绝对不要传）
.venv/
venv/

# 编辑器配置
.idea/
.vscode/

# 运行产物
*.log
输出结果.xlsx

# 含密码的配置（如果不想公开）
# account.json
```

规则写好后，被忽略的文件不会出现在 `git status` 里，`git add -A` 也会跳过。

**已经误加进仓库的怎么办？**

```powershell
git rm --cached 文件名        # 只从仓库移除，本地文件保留
git rm -r --cached .venv      # 整个文件夹
git commit -m "移除误提交的文件"
```

注意：这只是在最新提交里删掉，**历史里还在**，clone 时依然会下载。彻底清理见下一节。

---

## 八、八个最常见的坑

### 1. 仓库名不能用中文

GitHub 只接受英文字母、数字、`.`、`-`、`_`。用中文建仓会被截断（比如变成 `-`）。中文名写在仓库的 **Description** 里。

### 2. 大文件：50 MB 警告，100 MB 拒绝

而且**一旦提交进历史，之后删掉也还占体积**。25 MB 的安装包这类东西，正确做法是：

- 放到仓库的 **Releases**（`Releases` → `Create a new release` → 上传附件）
- 或者干脆只放下载链接

已经进了历史的，删除方式：

```powershell
git rm --cached 大文件.exe
git commit --amend -m "重新提交"
git push --force
```

（`--amend` 只能改最后一次提交；更早的历史要用 `git filter-repo` 或 BFG Repo-Cleaner。）

### 3. `git status` 里中文文件名变成 `"\344\275\277..."`

```powershell
git config --global core.quotepath false
```

### 4. 建仓时勾了 "Add a README"，push 被拒

远端已有提交，本地历史对不上。两种解法：

```powershell
git pull origin main --allow-unrelated-histories   # 合并
# 或者
git push --force                                   # 直接覆盖（新仓库没内容时安全）
```

### 5. push 一直要求输密码

GitHub 早就不能用账号密码了，要输的是 **Personal Access Token**。或者装 `gh` 用 `gh auth login` 配好凭据。

### 6. 换行符警告 `LF will be replaced by CRLF`

Windows 与 Linux 换行符不同，无害。想消掉：

```powershell
git config --global core.autocrlf true
```

### 7. 密码/密钥被提交了

立刻改密码、吊销密钥，然后从历史里清除（`git filter-repo`）。**光删文件不够**，历史里永远能翻出来。

### 8. `git push` 报 `rejected - non-fast-forward`

远端有你本地没有的提交（比如你在网页上改过文件）：

```powershell
git pull --rebase origin main
git push
```

---

## 九、日常循环与命令速查

上传新项目是「一次性动作」，之后每天只是这三行：

```powershell
git add -A
git commit -m "这次改了什么"
git push
```

### 命令速查表

| 目的 | 命令 |
| --- | --- |
| 看当前状态 | `git status` |
| 看具体改了什么 | `git diff` |
| 看提交历史 | `git log --oneline` |
| 撤销未提交的修改 | `git restore 文件名` |
| 取消暂存 | `git restore --staged 文件名` |
| 撤销上一次提交（保留改动） | `git reset --soft HEAD~1` |
| 查看远端地址 | `git remote -v` |
| 修改远端地址 | `git remote set-url origin 新地址` |
| 拉取远端更新 | `git pull` |
| 克隆别人的仓库 | `git clone https://github.com/用户/仓库.git` |
| 临时存一下改动 | `git stash` / `git stash pop` |

---

## 十、上传前检查清单

- [ ] `git status` 里没有 `.venv`、`.idea`、`__pycache__`
- [ ] 没有密码、token、身份证号、学号等敏感信息
- [ ] 没有几十 MB 的安装包/视频/数据集
- [ ] 有 `.gitignore`
- [ ] 有 `README.md`：项目是干什么的、怎么装、怎么跑
- [ ] 有 `requirements.txt` / `package.json` 等依赖清单
- [ ] 提交信息写得像人话
- [ ] 仓库是 Public 还是 Private 想清楚了

---

## 附：本教程对应的真实案例

`E:\MyPoints` 里的「第二课堂学时自动查询工具」就是这么传上去的，最终命令：

```powershell
cd E:\MyPoints
git init
git add main.py account.json requirements.txt 使用说明.txt .gitignore README.md
git commit -m "第二课堂学时自动查询工具"
gh repo create second-classroom-hours-query --public --source . --remote origin --push
```

中途踩到的坑：仓库名带中文被截成 `-`、25 MB 的 `VC_redist.x64.exe` 进了首次提交后又得改写历史移除。这两条都在第八节里。
