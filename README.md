<p align="center">
  <img src="docs/marketing/monito-devto-cover.png" alt="monito banner" width="640">
</p>

<h1 align="center">monito — API 健康检查监控 | API Health Check Monitoring</h1>

<p align="center">
  <strong>基于 Cloudflare Workers 的多租户 API 健康检查——每月 $0 基础设施成本。<br>
  Multi-tenant API health checks on Cloudflare Workers — $0/mo infrastructure cost.</strong>
</p>

<p align="center">
  <a href="#功能--features">中文</a> •
  <a href="#overview">English</a> •
  <a href="#quick-start">Quick Start</a> •
  <a href="#architecture">Architecture</a> •
  <a href="#cli">CLI</a> •
  <a href="#api-documentation">API</a> •
  <a href="#deployment">Deploy</a>
</p>

---

## 功能 | Features

**monito** 是一个开源多租户 API 健康检查监控服务，完全运行在 **Cloudflare 免费层**上——Workers、D1（边缘 SQLite）、KV 和 Pages。零基础设施管理，零服务器成本。

它每 1-5 分钟检查你的 API 端点，跟踪滚动窗口正常运行时间（24h / 7d / 30d），服务宕机时通过邮件告警，并提供公开状态仪表盘——所有基础设施成本 **$0/月**。

**monito** is an open-source, multi-tenant API health check monitoring service that runs entirely on **Cloudflare's free tier** — Workers, D1 (SQLite on edge), KV, and Pages. Zero infrastructure to manage, zero server cost.

| 功能 Feature | 说明 Description |
|---|---|
| **分钟级检查 Minute-level checks** | 每个监控端点每分钟检查一次 Every endpoint checked once per minute |
| **多租户隔离 Multi-tenant isolation** | SHA-256 API 密钥哈希，原始密钥永不持久化 Raw keys never persisted |
| **滚动正常运行时间 Rolling uptime** | 24小时/7天/30天窗口，实时计算 24h/7d/30d windows, calculated live |
| **邮件告警 Email alerts** | 宕机+恢复通知（via Resend），可配置冷却防疲劳 Down + recovery with cooldown |
| **公开状态页 Public status page** | Cloudflare Pages 仪表盘，实时状态指示器 Real-time status indicators |
| **CLI 工具** | 终端管理监控项，单 Node.js 文件零外部依赖 Single file, zero deps |
| **REST API** | 监控项完整 CRUD、状态查询、等待列表管理 Full CRUD |
| **边缘部署 Edge-deployed** | Cloudflare Workers + D1，全球边缘网络 Global edge network |
| **响应波形 Response waveforms** | 状态页可视化延迟趋势 Visualize latency trends |

## 快速开始 | Quick Start

### 方式 A：使用托管版 Use hosted version

访问 Visit [monito.yycomyy.workers.dev](https://monito.yycomyy.workers.dev)

### 方式 B：自托管 Self-host

```bash
git clone https://github.com/DrifterXXX/monito.git && cd monito
npm install
wrangler d1 create monito-db        # → 粘贴 database ID 到 wrangler.toml
wrangler kv namespace create monito-state
wrangler d1 migrations apply monito-db --remote
wrangler deploy
wrangler secret put MONITO_API_KEY  # openssl rand -hex 32
wrangler secret put RESEND_API_KEY
```

---

## Overview


**monito** is an open-source, multi-tenant API health check monitoring service that runs entirely on **Cloudflare's free tier** — Workers, D1 (SQLite on edge), KV, and Pages. Zero infrastructure to manage, zero server cost.

It checks your API endpoints every 1-5 minutes, tracks uptime across rolling windows (24h / 7d / 30d), alerts you via email when services go down, and provides a public status dashboard — all for **$0/mo** in infrastructure costs.

I built monito because I was tired of paying $20/mo to monitor 11 personal endpoints. Read the full story on [the blog](https://monito.yycomyy.workers.dev/blog).

## Features

| Feature | Description |
|---------|-------------|
| **Minute-level checks** | Every monitored endpoint is checked once per minute |
| **Multi-tenant isolation** | SHA-256 API key hashing — raw keys never persisted. Monitors scoped per user |
| **Rolling uptime stats** | 24-hour, 7-day, and 30-day windows. Calculated live from check history |
| **Email alerts** | Down + recovery notifications via Resend. Configurable cooldown prevents alert fatigue |
| **Public status page** | Cloudflare Pages dashboard with real-time status indicators and uptime bars |
| **CLI tool** | Manage monitors from the terminal — single Node.js file, zero external deps |
| **REST API** | Full CRUD for monitors, status queries, waitlist management |
| **Edge-deployed** | Runs on Cloudflare Workers with D1 storage. Global edge network |
| **Response waveforms** | Visualize latency trends on the status page |

## Quick Start

### Option A: Use the hosted version

Visit [monito.yycomyy.workers.dev](https://monito.yycomyy.workers.dev), sign up for the waitlist, and you'll be notified when the hosted service opens.

### Option B: Self-host (full control)

```bash
# 1. Clone the repo
git clone https://github.com/DrfterX/monito.git
cd monito

# 2. Install dependencies
npm install

# 3. Create D1 database
wrangler d1 create monito-db
# → Paste the returned database ID into wrangler.toml

# 4. Create KV namespace
wrangler kv namespace create monito-state
# → Paste the returned namespace ID into wrangler.toml

# 5. Run migrations
wrangler d1 migrations apply monito-db --remote

# 6. Deploy
wrangler deploy

# 7. Set secrets
wrangler secret put MONITO_API_KEY
# Generate one: openssl rand -hex 32

wrangler secret put RESEND_API_KEY
# Get from: https://resend.com/api-keys

wrangler secret put DEFAULT_ALERT_FROM
wrangler secret put ALERT_EMAIL
```

That's it. In ~10 minutes you have your own multi-tenant API monitoring service.

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                Cloudflare Workers (Hono)                  │
│  ┌──────────┐   ┌──────────┐   ┌──────────────────┐     │
│  │  Hono    │   │  Cron    │   │  D1 Database     │     │
│  │  Router  │   │  Trigger │   │  (SQLite on Edge) │     │
│  └──────────┘   └──────────┘   └──────────────────┘     │
│       │              │                │                  │
│  ┌────┴────┐   ┌─────┴─────┐   ┌──────┴──────┐          │
│  │  Auth   │   │  Checker  │   │  KV Store   │          │
│  │ Middle- │   │  Engine   │   │ (cooldown,  │          │
│  │  ware   │   │           │   │  last_run)  │          │
│  └─────────┘   └───────────┘   └─────────────┘          │
│       │              │                                   │
│  ┌────┴────┐   ┌─────┴─────┐                            │
│  │  Resend │   │  Pages    │                            │
│  │  Alerts │   │  Status   │                            │
│  └─────────┘   │  Dashboard│                            │
│                └───────────┘                            │
└─────────────────────────────────────────────────────────┘
```

### Components

| Component | Role |
|-----------|------|
| **Cloudflare Workers** | API gateway + cron engine (Hono router, 60s cron trigger) |
| **D1 (SQLite on edge)** | Persistent storage: monitors, checks, API keys, waitlist |
| **KV** | Ephemeral state: cron timestamp, alert cooldowns (TTL-based) |
| **Resend** | Email alerts: down + recovery notifications |
| **Cloudflare Pages** | Public status dashboard (static HTML, live data via API) |

### Key Design Decisions

**Multi-tenant isolation**: API keys are SHA-256 hashed before storage — raw keys are never persisted. Every database query scopes by `user_id`, ensuring one user can never see another's data.

**Smart check scheduling**: The cron runs every 60 seconds, but only checks monitors that are due (based on their `check_interval`). A free-tier monitor with 5-minute interval doesn't get checked every tick.

**Alert cooldown**: After 3 consecutive failures, an alert fires. A 30-minute KV-based cooldown prevents alert storms. Recovery alerts fire immediately.

**Rolling uptime**: Uptime is calculated live from check history — a single SQL query with `COUNT(*)` and time-window offsets. No pre-aggregation, no background jobs, no stale data.

## CLI

monito ships with a command-line interface (single Node.js file, no dependencies beyond built-in modules):

```bash
# Log in with your API key
monito login mk_a1b2c3d4...

# Add an endpoint to monitor
monito add https://api.github.com --name "GitHub API" --email alerts@example.com

# List all monitors
monito list

# Quick status overview
monito status

# Remove a monitor
monito remove <id>
```

## API Documentation

### Public Endpoints (no auth required)

```
GET  /api/status         → Status overview (all monitors)
GET  /api/uptime         → Global uptime stats (?window=24h|7d|30d)
GET  /api/uptime?window  → Per-monitor uptime breakdown
POST /api/waitlist       → Sign up for early access
GET  /api/waitlist/count → Waitlist signup count
GET  /health             → System health (cron status)
```

### Authenticated Endpoints (x-api-key header)

```
POST   /api/monitors          → Create a monitor
GET    /api/monitors          → List monitors (scoped to your API key)
GET    /api/monitors/:id      → Get a single monitor
DELETE /api/monitors/:id      → Delete a monitor (soft delete)
GET    /api/monitors/:id/checks   → Check history (?limit=20, max 100)
GET    /api/monitors/:id/uptime   → Per-monitor uptime (?window=24h)
```

### Creating a Monitor

```bash
curl -X POST https://monito.yycomyy.workers.dev/api/monitors \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://api.github.com",
    "name": "GitHub API",
    "method": "HEAD",
    "check_interval": 300,
    "alert_email": "you@example.com"
  }'
```

## Deployment

### Prerequisites

- [Node.js](https://nodejs.org) 18+
- [Wrangler CLI](https://developers.cloudflare.com/workers/wrangler/) (`npm install -g wrangler`)
- A [Cloudflare](https://cloudflare.com) account (free tier)

### Production deploy

```bash
npm run build
npm run deploy
```

### Staging environment

```bash
wrangler deploy --env staging
```

### Seeding test data

```bash
# Add test monitors for the status page to show
# (repeat for each endpoint you want to monitor)
wrangler d1 execute monito-db --remote --command="
INSERT INTO monitors (id, url, name, method, status, check_interval)
VALUES ('$(openssl rand -hex 8)', 'https://api.github.com', 'GitHub API', 'HEAD', 'up', 300);
"
```

### Cron trigger

The cron job runs every 60 seconds (configured in `wrangler.toml`). It picks up all monitors due for a check, runs them concurrently (configurable concurrency), records results, and fires alerts if needed.

## Running in Production

monito has been running in production monitoring **11 endpoints** for weeks. Current statistics:

| Metric | Value |
|--------|-------|
| Infrastructure cost | **$0/mo** |
| Maintenance | **Zero** — no SSH, no restarts, no patching |
| Availability | **99%+** for all operational monitors |
| Cron interval | Every 60 seconds |

## Limitations

| Limitation | Reason | Workaround |
|------------|--------|------------|
| **Single-region checks** | Workers run in one region per request | Add check nodes in multiple Workers |
| **D1 write throughput** | ~1000 writes/sec per database | Fine for <500 monitors |
| **30s minimum interval** | Workers cron minimum is 60s | Run two Workers at offset intervals |
| **Email-only alerts** | Only Resend integration currently | Slack/SMS are open for PRs |
| **Workers CPU limit** | 30ms I/O + 10ms compute per request | Keep per-monitor processing minimal |

If you need 30-second checks across 10 regions with Slack, PagerDuty, and on-call scheduling — go pay for BetterStack. But if you have 5-15 personal projects and want monitoring that costs nothing, monito will more than do.

## Tech Stack

- **Runtime:** Cloudflare Workers, Node.js 18+
- **Language:** TypeScript
- **Framework:** [Hono](https://hono.dev) (edge-first Express-like router)
- **Database:** Cloudflare D1 (SQLite on edge, built on SQLite)
- **Cache/State:** Cloudflare KV (alert cooldowns, cron state)
- **Email:** Resend API (down + recovery alerts)
- **Hosting:** Cloudflare Pages (status dashboard)
- **CLI:** Single-file Node.js CLI (zero external deps)

## License

[MIT](LICENSE)

## Links

- [Product Page](https://monito.yycomyy.workers.dev)
- [Live Status Dashboard](https://monito-5sy.pages.dev)
- [Blog Post — How I Built monito](https://monito.yycomyy.workers.dev/blog)
- [GitHub Issues](https://github.com/DrfterX/monito/issues)
- [Waitlist — Hosted Version](https://monito.yycomyy.workers.dev)