# Git 版本管理学习笔记：从三区模型到团队协作

## 一、概念引入：什么是版本控制

版本控制（Version Control）就是**给代码的每一次变更拍快照**，记录"谁、在什么时候、改了哪些文件、为什么改"，让我们能随时回看历史、对比差异、回到任意旧版本、多人并行开发而互不踩踏。

在没有版本控制的年代，团队协作的典型场景是灾难片：A 在某目录拷一份叫 `project_最终版.py`，B 又拷一份 `project_最终版2.py`，两周后全部分不清哪个是最新的。版本控制彻底解决了这个问题。

版本控制分两大流派：

| 模式 | 代表 | 特点 |
| ---- | ---- | ---- |
| 集中式（CVCS） | SVN | 只有一个中央仓库，所有人连它提交；离线不能提交，中央坏了全歇菜 |
| 分布式（DVCS） | **Git** | 每个人本地都有一份**完整历史**的克隆；离线可提交，中央没了也能自愈 |

条目：分布式 Git 之所以能成为绝对主流，核心在于"本地即仓库"——你的 `git commit` 不依赖任何服务器，只有 `push` 才需要联网。

## 二、为什么 Git 是 AI/数据工程玩家的必备技能

1. **这是整个开源生态的默认协作协议**。PyTorch、LangChain、Milvus、所有你 clone 的模型代码都在 GitHub/GitLab 上，贡献代码的唯一方式是 Git。
2. **实验管理的基本功**。训练代码、prompt 模板、数据清洗脚本同样需要版本化回溯——`git diff` 能让你看清"这次调参到底改了什么"。
3. **这不是"以后再用"的知识，而是这周就会用的知识**。你马上就要把学习笔记 push 到远程仓库（比如本库就在 GitHub 上），Git 是日常阅读你写的笔记的仓库的基础设施。
4. **协作与安全红线**。多人改同一份代码时，Git 的冲突解决能力让"并行开发"变得可控；配合 `.gitignore` 还能防止把 API Key、权重文件误传上网。

## 三、核心原理：Git 的三区模型

这是理解 Git 一切行为的钥匙。一次提交背后是三个区域协同：

```
工作区（Working Directory）  --add-->  暂存区（Index/Staging）  --commit-->  版本库（Repository/HEAD）
  你看到的文件                 git add 后                git commit 后
  改变 → git status 显示"modified"   改变 → 显示"Changes to be committed"   改变 → 进入历史快照
```

- **工作区**：你磁盘上看到的、正在编辑的文件。
- **暂存区**：一个中间缓冲区，`git add` 把改动从工作区拣选进来；它让你可以"把多个文件的改动分批提交成多个 commit"。
- **版本库**：`git commit` 后改动固化成一个历史快照，用 commit 哈希（如 `3f8d2a1`）唯一标识。

`git status` 永远是理解当前状态的第一命令——它精确告诉你当前的改动位于哪个区、下一步该 `add` 还是 `commit`。

## 四、关键知识点速查表

| 维度 | 命令 | 一句话记忆 |
| ---- | ---- | ---------- |
| 初始化 | `git init`、`git clone <url>` | 新建仓库 / 克隆远程 |
| 提交三连 | `git add .`、`git commit -m "msg"`、`git push` | 拣选→固化→上传 |
| 状态 | `git status`、`git log --oneline`、`git diff` | 看现状、看历史、看差异 |
| 分支 | `git branch -a`、`git checkout -b feat`、`git merge feat` | 建分支、切分支、合并 |
| 变基 | `git rebase main`、`git rebase -i HEAD~3` | 线性化历史 / 交互式整理提交 |
| 撤销 | `git reset --hard HEAD~1`、`git revert <hash>`、`git checkout -- file` | 三种不同粒度的"后悔药" |
| 远程 | `git remote -v`、`git pull`、`git fetch`、`git push -u origin main` | 远端别名/同步 |
| 协作 | Fork + PR、`.gitignore` | 见第八节 GitHub 流程 |
| 标签 | `git tag v1.0.0`、`git push --tags` | 给里程碑打点 |

## 五、实践示例一：日常提交流程（写在写字板上的流程）

```bash
##### 0) 第一次提交前的全局身份（提交记录里要署名）
git config --global user.name "qiaokj"
git config --global user.email "you@example.com"

##### 1) 日常开发循环（一天重复 N 次）
git status                  # 先看当前状态
git diff                    # 看工作区的具体改动，确认没改错
git add file1.py file2.py   # 只 add 本次想提交的文件（代替 git add . 更精准）
git commit -m "feat: 新增向量检索接口"   # 写好说明性提交信息
git push                    # 推到远程

##### 2) 提交信息规范（commit message 三要素）
# 推荐格式：<type>: <subject>
# type: feat(新功能) / fix(修bug) / docs(文档) / refactor(重构) / chore(杂务)
git commit -m "fix: 修复 connect 超时导致的内存泄漏"

##### 3) 反悔/纠正类
git add . && git commit --amend -m "新信息"   # 修改最近一次提交的信息（仅限未 push）
git reset --soft HEAD~1        # 撤销 commit，改动保留在暂存区
git reset --hard HEAD~1        # 撤销 commit 且丢弃改动（危险）
git revert <commit哈希>        # 生成一个"反向提交"，历史不被改写
```

## 五.二、完整实操示例：从零到远程

```bash
# 1. 在 GitHub 网页上建空仓库后（或直接本地新建）
mkdir my-rag && cd my-rag && git init

# 2. 配置远程仓库（两种方式）
git remote add origin git@github.com:qiaokj/my-rag.git   # SSH 方式
git remote -v                                             # 确认已指向

# 3. 提交并推送
echo "hello rag" > README.md
git add README.md && git commit -m "init: 初始化项目"
git push -u origin main      # -u 建立跟踪关系，之后只需 git push

# 4. 认领任务的新分支流程
git checkout -b feat/vector   # 从当前分支拉新分支并切过去
# ... 开发若干小时 ...
git status && git diff
git add . && git commit -m "feat: 接入 milvus 客户端"
git push -u origin feat/vector   # 推到远端这些分支
# 在 GitHub 上发 PR 合并到 main（见第六节）
```

## 六、分支模型：branch / checkout / merge / rebase

### 6.1 分支是"移动的指针"

`git branch` 本质上是一个指向某个 commit 的可移动指针。`git checkout -b feat/x` 就是在当前 commit 上新建一个指针并让 HEAD 指向它。分支的代价极低（一个 40 字节的哈希），所以 Git 鼓励"一功能一分支"。

### 6.2 merge 合并

```bash
git checkout main
git merge feat/vector       # 把 feat/vector 的历史并进 main
git branch -d feat/vector   # 合并完成后删除本地特性分支
```

`git merge` 会**保留双方的历史**（产生 merge commit），适合长期分支与团队主干合并；问题是主干的历史会变成"分岔再合成"的网状。

### 6.3 rebase 变基：合并 vs 变基的对比

```bash
git checkout feat/vector
git rebase main   # 把这分支的提交一个个"搬到" main 的最新提交之后
```

`git rebase` 是**改写提交顺序**，把当前分支的历史"重放"到另一分支顶端，结果是完全线性的历史。对比：

| 维度 | merge | rebase |
| ---- | ----- | ------ |
| 历史形状 | 保留分岔，产生 merge commit | 线性，无额外合并提交 |
| 是否改写历史 | 否 | 是（commit 哈希会变） |
| 适用场景 | 合入主干、多人协作主干 | 个人特性分支追平主干、整理提交 |
| 黄金法则 | — | **只 rebase 自己还没 push 的分支，绝不 rebase 远程已共享的分支** |

简化决策：**"我的分支追新就 rebase，把分支并入主干就 merge。"**

## 七、远程协作：remote / push / pull / fetch / 冲突

```bash
git fetch origin main      # 只下载远端最新状态到本地"远程跟踪分支"，不改动工作区
git pull origin main       # = fetch + merge，一步到位
git push origin main       # 把本地提交推到远端
```

`fetch` 与 `pull` 的区别是笔试高频坑：**fetch 只更新仓库里的远程镜像，pull 会真把远端内容合并进你的当前分支**。想"先看看远端有什么再决定"，就 fetch；确定要同步，就 pull。

### 7.1 解决合并冲突（人人都会遇到）

两人都改了同一文件的同一处，push 必然冲突：

```bash
git pull          # 报错：CONFLICT (content): Merge conflict in app.py
vim app.py
```

冲突文件里 Git 会插入标记：

```
<<<<<<< HEAD
你本地的改动
=======
远端/他人在另一分支的改动
>>>>>>> origin/main
```

把 `<<<<<<<` 到 `=======`（或到 `>>>>>>>`）取舍后删掉标记行，然后：

```bash
git add app.py && git commit -m "merge: 解决 app.py 冲突"
```

## 七.二、撤销与回滚：reset / checkout / revert 对比表

| 命令 | 影响范围 | 是否改写历史 | 推荐场景 |
| ---- | -------- | ------------ | -------- |
| `git checkout -- file` | 工作区单个文件 | 否 | 工作区改乱了，想扔回暂存区/版本库版本 |
| `git reset --soft HEAD~1` | 仅撤销 commit，改动留在暂存区 | 是 | 提交信息写错了，想撤回来重新 commit |
| `git reset --mixed HEAD~1` | 撤销 commit + add，改动回到工作区 | 是 | commit 加错了文件，想重加 |
| `git reset --hard HEAD~1` | 撤销 commit 且**丢弃**全部改动 | 是 | 明确要扔掉脏改动（小心！） |
| `git revert <hash>` | 生成一个"撤销该提交"的新提交 | 否 | 对**已 push** 的提交回滚，最安全 |
| `git reset --hard origin/main` | 把本地彻底对齐远端 | 是 | 本地搞乱了想以远端为准 |

**黄金法则**：已推到远端、别人可能已拉取的历史，**只用 `revert`**（新增反向提交，不动历史）；只有自己还没 push 的提交，才放心大胆用 `reset`。

## 八、GitHub 协作流程：Fork + PR + issue + gitignore

### 8.1 给开源项目贡献的完整链路

1. 在 GitHub 上点击目标仓库的 **Fork**（在你自己账号下生成一份拷贝）。
2. `git clone git@github.com:<你的账号>/<repo>.git`
3. 添加上游：`git remote add upstream <原项目URL>`，定期 `git fetch upstream && git merge upstream/main` 保持同步。
4. 开自己的特性分支开发（**绝不要在 main 上直接改**）。
5. `git push` 后回到 GitHub 点 **Pull Request**，写明"修了什么、为什么、如何测试"。
6. 维护者 review 后合入。若 review 中要求修改，继续在同一分支 push，PR 会自动更新。

**issue** 是项目的问题/需求工单，贡献前先在 issue 里打声招呼，避免重复劳动。

### 8.2 .gitignore：防止武器入库

`.gitignore` 声明不纳入版本控制的路径。AI 项目必须忽略的是：

```gitignore
# Python
__pycache__/
*.pyc
venv/ .venv/
*.pth

# 权重与大文件（别把几十 G 的模型硬塞进 git！用 Git LFS）
*.pt *.safetensors *.bin *.onnx

# 密钥与环境变量（重中之重！）
.env
*.key *.pem
config_local.yaml

# 日志与临时文件
*.log
nohup.out
.DS_Store
.idea/ .vscode/
```

**血的教训**：`.env` 里通常放着 API Key，一旦被 push 到公开仓库，几小时内就会被爬虫扫走盗用。提交前养成 `git status` 检查习惯，必要时用 `git rm --cached .env` 从暂存区移除。

## 九、常见坑与小结

1. **`git add .` 无脑提交**会带入 `.env`、模型文件、`__pycache__`。改成 `git add <具体文件>`，或用 `git add -p` 分批拣选。
2. **push 前先 pull / pull 前先 commit**。本地有未提交改动时 pull 可能直接把冲突糊到你脸上，先 commit 再 pull 更可控。
3. **不要 `git push -f`**（force push）到共享分支，这会覆盖别人历史，是社区社交性死亡操作。
4. **Windows 换行符 CRLF** 会造成整文件假 diff：用 `git config --global core.autocrlf input`（Linux 风格）或建立 `.gitattributes` 统一 `* text=auto`。
5. **克隆大模型仓库超慢是正常的**，那往往是几百 GB 的 LFS 或子模块：`git clone --recurse-submodules` 用法要看仓库说明，必要时只 `--depth 1` 浅克隆。
6. **别名值得配**：`git config --global alias.lg "log --oneline --graph --all"`，之后 `git lg` 一览全貌。

**小结**：Git 的全部心智模型可以浓缩成三句话——**改码进工作区，add 进暂存区，commit 进历史**；**分支就是指针，merge 保历史、rebase 理线性、push 前只对自己 history 任性**；**已落地远程的回滚永远用 revert**。配合"一功能一分支 + 规范提交信息 + Fork/PR"的工作流，就足以支撑从个人项目到开源协作的所有需求。

## 十、参考资料

- [Pro Git（官方电子书，中文版免费，必读）](https://git-scm.com/book/zh/v2)
- [Git Reference（官方权威参考）](https://git-scm.com/docs)
- [GitHub 官方文档：Fork a repo / About pull requests](https://docs.github.com/zh/pull-requests)
- [Learn Git Branching（可视化交互教程，强烈推荐通关）](https://learngitbranching.js.org/?locale=zh_CN)
- 本知识库配套：`00-准备与基础/03-Linux基础/Linux基础.md`（`.bashrc`、SSH 免密等内容与本笔记互补）