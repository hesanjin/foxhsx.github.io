---
title: 52.4k star 的Multica:集多 Agent 和看板于一体的 AI 平台
date: 2026-11-02
tags:
  - AI
description: VibeKanban 停止维护后，我找到了 52.4k star 的 Multica：一个集多 Agent 协作和任务看板于一体的 AI 平台。本文分享 Docker Compose 自部署步骤、智能体创建，以及多 Agent 组队开发 3D 划船小游戏的完整体验，需要多 Agent 协作和 AI 任务看板的小伙伴可以去试试看了～
---

Hi\~大家好，我是三金。

我最近在使用 VibeKankan 的时候发现它的版本已经很久不更新了！打开它的 Github 一看，竟然要停止维护了，最新版本还是我现在使用的 0.1.44，已经半年没更新过版本了！

最近一次的 commit 还是3 周以前，这就很难受，好担心它后面都没人维护。也想过直接拉源码改，但最终还是想拿来即用。故而又找到了另外一个平台——Multica。

### 能做什么

简单来说，这是一个可自定义多 Agent 协作的平台。

官方的定位 Multica 是一个人类和 AI Agent 共同工作的团队工作区，我们可以：

* 自定义各种 Agent；
* 人做中枢或者指定 AI Agent 做中枢，给其他 AI 同事派活；
* 同一个团队里，上下文共享，工作里的讨论、记录以及产出都会留在工作区；
* 如果已经有成熟的 AI Native，我们甚至可以将多个 Agent 组成小队，然后实现任务下发后的自动化开发；
* 根据会话和 memory 自动沉淀团队经验，将可重复的做法收敛成 skill，然后给多个智能体复用。

以下是「工作」菜单下的任务、项目以及自动化的截图：

![](assets/siT8BFR03dw9KIeoXCQMGkMdQ8GTnzpsGllOqILfB8M=.webp)

### 使用

使用官方 Online 的话有两种模式：Web（multica.ai）以及桌面客户端 Multica Desktop，支持邮箱注册登录或者 Google 一键登录。

客户端的优势是登录后自动连接当前电脑，而 Web 端则需要通过命令行进行连接。

![](assets/5k-dg23PyY4gGUspxkKShvPBqF_sK6brEGAue1Fb_0U=.webp)

三金没有选择官方 Online 的方式来玩儿，而是通过自部署 + 客户端的方式启动。

### 自部署以及托管

自托管的 Multica 服务，主要运行 Web、API 和 PostgreSQL，我们可以通过 Docker Compose 快速启动。

另外，电脑上还需要：Git、Mack、curl 和 OpenSSL。这些前置条件准备好之后，我们就可以进行自部署了。

```shellscript
git clone --depth 1 https://github.com/multica-ai/multica.git
cd multica
make selfhost
```

> **执行 make selfhost 的作用**
> 会从 `.env.example` 创建 `.env`，然后随机生成 `JWT_SECRET`、PostgreSQL 密码和 `MULTICA_VCS_SECRET_KEY`（自托管 Git 集成的加密密钥）。 另外，PostgreSQL 以及 Multica 的前端和后端镜像也会随之拉取。

上述任务结束之后我们可以通过以下两个操作来确认服务是否已就绪：

```shellscript
# 查看容器状态
docker compose -f docker-compose.selfhost.yml ps

# 检查 backend、数据库以及 migration
curl -fsS http://localhost:8080/readyz

# 正常响应是 {"status":"ok","checks":{"db":"ok","migrations":"ok"}}
```

确认 ready 之后，我们就可以通过 `http://localhost:3000` 来访问了：

![](assets/XtzFV7FT8uGxnwD4evZ_EBW2N56RzWc9b2mftzI9Svo=.webp)

三金这里主要使用桌面应用，从 [Multica 下载页](https://multica.ai/download) 下载对应系统的安装包进行安装，然后使用和 Web 版相同的邮箱进行登录。登录后，Desktop 会启动自己内置的 Multica CLI，并检测本机已有的 AI 编程工具。

> Multica 默认是没有配置邮件服务的，所以我们输入自己的邮箱以后，需要从 backend 容器日志中进行读取：`docker compose -f docker-compose.selfhost.yml logs backend | grep "Verification code"
> `，然后在日志中会出现类似这样的内容：`[DEV] Verification code for you@example.com: 123456`

那如何配置邮件服务呢？可以通过在配置文件中设置主机和发件地址（以下只是引导，需要大家填写真实的配置信息），然后重启容器来激活邮件功能：

```shellscript
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USERNAME=multica
SMTP_PASSWORD=<password>
SMTP_FROM_EMAIL=noreply@example.com
```

### 快速上手

登录之后的第一件事，就是要创建工作区：

![](assets/GRPqZogS68aQud9M4d0WmGMfIUz43cWbEAEN_Zk7qEI=.webp)

以 AI-TEAM 为例，创建好之后视图如下：

![](assets/a--snvZYfMRJcASJVOpDWljGMNmMTZlEmGGIdUGIzzo=.webp)

我们先点击运行时来看下当前机器上都有哪些工具：

![](assets/06C1UJ2q3QEOKxBXI6gQ4BGvXbgtiVPOKxlwdnf_Fpg=.webp)

然后回到智能体中，创建新的智能体：

![](assets/o5_sFQerqf5USN_RPvJ9554zvPU-1m1BbpMWT0ZXH_U=.webp)

![](assets/krt6ZUj1FQli-ecrTG8v0aeXjxxGC_i0i45TTtC0f5w=.webp)

![](assets/FrkMbLjRkA5A8G5QveRietIsmE6H2l1mH-6h0La_-Zc=.webp)

接下来让我们来测试一下，输入提示词：在 Blender 中制作一只豚鼠。

![](assets/jz6j6q1LKToeUqSRai-LecR9hw7C9f8scvkGT95cV0E=.webp)

![](assets/SOPS5JT3b3ZOGx3ugRolY4PQ7lQV6BH8xuKGYRbWi0U=.webp)

![](assets/308kb91bUb2lDEpKuCuaWOvR6mzf6xC5eO9fzqf3Ruk=.webp)

![](assets/53kdsq9lnELjCA0wU3ZHG-Nk64R371KvTzz2nAxBFDc=.webp)

![](assets/AV8fpzwpwzR27Mt1JQbpwjDKM95vNvg4wP8WN0xJjU8=.webp)

这里用的 glm-5.3-flash 模型，形状是有的，但是有一种石质感，可能换成 gpt-6.1-sol 模型就好了。

> 有一说一，3D 建模的时间很长，足足一个多小时将近两个小时。

### 创建多 Agent

在测试单 Agent 成功以后，我们就可以尝试创建一个 Agent 团队出来了，比如搞一个专门做 3D 小游戏的团队：Game Designer + 3D 建模专家 + Game Development + Game QA & Playtest。

在小队菜单里将这几个 Agent 添加进去，3D 开发团队就这样诞生了（但是能不能投入使用肯定是需要很长的摸索迭代）：

![](assets/AQaX3Lx8RS2_VYXKm6ihHhJfCKSmuJpyYK-CRt2Covk=.webp)

### 组团打怪

我们新建一个任务，然后选择对应的团队：

![](assets/6om3jc1JIB1QVzl3wnXSFjJPEsJAEoLcAsX5UFIX0hA=.webp)

接下来就是等待时间，先由 Game Designer 进行设计，然后再让 3D 建模专家进行 3D 建模，再将建模素材转交给开发 Agent 进行开发（不过后面没有执行 QA🤷‍♀️）：

![](assets/iOnh1NRplISUpiOcliFZaJe6E7UXAtcfFMuUZ18bfrU=.webp)

经历了 1 个小时 16 分钟以后，一个划船游戏就完成了：

![](assets/SNCotSSd-8MBpXbZtS3MHqm_Bert1wZpFOC4dwCi-mk=.webp)

<iframe src="https://player.bilibili.com/player.html?bvid=BV1LW4A6BE5v&autoplay=0" scrolling="no" frameborder="no" allowfullscreen="true" style="width: 100%; aspect-ratio: 16 / 9;"></iframe>

### 看板

任务菜单这里，就是一个进度看板，在这里会展示「待规划\待办\进行中\审核中\已阻塞\已完成」6 个任务状态。

![](assets/cBJVG8Qk7oUDcRMSFqOQqG3grNTezKd1ENBCXXFP6is=.webp)

可以看到，我们在上一步中完成的长船之旅的任务目前已经到了审核阶段，用鼠标进行任务卡片拖动或者在任务详情中更换状态，都可以更新该任务到下一个阶段：

![](assets/X0gpQY9b_Rw3R-AuN2-yFd35awE5786WB2h5Z8ehO4Q=.webp)

![](assets/sGnwwoh-UQHqdpVdNpuEFg5MWbfrlzp7MmrxB0tfZj0=.webp)

### 定时任务

这是目前市面上大多数 AI Agent 客户端都有的能力：

![](assets/4KWzqGSibn3W2Nq0ToQ6YqFAuDU-92__XRqTNQ0LsT4=.webp)

包括 WorkBuddy、ZCode、Codex 这些，这里就不过多赘述了。

有需要做多 Agent 以及 AI 任务面板能力的小伙伴，可以试试 Multica。

Github：https://github.com/multica-ai/multica。觉得不错记得 star 哦～

官网：https://multica.ai

