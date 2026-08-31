# TrendRadar 详细使用教程

> 适用版本：**v6.10.0**（主程序）· **MCP v4.1.0**（AI 分析服务）
> 仓库：https://github.com/sansan0/TrendRadar ｜ 官网文档：https://trendradar.sandev.cc/zh/
> 本教程基于本地克隆 `C:\Users\wangjiejin\Desktop\TrendRadar-master` 编写

---

## 目录

1. [项目是什么](#一项目是什么)
2. [核心功能一览](#二核心功能一览)
3. [项目目录结构](#三项目目录结构)
4. [三种部署方式怎么选](#四三种部署方式怎么选)
5. [方式 A：Docker 部署（推荐）](#方式-adocker-部署推荐)
6. [方式 B：本地部署（uv / Windows 一键脚本）](#方式-b本地部署uv--windows-一键脚本)
7. [方式 C：GitHub Actions 部署（免费无服务器）](#方式-cgithub-actions-部署免费无服务器)
8. [配置详解](#八配置详解)
9. [推送渠道配置速查](#九推送渠道配置速查)
10. [AI 能力：分析推送 / 智能筛选 / 翻译 / MCP 对话](#十ai-能力分析推送--智能筛选--翻译--mcp-对话)
11. [查看网页版报告](#十一查看网页版报告)
12. [常见问题 FAQ](#十二常见问题-faq)
13. [从零到推送：3 分钟快速清单](#十三从零到推送3-分钟快速清单)

---

## 一、项目是什么

TrendRadar 是一个**热点新闻聚合与推送工具**：定时抓取全网热榜（知乎、微博、抖音、B 站、百度、今日头条等 11 个平台 + 任意 RSS 源），按你设定的关键词或 AI 兴趣筛选，再通过飞书/钉钉/企业微信/Telegram/邮件/ntfy/Bark/Slack/Webhook 推送到你的手机或邮箱。还提供：

- **AI 分析推送**：让大模型每天自动生成热点洞察报告推给你
- **MCP 智能分析**：通过标准 MCP 协议接入 Cherry Studio / Cursor / Claude Desktop 等客户端，用自然语言对话查询和分析本地新闻数据

**核心特点**：轻量（约 10 个 Python 依赖）、最快 30 秒部署、数据本地可控（SQLite）、支持 Docker / GitHub Actions / 本地三种运行方式。

---

## 二、核心功能一览

| 功能 | 说明 | 默认 |
|---|---|---|
| 全网热点聚合 | 知乎、抖音、bilibili、华尔街见闻、贴吧、百度、财联社、澎湃、凤凰、头条、微博 共 11 个平台 | 全开 |
| RSS 订阅 | 任意 RSS/Atom 源，与热榜统一格式、合并推送、新鲜度过滤 | 开（含 HN、雅虎财经） |
| 精准筛选 | `frequency_words.txt`关键词（支持分组/正则/必须词/过滤词/别名） | 有关键词 |
| AI 智能筛选 (v6.5+) | 自然语言兴趣描述 `ai_interests.txt`，AI 自动分类打分 | 关（filter.method=keyword） |
| 三种推送模式 | daily 当日汇总 / current 当前榜单 / incremental 增量监控（零重复） | current |
| 调度系统 (v6.0+) | `timeline.yaml`预设：always_on / morning_evening / office_hours / night_owl / custom | 关（schedule.enabled=false） |
| 热点趋势分析 | 时间轴、排名变化、新增 🆕 标记、跨平台对比 | 开 |
| 多渠道推送 | 企业微信(+个人微信)、飞书、钉钉、Telegram、邮件、ntfy、Bark、Slack、通用 Webhook | 需自配 |
| AI 分析推送 (v5.0+) | 大模型生成热点洞察报告，LiteLLM 支持 100+ 提供商 | 开（需 API Key） |
| AI 多语言翻译 (v5.2+) | 推送标题翻译为任意语言 | 开（目标中文） |
| HTML 报告 | 网页版报告（宽屏/暗色/搜索/快捷键），邮件需 `storage.formats.html: true` | 开 |
| 存储架构 | 本地 SQLite（默认）/ S3 兼容云存储（R2/OSS/COS） | auto |
| MCP 服务 | `mcp_server` 提供 17 个工具，供 AI 客户端对话分析 | 独立服务 |

---

## 三、项目目录结构

```
TrendRadar/
├── config/                     # 所有配置都在这里（Docker 挂载只读）
│   ├── config.yaml             # ★ 主配置：平台/RSS/推送模式/通知/AI/存储
│   ├── frequency_words.txt     # ★ 关键词文件（最常改）
│   ├── timeline.yaml           # 调度时间线（preset 预设 + custom 自定义）
│   ├── ai_analysis_prompt.txt  # AI 分析提示词（自定义“人设”）
│   ├── ai_translation_prompt.txt
│   ├── ai_interests.txt        # AI 智能筛选的兴趣描述
│   ├── ai_filter/              # AI 筛选内部提示词（一般不动）
│   └── custom/                 # 自定义扩展（避免被升级覆盖）
│       ├── ai/                 #   自定义 AI 提示词
│       └── keyword/            #   自定义关键词文件
├── docker/
│   ├── .env                    # ★ 敏感信息 + Docker 特有配置（gitignore）
│   ├── docker-compose.yml      # 编排：trendradar + trendradar-mcp
│   ├── docker-compose-build.yml# 本地构建版
│   ├── Dockerfile / Dockerfile.mcp
│   ├── entrypoint.sh
│   └── manage.py               # 容器内管理命令入口
├── trendradar/                 # 主程序源码（Python 包）
│   ├── __main__.py             # CLI 入口（python -m trendradar）
│   ├── context.py / core/      # 配置加载、调度、分析核心
│   ├── crawler/                # 热榜抓取 + RSS
│   ├── ai/                     # AI 分析/翻译/筛选（LiteLLM）
│   ├── notification/           # 各渠道推送（批量/拆分/格式化）
│   ├── report/                 # HTML/TXT/Markdown 报告生成
│   ├── storage/                # SQLite 本地 + S3 远程
│   └── commands/               # doctor / status / test_notification 等
├── mcp_server/                 # MCP AI 分析服务（fastmcp）
│   ├── server.py               # 入口（python -m mcp_server.server）
│   ├── services/               # 数据/缓存/解析服务
│   └── tools/                  # 17 个 MCP 工具
├── output/                     # ★ 数据目录：SQLite + HTML 报告（含 7 天测试数据）
│   └── news/2025-12-*.db
├── docs/                       # 官网静态站源码
├── .github/workflows/          # crawler.yml（GHA 定时任务）/ clean-crawler.yml / docker.yml
├── pyproject.toml              # Python >=3.12，依赖见下
├── setup-windows.bat / setup-mac.sh / setup-windows-en.bat  # 一键安装脚本
├── start-http.bat / start-http.sh                            # 启动 MCP HTTP 服务
├── version / version_configs / version_mcp                   # 版本号
└── index.html                  # GitHub Pages 网页报告
```

**Python 依赖**（`pyproject.toml`）：requests、PyYAML、fastmcp、websockets、feedparser、boto3（S3）、litellm（AI）、json-repair、tenacity、pytz。要求 **Python ≥ 3.12**，uv 会自动管理。

---

## 四、三种部署方式怎么选

| 方案 | 优点 | 缺点 | 适合谁 |
|---|---|---|---|
| **A. Docker**（推荐） | 稳定、定时准确、数据本地、一条命令更新 | 需要 Docker 环境 | 有服务器/NAS/常开电脑 |
| **B. 本地 uv** | 无需 Docker、双击脚本即装、适合调试 | 电脑需常开；定时需自配计划任务 | Windows/Mac/Linux 单机用户 |
| **C. GitHub Actions** | 完全免费、无服务器 | 定时有 ±15 分钟偏差；数据在云存储；需每 7 天手动“签到”续期 | 没有服务器的用户 |

> 建议：先用 **B（本地）** 或 **A（Docker）** 跑通推送，再决定要不要上 GHA。

---

## 方式 A：Docker 部署（推荐）

### A1. 前置条件
安装 Docker Desktop（Windows）或 Docker Engine（Linux/NAS），并启动 Docker 服务。

### A2. 准备配置
```bash
git clone https://github.com/sansan0/TrendRadar.git
cd TrendRadar
```

需要编辑的文件（都在项目内）：
- `config/config.yaml` —— 推送模式、平台开关、RSS、AI 等（功能行为）
- `config/frequency_words.txt` —— 你的关注关键词
- `docker/.env` —— **Webhook / API Key 等敏感信息**（不会被 git 追踪）

> 配置分工：功能 → `config.yaml`；关注内容 → `frequency_words.txt`；密钥凭证 → `docker/.env`。环境变量会**覆盖** `config.yaml` 对应项。

### A3. 启动
```bash
cd docker

# 方案 A：全部服务（新闻推送 + MCP AI 分析）
docker compose pull
docker compose up -d

# 方案 B：只要新闻推送（大多数用户）
docker compose pull trendradar
docker compose up -d trendradar

# 方案 C：只要 MCP AI 分析
docker compose pull trendradar-mcp
docker compose up -d trendradar-mcp
```

容器首次启动会立即执行一次抓取（`IMMEDIATE_RUN=true`），之后按 `CRON_SCHEDULE`（默认每 30 分钟）运行。

### A4. 常用管理命令
```bash
docker logs -f trendradar                                   # 实时日志
docker exec -it trendradar python manage.py status          # 运行状态
docker exec -it trendradar python manage.py run             # 手动执行一次抓取+推送
docker exec -it trendradar python manage.py logs            # 查看日志
docker exec -it trendradar python manage.py config          # 显示当前生效配置
docker exec -it trendradar python manage.py files           # 显示输出文件
docker exec -it trendradar python manage.py start_webserver # 启动网页服务器(默认自动)
docker exec -it trendradar python manage.py stop_webserver
docker exec -it trendradar python manage.py webserver_status
docker exec -it trendradar python manage.py help            # 全部命令
docker restart trendradar                                   # 重启
docker compose stop trendradar                              # 停止
```

### A5. 更新镜像
```bash
cd docker
docker compose pull
docker compose up -d
```

### A6. 数据持久化
- 数据保存在宿主机 `output/` 目录（compose 已挂载 volume），删容器不丢数据
- 网页报告：浏览器打开 `http://localhost:8080`（端口在 `.env` 的 `WEBSERVER_PORT`，默认 8080）
- 历史报告：`output/html/YYYY-MM-DD/当日汇总.html`

### A7. 故障排查
```bash
docker inspect trendradar          # 容器状态
docker logs --tail 100 trendradar  # 最近 100 行日志
docker exec -it trendradar /bin/bash   # 进容器
docker exec -it trendradar ls -la /app/config/  # 检查配置挂载
```

---

## 方式 B：本地部署（uv / Windows 一键脚本）

> 当前机器的项目路径：`C:\Users\wangjiejin\Desktop\TrendRadar-master`

### B1. 安装 uv（无需预装 Python）
PowerShell 执行：
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```
macOS/Linux：
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### B2. 安装依赖
```bash
cd C:\Users\wangjiejin\Desktop\TrendRadar-master
uv sync          # 自动下载 Python 3.12 + 全部依赖
```
或直接双击 `setup-windows.bat`（Mac 用 `bash setup-mac.sh`）。

### B3. 编辑配置
- 打开 `config/config.yaml`：填推送渠道（`notification.channels`）、AI Key（`ai.api_key`）等
- 编辑 `config/frequency_words.txt` 写关键词

### B4. 运行
```bash
uv run python -m trendradar                     # 完整跑一次：抓取→分析→推送
uv run python -m trendradar --test-notification # 只发一条测试通知到已配置渠道
uv run python -m trendradar --doctor            # 环境与配置体检
uv run python -m trendradar --show-schedule     # 显示当前调度状态
```

> 本地运行默认会**自动打开浏览器**展示报告（非 Docker/GHA 环境）。
> 想定时自动跑？Windows 可用「任务计划程序」定时执行上面的命令，或直接改用 Docker（自带 cron）。

### B5. MCP 服务（AI 对话分析）
```bash
uv run python -m mcp_server.server             # STDIO 模式（被客户端拉起）
# 或
start-http.bat                                  # HTTP 模式，监听 127.0.0.1:3333
```
健康检查：浏览器访问 `http://127.0.0.1:3333/mcp` 或 `curl http://127.0.0.1:3333/mcp`。

---

## 方式 C：GitHub Actions 部署（免费无服务器）

### C1. 创建你自己的仓库
打开 [sansan0/TrendRadar](https://github.com/sansan0/TrendRadar) → 右上角绿色 **Use this template** → Create a new repository。（⚠️ 优先用 template 而非 fork，见 Issue #606）

### C2. 添加 GitHub Secrets
路径：仓库 → `Settings` → `Secrets and variables` → `Actions` → `New repository secret`。
**Name 必须严格一致**，一个 Name 对应一个 Secret，可同时配置多个平台。

| Name | 值 | 必填 |
|---|---|---|
| `WEWORK_WEBHOOK_URL` | 企业微信机器人 Webhook（个人微信方案加 `WEWORK_MSG_TYPE=text`） | 选 |
| `FEISHU_WEBHOOK_URL` | 飞书群自定义机器人 Webhook | 选 |
| `DINGTALK_WEBHOOK_URL` | 钉钉自定义机器人 Webhook（安全设置关键词填“热点”） | 选 |
| `TELEGRAM_BOT_TOKEN` + `TELEGRAM_CHAT_ID` | Telegram 机器人（@BotFather 创建；Chat ID 用 getUpdates 或 @userinfobot） | 选 |
| `EMAIL_FROM` + `EMAIL_PASSWORD` + `EMAIL_TO` | 邮件（授权码而非密码；可选 `EMAIL_SMTP_SERVER`/`EMAIL_SMTP_PORT`） | 选 |
| `NTFY_TOPIC` | ntfy 主题名（如 `trendradar-zs-8492`；可选 `NTFY_SERVER_URL`/`NTFY_TOKEN`） | 选 |
| `BARK_URL` | iOS Bark 推送 URL | 选 |
| `SLACK_WEBHOOK_URL` | Slack Incoming Webhook | 选 |
| `GENERIC_WEBHOOK_URL` | 通用 Webhook（可选 `GENERIC_WEBHOOK_TEMPLATE`） | 选 |
| `S3_BUCKET_NAME` / `S3_ACCESS_KEY_ID` / `S3_SECRET_ACCESS_KEY` / `S3_ENDPOINT_URL` | S3 云存储（R2/OSS/COS）4 项；可选 `S3_REGION` | 建议 |
| `AI_API_KEY` + `AI_PROVIDER` | AI 分析推送（如 deepseek / openai） | 选 |

### C3. 手动测试
1. 进入你的仓库 → **Actions** 标签
2. 找到 **“Get Hot News”** 工作流 → 右侧 **Run workflow**
3. 约 3 分钟后消息推送到你配置的渠道
4. 测试成功后，`config/` 下的配置可直接在仓库里编辑提交

### C4. 签到续期（重要）
v4.0 起 GHA 有活跃度检测：**每 7 天需手动运行一次 “Check In” 工作流**续期，否则服务挂起。路径：`Actions` → `Check In` → `Run workflow`。

### C5. 可选：Cloudflare Pages 加速访问
配置 `CLOUDFLARE_API_TOKEN` / `CLOUDFLARE_ACCOUNT_ID` / `CLOUDFLARE_PROJECT_NAME` 三个 Secret，每次运行自动把网页报告部署到 `https://<项目名>.pages.dev`（国内访问比 GitHub Pages 快）。

---

## 八、配置详解

### 8.1 config/config.yaml 分段说明

| 配置段 | 作用 | 常用项 |
|---|---|---|
| `app` | 时区、版本提示 | `timezone: "Asia/Shanghai"` |
| `schedule` | 调度系统总开关 | `enabled: false`，`preset: "morning_evening"` |
| `platforms` | 热榜平台 | `enabled`、`api_url`（自建 newsnow）、`sources` 列表（可加平台/改显示名/配 expected_domain） |
| `rss` | RSS 订阅 | `enabled`、`freshness_filter.max_age_days`（默认 1 天）、`feeds` 列表 |
| `report` | 报告模式 | `mode`: daily/current/incremental；`display_mode`: keyword/platform；`max_news_per_keyword` |
| `filter` | 筛选策略 | `method`: keyword（默认）| ai；`priority_sort_enabled` |
| `ai_filter` | AI 筛选参数 | `min_score: 0.7`（0~1，阈值越高越严）、`batch_size`、`reclassify_threshold` |
| `display` | 推送区域 | `region_order`（顺序）、`regions`（hotlist/new_items/rss/standalone/ai_analysis 开关）、`standalone` 独立展示区 |
| `notification` | 推送总开关+渠道 | `enabled`；`channels.feishu/dingtalk/wework/telegram/email/ntfy/bark/slack/generic_webhook` |
| `storage` | 存储 | `backend`: auto/local/remote；`formats`（sqlite 必须 true，html 邮件必开）；`remote` S3 配置 |
| `ai` | AI 模型（共用） | `model`（LiteLLM 格式如 `deepseek/deepseek-v4-flash`）、`api_key`、`api_base`、`timeout`、`fallback_models` |
| `ai_analysis` | AI 分析推送 | `enabled`、`language`、`mode`（follow_report/daily/...）、`max_news_for_analysis: 150`、`include_rss` |
| `ai_translation` | AI 翻译 | `enabled`、`language`、`scope`（hotlist/rss/standalone） |
| `advanced` | 高级 | `debug`、`crawler.request_interval`、`weight`（rank/frequency/hotness 权重，合计=1）、`max_accounts_per_channel` |

### 8.2 frequency_words.txt 关键词语法

文件分两个区：`[GLOBAL_FILTER]`（全局过滤，优先级最高）和 `[WORD_GROUPS]`（词组定义）。**词组之间用空行分隔**，同组内是“或”的关系。

| 语法 | 含义 | 示例 |
|---|---|---|
| `关键词` | 标题包含即匹配 | `华为` |
| `/正则/` | 正则匹配（自动忽略大小写） | `/\bai\b/ => AI 相关` |
| `关键词 => 别名` | 自定义显示名称 | `deepseek => DeepSeek 动态` |
| `[组别名]` | 整组显示名（组首行） | `[华为]` |
| `+关键词` | 必须词，全部满足才算匹配 | `+发布` |
| `!关键词` | 过滤词，排除该条（仅当前组） | `!广告` |
| `@数字` | 该组最多显示 N 条 | `@5` |

示例：
```txt
[GLOBAL_FILTER]
震惊
广告

[WORD_GROUPS]
iPhone
华为
OPPO
+发布        # 必须含“发布”

A股
上证
+涨跌
!预测        # 排除预测类

/\bai\b/i => AI 相关
人工智能
```

> 配置技巧：先宽后严——先用宽泛词测试，误匹配就加 `+` 必须词，干扰就加 `!` 过滤词。不会写正则直接让 AI 生成。留空文件 = 不筛选，推送全部热点。

### 8.3 timeline.yaml 调度（什么时候推送）

- 总开关在 `config.yaml`：`schedule.enabled: true` + 选一个 `preset`
- 预设：`always_on`（全天增量推）｜`morning_evening`（全天推送+晚间 20:00-22:00 当日汇总，推荐）｜`office_hours`（工作日三段式）｜`night_owl`（午后速览+深夜汇总）｜`custom`（改 timeline.yaml 底部 custom 段）
- 每个时间段可独立设置 `collect / analyze / push / report_mode / ai_mode / filter_method / frequency_file / once`（限推一次）
- **总开关优先**：`notification.enabled: false` 永远不推；`ai_analysis.enabled: false` 永远不分析
- GHA 用户时间段建议留 ≥2 小时余量（定时有偏差）；Docker 定时准确无此限制

### 8.4 docker/.env（敏感信息 + Docker 特有配置）

| 变量 | 说明 | 默认 |
|---|---|---|
| `WEBSERVER_PORT` | 网页报告端口 | 8080 |
| `FEISHU_WEBHOOK_URL` 等渠道变量 | 多账号用 `;` 分隔（配对项数量需一致） | 空 |
| `AI_API_KEY` / `AI_MODEL` / `AI_API_BASE` / `AI_ANALYSIS_ENABLED` | AI 配置（覆盖 config.yaml） | 空 |
| `S3_*` | 远程存储 5 个参数 | 空 |
| `CRON_SCHEDULE` | 定时表达式（cron 格式） | `*/30 * * * *` |
| `RUN_MODE` | cron（定时）| once（单次） | cron |
| `IMMEDIATE_RUN` | 启动时立即执行一次 | true |
| `MCP_HOST` / `MCP_PORT` | MCP 服务监听地址/端口 | 127.0.0.1:3333 |

**优先级：环境变量 > config.yaml**

---

## 九、推送渠道配置速查

| 渠道 | config.yaml 字段 | docker/.env 变量 | GitHub Secret |
|---|---|---|---|
| 飞书 | `channels.feishu.webhook_url` | `FEISHU_WEBHOOK_URL` | `FEISHU_WEBHOOK_URL` |
| 钉钉 | `channels.dingtalk.webhook_url` | `DINGTALK_WEBHOOK_URL` | `DINGTALK_WEBHOOK_URL` |
| 企业微信 | `channels.wework.webhook_url` + `msg_type` | `WEWORK_WEBHOOK_URL` / `WEWORK_MSG_TYPE` | 同上 |
| Telegram | `channels.telegram.bot_token` + `chat_id` | `TELEGRAM_BOT_TOKEN` / `TELEGRAM_CHAT_ID` | 同上 |
| 邮件 | `channels.email.*` | `EMAIL_FROM/PASSWORD/TO`（可选 SMTP） | 同上 |
| ntfy | `channels.ntfy.*` | `NTFY_TOPIC`（可选 SERVER_URL/TOKEN） | `NTFY_TOPIC` |
| Bark | `channels.bark.url` | `BARK_URL` | `BARK_URL` |
| Slack | `channels.slack.webhook_url` | `SLACK_WEBHOOK_URL` | `SLACK_WEBHOOK_URL` |
| 通用 Webhook | `channels.generic_webhook.*` | `GENERIC_WEBHOOK_URL`（可选 TEMPLATE） | 同上 |

> ⚠️ 钉钉安全设置需自定义关键词填“热点”；邮件推送必须 `storage.formats.html: true`；webhook 地址视为密码，切勿公开。

---

## 十、AI 能力：分析推送 / 智能筛选 / 翻译 / MCP 对话

AI 功能共用一段模型配置（`ai` 段 / `AI_API_KEY`、`AI_PROVIDER` 环境变量），支持 DeepSeek、OpenAI、Gemini、Anthropic、本地 Ollama 等 100+ 提供商（LiteLLM）。

### 10.1 AI 分析推送（被动日报）
`config.yaml` 里 `ai_analysis.enabled: true` + 填 `ai.api_key` 即可。默认模型 `deepseek/deepseek-v4-flash`。自定义分析角度改 `config/ai_analysis_prompt.txt`；控成本调 `ai_analysis.max_news_for_analysis`。

### 10.2 AI 智能筛选（自然语言过滤）
```yaml
filter:
  method: ai          # keyword（默认）| ai
ai_filter:
  min_score: 0.7      # 推送阈值 0.0~1.0
```
兴趣写在 `config/ai_interests.txt`。失败自动回退关键词匹配，推送不中断。

### 10.3 AI 翻译
```yaml
ai_translation:
  enabled: true
  language: "中文"     # 目标语言
```

### 10.4 MCP 智能分析（主动对话）

**原理**：MCP 服务读取本地 `output/` 中的新闻数据（不是实时网上的），所以先跑几天积累数据，或直接用自带的 **2025-12-21 ~ 12-27 测试数据**体验。

**启动方式**：

| 模式 | 命令 | 特点 |
|---|---|---|
| STDIO（推荐） | 客户端拉起 `uv --directory <项目路径> run python -m mcp_server.server` | 一次配置永久可用 |
| HTTP | `start-http.bat`（Windows）/`./start-http.sh`（Mac/Linux）；Docker 则 `docker compose up -d trendradar-mcp` | 一行 URL，需手动启动 |

**Cherry Studio 配置**（零基础推荐）：
1. 下载 https://cherry-ai.com/
2. Windows 双击 `setup-windows.bat` 安装依赖（Mac 拖 `setup-mac.sh` 到终端）
3. Cherry Studio → 设置 → **MCP 服务器** → 添加：
   - STDIO：填 setup 脚本显示的 command + args（uv --directory ... run python -m mcp_server.server）
   - HTTP：类型 `streamableHttp`，URL `http://127.0.0.1:3333/mcp`
4. 保存并开启开关 ✅，对话测试：“搜索最近3天关于人工智能的新闻”

**Cursor / Cline / Continue 配置模板**（`.cursor/mcp.json` 等）：
```json
{
  "mcpServers": {
    "trendradar": {
      "command": "uv",
      "args": [
        "--directory",
        "C:\\Users\\wangjiejin\\Desktop\\TrendRadar-master",
        "run",
        "python",
        "-m",
        "mcp_server.server"
      ]
    }
  }
}
```

**MCP 提供的 17 个工具**：
- 基础查询：`get_latest_news`、`get_news_by_date`、`get_trending_topics`
- 智能检索：`search_news`、`find_related_news`
- 高级分析：`analyze_topic_trend`、`analyze_data_insights`、`analyze_sentiment`、`aggregate_news`、`compare_periods`、`generate_summary_report`
- RSS 查询：`get_latest_rss`、`search_rss`、`get_rss_feeds_status`
- 系统管理：`get_current_config`、`get_system_status`、`resolve_date_range`

**对话示例**：
```
分析最近7天“特斯拉”的热度变化趋势
生成今天的热点摘要报告
搜索“比特币”相关新闻并分析情感倾向
```

> Docker 部署时 MCP 服务在 `127.0.0.1:3333`，Cherry Studio 直接填 `http://127.0.0.1:3333/mcp`（streamableHttp）。

---

## 十一、查看网页版报告

| 场景 | 路径 |
|---|---|
| Docker | `http://localhost:8080`（Web 服务器 cron 模式自动启动） |
| 本地 | 运行后自动开浏览器；或直接打开 `output/index.html` |
| 历史归档 | `output/html/YYYY-MM-DD/当日汇总.html`；Docker 里 `http://localhost:8080/html/YYYY-MM-DD/` |
| GitHub Pages | 部署后访问 `https://<你的用户名>.github.io/<仓库名>/` |

浏览器打开后支持：`W` 宽屏、`D` 暗色、`/` 搜索、`?` 快捷键、悬停序号复制新闻。

---

## 十二、常见问题 FAQ

1. **增量模式收不到推送？** 说明当前时段没有符合关键词的新热点。调整关键词、换 current/daily 模式或增加监控平台。
2. **邮件推送报“HTML文件不存在”？** 确保 `storage.formats.html: true`。
3. **推送时间不对？** 检查 `app.timezone`；GHA 用的是 UTC（北京时间 = UTC+8），且执行有 ±15 分钟偏差。
4. **GHA 不运行了？** 检查是否超过 7 天没“Check In”签到。
5. **飞书 2026-06-30 后失效？** 原 BotBuilder 机器人下线，改用“群组自定义机器人”并重新获取 Webhook。
6. **AI 报错 / 模型不支持？** 检查 `ai.api_key` 与 `ai.model`（LiteLLM 格式 `提供商/模型名`）；部分模型要求 `temperature=1.0`、不支持 `max_tokens` 时设为 0。
7. **MCP 查不到最新新闻？** MCP 只分析本地 `output/` 已有数据，先让爬虫跑几天，或开启 `storage.pull.enabled` 从远程拉取。
8. **端口被占用？** 改 `.env` 的 `WEBSERVER_PORT` / `MCP_PORT` 后重启容器。
9. **想同时推多个群/账号？** 渠道值用 `;` 分隔；Telegram 的 token 与 chat_id 数量必须一一对应。
10. **GHA 数据丢失 / 无历史追踪？** 必须配置 S3 云存储（R2/OSS/COS），否则运行在轻量模式。

---

## 十三、从零到推送：3 分钟快速清单

**场景：本机（Windows）快速体验，推送用飞书/企业微信，AI 用 DeepSeek**

1. **装环境**：安装 uv（PowerShell 执行 install.ps1 命令）；本目录已含项目，无需 clone
2. **装依赖**：`uv sync`（或双击 `setup-windows.bat`）
3. **配关键词**：编辑 `config/frequency_words.txt`，把关注词写好（如“AI、比亚迪”）
4. **配推送渠道**：`config/config.yaml` → `notification.channels`，填一个渠道（如飞书 webhook_url）
5. **配 AI（可选）**：`ai.api_key` 填 DeepSeek Key，`ai_analysis.enabled: true`
6. **测试通知**：`uv run python -m trendradar --test-notification`（渠道通不通，先验证）
7. **跑一次**：`uv run python -m trendradar` —— 抓取 11 平台+RSS → 筛选 → AI 分析 → 推送
8. **看报告**：`output/index.html` 浏览器打开
9. **定时化**：装 Docker → `cd docker && docker compose up -d`（自带每 30 分钟定时 + 网页服务）
10. **进阶**：接 Cherry Studio（STDIO）用自然语言深度分析本地新闻数据

---

*本教程依据仓库 README.md / config 注释 / 源码整理，具体以 [官方文档](https://github.com/sansan0/TrendRadar?tab=readme-ov-file) 与 config 内注释为准。*


---

# 附录：GitHub Pages + ntfy 推送 实战部署

> 目标：GitHub Actions 定时抓取 → ntfy 推送到手机 + 报告自动发布到 GitHub Pages。
> 前置：一个 GitHub 账号。本地这份代码已把 `.github/workflows/crawler.yml` 改好（加了"提交 index.html 回仓库"步骤），直接可用。

## ① 创建你的仓库（二选一）

**方式 A（推荐，官方 template）**：打开 https://github.com/sansan0/TrendRadar → 绿色 **Use this template** → Create a new repository（例如 `my-trendradar`）。之后把本地改好的 `.github/workflows/crawler.yml` 覆盖到仓库对应路径（或直接把本地这份代码整体 push 上去）。

**方式 B（push 本地这份）**：GitHub 新建空仓库 → 本地执行：
```bash
cd C:\Users\wangjiejin\Desktop\TrendRadar-master
git init
git add .
git commit -m "init TrendRadar"
git branch -M master
git remote add origin https://github.com/<你的用户名>/<仓库名>.git
git push -u origin master
```

## ② 开启 GitHub Pages

仓库 → **Settings** → **Pages** → Source 选 **Deploy from a branch** → 分支 `master`，目录 `/ (root)` → **Save**。
访问 `https://<你的用户名>.github.io/<仓库名>/`，立刻能看到仓库里已有的示例报告页面。

## ③ 配置 ntfy 推送

1. 手机安装 ntfy App（Google Play / F-Droid / iOS App Store），或电脑打开 https://ntfy.sh
2. 订阅一个**难猜的主题名**：建议格式 `trendradar-你的名字缩写-随机数字`（如 `trendradar-zs-8492`，不能是中文）
3. 先手动测试（手机应收到消息）：
   ```bash
   curl -d "测试消息" https://ntfy.sh/trendradar-zs-8492
   ```
4. 仓库 → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**，添加：

| Name | 值 | 必填 |
|---|---|---|
| `NTFY_TOPIC` | 你的主题名（如 trendradar-zs-8492） | ✅ 必填 |
| `NTFY_SERVER_URL` | 自托管服务器地址；留空默认 https://ntfy.sh | 可选 |
| `NTFY_TOKEN` | 私有主题的访问令牌 | 可选 |

## ④ 工作流已支持 Pages 自动更新

本地 `crawler.yml` 已修改：
- 权限 `contents: read` → `contents: write`（允许把报告提交回仓库）
- 新增步骤 **"Deploy to GitHub Pages (commit index.html)"**：每次爬取成功后把根目录 `index.html` 提交回 master 分支，Pages 约 1 分钟后自动更新；报告无变化时自动跳过提交。
- 注意：该工作流只触发于 `schedule` 和 `workflow_dispatch`，提交报告不会造成死循环。

## ⑤ 手动测试

1. 仓库 → **Actions** → **"Get Hot News"** → 右侧 **Run workflow**
2. 等 3~5 分钟，检查：
   - ✅ 手机收到 ntfy 热点推送
   - ✅ 运行日志出现 `✅ 已提交 index.html`
   - ✅ 刷新 `https://<你的用户名>.github.io/<仓库名>/` 看到最新报告

## ⑥ 日常维护

- **签到续期（务必）**：每 7 天去 Actions 运行一次 **"Check In"** 工作流，否则 "Get Hot News" 会自动停用
- **改关注词**：直接在仓库里编辑 `config/frequency_words.txt` 提交即可（无需改 Secrets）
- **改推送频率**：编辑 `crawler.yml` 第 38 行 `cron: "33 * * * *"`，只改第一个数字（0-59）= 每小时第几分钟
- **没有 S3 云存储 = 轻量模式**：current/daily 推送正常；incremental 增量推送与跨次历史追踪不可用。需要完整功能就配 Cloudflare R2（免费 10GB/月）

## ⑦ 常见坑

- GitHub Actions 定时用 UTC 且有 ±15 分钟偏差，别把时间段卡太紧
- ntfy 免费额度每天 250 条，30 分钟一次绰绰有余
- 主题名就是"密码"，别用 `news`、`alerts` 这种易猜的名字
- 首次运行时 "Check Expiration" 步骤会跳过（没有历史记录），从第二次起开始计算 7 天周期
