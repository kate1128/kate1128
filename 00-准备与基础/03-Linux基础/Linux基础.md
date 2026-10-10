# Linux 基础笔记：从零命令行到 AI 服务器实操

## 一、什么是 Linux：一句话概念引入

Linux 是一个自由开源的、类 Unix 的操作系统内核（Kernel），最早由林纳斯·托瓦兹（Linus Torvalds）在 1991 年发布。我们平时说的"用 Linux"，其实用的是"Linux 发行版"（Distribution）——也就是在 Linux 内核之上，打包了 GNU 工具链、Shell、包管理器、图形界面（可选）和应用软件的一个完整可用的操作系统。

常见发行版大致分两大家族：

| 发行版 | 所属家族 | 包管理器 | 适用场景 |
| ------ | -------- | -------- | -------- |
| Ubuntu / Debian | Debian | apt / dpkg | 最普及的开发环境，教程多，AI 框架编译友好 |
| CentOS / Rocky / Alma | RedHat | yum / dnf / rpm | 服务器、企业生产，稳定优先 |
| openSUSE / Arch | 各自独立 | zypper / pacman | 小众 / 极致自订（如 Arch） |
| 企业云环境 | — | 多为 Debian/RedHat 系 | 云服务器（ECS 等）绝大多数是这两系 |

对我们做 AI/大模型的人来说，还有个隐形标准：**绝大多数学术开源项目、深度学习框架（PyTorch、TensorFlow）、推理引擎（vLLM、Triton）、向量数据库（Milvus）的官方文档和部署教程，均以 Ubuntu / Debian 作为第一优先说明对象**。所以本笔记的命令以 Ubuntu 系为主，同时标注 RedHat 系的对应命令。

## 二、为什么 AI/服务端开发"必须"会 Linux

这不是口号，而是现实约束，理由非常具体：

1. **训练与推理环境就是 Linux**。CUDA、NVIDIA 驱动程序、GPU 容器（NGC Container）基本都是针对 Linux 提供的；H100/A100 集群跑的就是 Linux。Windows 上做深度学习只能靠 WSL2 或 Docker Desktop，本质还是借 Linux 内核。
2. **生产服务器几乎全是 Linux**。你微调出的模型、部署的 RAG 服务、Milvus 向量库、MinIO 对象存储，最终都跑在 Linux 机器上。不会 Linux，等于不会"线上那台机器"。
3. **一切自动化依赖 Shell**。训练脚本、数据预处理 pipeline、定时任务（cron）、自动重启、日志清理，全是 shell/python 脚本 + Linux 命令的组合。
4. **远程开发方式就是 SSH**。4 张 8 卡服务器放在机房或云上，你本地只有一台笔记本，唯一的交互入口就是 `ssh` + 终端。

一句话：Linux 是真刀真枪干活的地方，不是可选项。

## 三、核心原理：文件系统目录结构、一切皆文件

### 3.1 目录结构（FHS 标准）

Linux 采用单一根目录 `/` 的分层结构，所有绝对路径都从根开始，没有 Windows 那样的 C:/D:/ 盘符。

| 目录 | 作用 | 我们的关注点 |
| ---- | ---- | ------------ |
| `/` | 根目录，所有路径的起点 | — |
| `/home` | 普通用户的家目录，如 `/home/kate` | 个人文件、`~/.bashrc` 在这里 |
| `/root` | root 用户（超级管理员）的家目录 | 别把东西乱塞进这里 |
| `/etc` | 系统与应用全局配置文件 | `requirements` 类比：容器/服务配置都在这 |
| `/var` | 可变数据，尤其日志 `/var/log` | 排查线上问题最先看的地方 |
| `/tmp` | 临时文件，重启可能被清空 | 大文件别放这，丢数据别哭 |
| `/usr` | 系统安装的程序与库 | 类似 Windows 的 Program Files |
| `/opt` | 第三方大型软件（Anaconda 常装在这） | 装 CUDA、Anaconda 的常见位置 |
| `/dev` | 设备文件 | GPU 就会被挂载为 `/dev/nvidia0` 等 |
| `/proc` | 内核与进程的虚拟文件系统，实时反映运行状态 | 排查进程、内存的"实时仪表盘" |

### 3.2 一切皆文件的哲学

Linux 的设计哲学是"一切皆文件"：普通文件、目录、磁盘、显卡、网络 socket 都被抽象成文件描述符，用统一的 `open/read/write` 接口访问。这也是为什么你可以用 `cat /proc/meminfo` 看内存、用 `cat /etc/os-release` 看系统信息——它们本质都是"读文件"。

### 3.3 用户权限模型

Linux 是多用户系统，权限靠三组三元组（x）决定：**属主（user）、属组（group）、其他（other）**，每组三个权限位 **读 r / 写 w / 执行 x**。

权限数字表示法是笔试面试高频考点，务必写死到肌肉记忆：

```
r = 4（二进制 100）
w = 2（二进制 010）
x = 1（二进制 001）
```

把三类角色的权限分别求和拼起来即可，比如 `chmod 754 file` 表示：属主=7（rwx），属组=5（r-x），其他=4（r--）。看到 `rwxr-xr--` 也要能立刻说出等效数字 `754`。

## 四、关键知识点速查表

| 类别 | 命令示例 | 一句话记忆 |
| ---- | -------- | ---------- |
| 文件目录 | `cd /tmp`、`ls -lah`、`mkdir -p a/b`、`cp -r`、`mv`、`rm -rf` | 增删改查四件套 |
| 查找 | `find /app -name "*.log"`、`grep -r "ERROR" .` | 按名搜文件 / 按内容搜文本 |
| 文本处理 | `cat`、`head -n 20`、`tail -f`、`grep`、`sed 's/a/b/g'`、`awk '{print $1}'` | 看、切、淘、换、取列 |
| 权限 | `chmod 754 file`、`chown kate:kate file`、`sudo` | 数字权限 + 属主/组 |
| 进程 | `ps aux`、`top`、`htop`、`kill -9 PID` | 看进程、看资源、杀进程 |
| 网络 | `curl -s`、`wget`、`ssh user@host`、`scp file host:/path` | 抓接口、下载、远程、传文件 |
| 包管理 | `apt install/uninstall/update`、`yum install` | Debian/RedHat 两大家 |
| GPU | `nvidia-smi`、`nvidia-smi -L` | 看显存占用、看卡数量 |
| Shell | `|`（管道）、`>`（重定向）、`$PATH`、`source ~/.bashrc` | 组合、落盘、环境变量 |

## 五、实践示例一：日常高频命令组合

### 5.1 全程查一中台大数据项目最常用的"生产三连"

```bash
##### 1. 看磁盘与内存，判断是不是"不知不觉满了"
df -h                          # 磁盘占用，-h 人类可读
free -h                        # 内存与 swap

##### 2. 看 GPU，训练前必查
nvidia-smi                     # 显存、卡利用率、温度
nvidia-smi -l 5                # 每 5 秒刷新一次，观察是否吃满

##### 3. 看日志，tail -f 是运维最常用的命令
tail -f /var/log/web.log       # 实时滚动查看日志
tail -n 100 /var/log/web.log   # 看最后 100 行
grep -i "error\|exception" web.log | head -n 20   # 找报错
```

### 5.2 文本处理三板斧：grep / sed / awk

```bash
# 在日志里找 ERROR 行，并统计出现次数
grep -c "ERROR" app.log
grep -i "timeout" app.log | wc -l

# sed 替换：把旧域名替换成新域名（-i 写回原文件）
sed -i 's/api.old.cn/api.new.cn/g' config.yaml

# awk 取列：ps 输出里取第 1 列 PID，批量 kill
ps aux | awk '{print $2}' > pids.txt
# 典型用途：按第 3 列（RSS 内存）排序找最占内存的进程
ps aux --sort=-%mem | awk 'NR<=6{print $1,$3,$4,$11}'
```

### 5.3 进程管理与后台运行

```bash
ps -ef | grep python           # 看 python 进程
top                           # 实时看 CPU/内存占用，q 退出
kill -9 12345                 # 强杀进程（-9 别乱用，先试普通 kill 12345）

# 后台常驻：训练脚本挂上 nohup，重定向日志，断网也不挂
nohup python train.py > train.log 2>&1 &
echo $!                       # 打印刚挂到后台的 PID，记下来以便日后 kill

# 更推荐 tmux 多会话管理，断线可恢复
tmux new -s train             # 新建会话
tmux ls                       # 列出会话
tmux attach -t train          # 重新接入
# 退出但不杀会话：先 Ctrl+b 再按 d
```

为什么训练脚本要用 `nohup ... &`？因为 SSH 断开时终端会发 SIGHUP 信号把前台进程杀掉；`nohup`（no hang up）让进程忽略该信号，`2>&1` 把标准错误并到标准输出一起写进日志文件，`&` 让命令转后台。三者缺一不可。

### 5.4 SSH 与远程传输

```bash
# 免密登录推荐：把本地公钥提前放到目标机的 ~/.ssh/authorized_keys
ssh-keygen -t ed25519         # 生成密钥（一路回车）
ssh-copy-id kate@10.0.5.73    # 一键把公钥部署到远程（RedHat 系可能没有此命令，手动追加即可）
ssh kate@10.0.5.73            # 之后免密登录

# 传文件
scp train.py kate@10.0.5.73:/home/kate/    # 本地上传
scp kate@10.0.5.73:/opt/model.bin .        # 远程下载
# 传大目录建议用 rsync，断点续传且可增量
rsync -avz ./models/ kate@10.0.5.73:/home/kate/models/
```

### 5.5 权限实操

```bash
ls -l                     # 第一列形如 -rw-r--r--：第1位文件类型，后9位是三组权限
chmod +x run.sh           # 给脚本加执行权限
chmod 600 id_rsa          # 私钥必须 600，否则 SSH 直接拒绝（权限过宽=安全警告）
chown kate:kate data/     # 改属主和属组，部署后文件归属混乱时最常用
chmod -R 755 deploy/      # -R 递归改整个目录

# 提权：sudo 以 root 身份执行
sudo apt update
sudo -i                   # 进入 root shell（生产环境谨慎）
sudo useradd -m deploy    # 常见：建专门的部署账号，不用 root 裸奔
```

## 六、工具 / 选型对比

**包管理：apt（Debian 系）vs yum/dnf（RedHat 系）**

| 能力 | Ubuntu apt | CentOS/Rocky yum |
| ---- | ---------- | ---------------- |
| 安装 | `apt install vim` | `yum install -y vim` |
| 卸载 | `apt remove vim` | `yum remove vim` |
| 更新索引 | `apt update` | `yum makecache` |
| 升级 | `apt upgrade` | `yum update` |
| 搜索 | `apt search vim` | `yum search vim` |

**Shell：bash vs zsh**——生产环境几乎都用 bash（系统默认、脚本兼容性最好）；本地开发用 zsh + oh-my-zsh 更舒服，但写脚本仍建议用 bash 语法。

**vim 快速上手**：`vim` 分插入模式（按 `i`）与命令模式（按 `Esc`）。只需记住三个动作即可云上救命：`i` 进入编辑、`Esc` 回命令模式、`:wq` 保存退出、`:q!` 不保存强退。实在不想记就装 `nano`，`Ctrl+O` 保存、`Ctrl+X` 退出。

## 七、常见坑与小结

1. **不要上来就 `rm -rf`**。`rm -rf /`、`rm -rf $var`（变量为空时）能瞬间毁掉整个系统/数据，用前先 `ls` 确认，养成 `rm -i` 或先 `mv 到/tmp` 的习惯。
2. **权限数字第一位别漏**。`chmod 777` 等于给全世界读写执行，安全红线；部署用 755（目录）与 644（文件），密钥/凭据用 600。
3. **注意 `tail -f` 与 `grep` 的缓冲**。`tail -f x.log | grep --line-buffered "ERROR"` 要加 `--line-buffered`，否则 grep 默认块缓冲会导致结果迟迟不出现。
4. **一切皆文件≠一切都能 `rm`**。`/proc`、`/dev` 下的虚拟文件删了会自动重建，但别去动它们。
5. **Windows 换行符坑**：在 Windows 编辑过的 `.sh` 脚本上 Linux 跑会报 `\r` 错误，用 `sed -i 's/\r$//' run.sh` 清理。
6. **PATH 环境变量**：`export PATH=$PATH:/opt/cuda/bin` 只在当前会话生效，要永久得写进 `~/.bashrc` 再 `source ~/.bashrc`。`echo $PATH` 能看到全路径，`which python` 能查解释器到底用的是哪个。

**小结**：Linux 本质是"把一切抽象成文件 + 把一切操作变成命令"。AI 工程师日常落到四类技能：**查（磁盘/GPU/日志/进程）→ 找（find/grep）→ 改（sed/vim/权限）→ 跑（后台/nohup/tmux）**。把上面的命令亲手敲一遍，配上 `curl`/`scp` 就能独立完成"上传代码 → 看日志 → 排查问题 → 重启服务"的闭环。这是后面学 Git、Docker、以及所有线上部署笔记的共同地基。

## 八、参考资料

- [Linux 运维入门教程（Reboot 实验室）](https://www.runoob.com/linux/linux-tutorial.html)
- [The Linux Command Line（免费在线书，强烈推荐精读前半本）](https://linuxcommand.org/tlcl.php)
- [GNU Coreutils 手册（ls/cp/mv 等的权威文档）](https://www.gnu.org/software/coreutils/manual/)
- [tmux Cheat Sheet](https://tmuxcheatsheet.com/)
- 本知识库配套笔记：`03-向量数据库与Embedding/04-Milvus向量库/milvus用docker-compose部署.md`、`00-准备与基础/05-Docker容器化/Docker容器化.md`