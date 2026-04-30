# BistroCode 更新与部署流程

这份文档是按当前实际环境整理的：

- 本地代码目录：`~/ideaprojects/BistroCode`
- 你的 GitHub fork：`https://github.com/Atoooooooon/sub2api`
- 官方上游仓库：`https://github.com/Wei-Shaw/sub2api`
- 服务器代码目录：`/opt/BistroCode`
- 服务器部署方式：`docker compose -f deploy/docker-compose.local.yml`

## 一次性准备

只需要做一次，当前仓库已经配置好：

```bash
cd ~/ideaprojects/BistroCode
git remote -v
```

你应该能看到：

- `origin` 指向你的 fork
- `upstream` 指向官方仓库 `Wei-Shaw/sub2api`

## 推荐更新流程

推荐思路：

1. 先把官方最新代码同步到本地
2. 处理你自己的品牌/定制改动冲突
3. 推送到你自己的 fork
4. 在服务器上拉取你 fork 的最新代码并重建容器

---

## 第 1 步：本地同步官方最新代码

先进入项目目录：

```bash
cd ~/ideaprojects/BistroCode
```

### 1.1 先检查本地是否有未提交改动

```bash
git status
```

如果有改动，先提交一版。不要在一堆未提交修改的状态下直接合并官方更新。

```bash
git add -A
git commit -m "chore: save local BistroCode changes before upstream sync"
```

### 1.2 拉取官方最新代码

```bash
git fetch upstream
```

查看官方最新提交：

```bash
git log --oneline --decorate --graph -n 10 upstream/main
```

### 1.3 合并官方 main 到你本地 main

```bash
git checkout main
git merge upstream/main
```

说明：

- 推荐先用 `merge`，因为它最稳，也更适合已有线上部署的分支
- 如果没有冲突，Git 会自动完成合并
- 如果有冲突，按提示改完文件后继续：

```bash
git add -A
git commit
```

### 1.4 本地自检

至少建议做下面几项：

```bash
git status
docker compose -f deploy/docker-compose.local.yml config >/dev/null
```

如果你后面要在本地继续开发，再做你自己的测试命令。

### 1.5 推送到你自己的 fork

```bash
git push origin main
```

到这里，你的 GitHub fork 就已经带着“官方最新代码 + 你的 BistroCode 定制”了。

---

## 第 2 步：服务器更新部署

登录服务器：

```bash
ssh root@43.166.202.103
```

进入项目目录：

```bash
cd /opt/BistroCode
```

### 2.1 确认服务器工作区干净

```bash
git status
```

正常情况下应该是干净的。如果这里出现未提交修改，先停下来确认这些改动是不是你有意留在服务器上的。

### 2.2 拉取你 fork 的最新代码

```bash
git pull --ff-only origin main
```

说明：

- 用 `--ff-only` 是为了避免服务器上意外产生合并提交
- 如果这一步失败，先看 `git status` 和 `git log --oneline -n 5`

### 2.3 重新构建并启动

```bash
cd /opt/BistroCode/deploy
docker compose -f docker-compose.local.yml build bistrocode
docker compose -f docker-compose.local.yml up -d
```

如果你想省一点命令，也可以直接：

```bash
cd /opt/BistroCode/deploy
docker compose -f docker-compose.local.yml up -d --build
```

### 2.4 检查容器状态

```bash
docker compose -f docker-compose.local.yml ps
docker compose -f docker-compose.local.yml logs --tail=100 bistrocode
```

### 2.5 检查站点是否正常

```bash
curl -I http://127.0.0.1:8080
curl -I http://127.0.0.1
curl -kI https://bistrocode.online
```

---

## 常用命令速查

### 本地同步官方

```bash
cd ~/ideaprojects/BistroCode
git status
git add -A
git commit -m "chore: save local changes"
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

### 服务器发布

```bash
ssh root@43.166.202.103
cd /opt/BistroCode
git pull --ff-only origin main
cd deploy
docker compose -f docker-compose.local.yml up -d --build
docker compose -f docker-compose.local.yml ps
docker compose -f docker-compose.local.yml logs --tail=100 bistrocode
```

---

## 冲突处理建议

你现在这套 BistroCode 定制主要集中在：

- 前端品牌文案
- 默认站点名
- 默认管理员邮箱
- Docker 部署命名

所以以后最容易冲突的地方通常是：

- `frontend/`
- `backend/internal/setup/`
- `backend/internal/service/`
- `deploy/.env.example`
- `deploy/docker-compose.local.yml`

如果官方更新正好改到这些文件，处理方式是：

1. 先保留官方功能改动
2. 再把你的 `BistroCode` 品牌定制补回去
3. 合并后本地先看 `git diff`
4. 再推送、再上服务器

---

## 一个很重要的习惯

以后每次准备同步官方代码前，先做一次提交。

即使只是临时提交，也比带着一堆未提交改动去 `merge upstream/main` 安全得多。这样真出冲突时，你也能清楚分辨：

- 哪些是官方新增
- 哪些是你自己的 BistroCode 改动

