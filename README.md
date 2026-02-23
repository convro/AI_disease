# AI Disease

**It spreads.** Autonomous AI agents that operate your social media accounts, execute missions, and propagate your message — 24/7, asynchronously, at scale.

---

## What Is This?

AI Disease is a self-hosted command & control platform for autonomous social media agents. You give it accounts, you give it missions, and it gets to work — sending messages, accepting friend requests, posting content, engaging with people — all without you lifting a finger.

Each agent operates independently and asynchronously. They don't waste tokens sitting idle — they know when to act and when to wait. If someone hasn't replied yet, the agent moves on and circles back later. If a friend request is still pending, it doesn't burn cycles refreshing — it checks back on a smart schedule.

You can run one agent or thirty. Each one can have its own personality, its own mission, its own accounts. They coordinate when needed and stay out of each other's way when they don't.

## How It Works

```
You (Dashboard/CLI)
    |
    v
+------------------+
|   Mission Control |  <-- Define missions, assign agents, monitor activity
+------------------+
    |
    v
+-----------+  +-----------+  +-----------+
|  Agent 1  |  |  Agent 2  |  |  Agent 3  |   <-- Autonomous workers
| (FB + IG) |  |  (FB x2)  |  |   (IG)    |
+-----------+  +-----------+  +-----------+
    |               |               |
    v               v               v
[ Browser Sessions via Playwright/Selenium on your VPS ]
    |               |               |
    v               v               v
  Facebook       Facebook       Instagram
```

1. **You create agents** — each agent is a worker with its own identity and behavior profile
2. **You link accounts** — attach Facebook, Instagram (more platforms coming) to each agent
3. **You define missions** — either broad ("spread awareness about X topic") or specific ("message these 50 people with Y")
4. **Agents execute autonomously** — they log in, perform actions, handle CAPTCHAs, respect rate limits, and adapt
5. **You observe** — real-time dashboard shows what each agent is doing, what's pending, what succeeded

## Missions

Missions are the core concept. A mission is a high-level objective that an agent breaks down into actionable steps using DeepSeek as its brain.

### Mission Examples

| Mission Type | Description |
|---|---|
| **Spread the Word** | Post about a topic across groups, walls, stories. Engage in relevant comment sections. |
| **Network Builder** | Send friend requests to targeted profiles, accept incoming ones, build connections. |
| **Messenger Outreach** | DM a list of people (or a type of people) with a personalized message. Follow up if no reply. |
| **Engagement Farm** | Like, comment, share content matching certain keywords or from certain pages. |
| **Custom Script** | Define a precise sequence of actions — the agent follows your exact playbook. |

Missions run **to completion**. If an agent gets blocked, rate-limited, or needs to wait for a human response, it pauses intelligently and resumes when conditions are right. No token waste. No blind retrying.

## Requirements

Before you start, you need four things:

| # | Requirement | Details |
|---|---|---|
| 1 | **Courage** | You're automating real social media accounts. Understand the risks. Accounts can get banned. Use wisely. |
| 2 | **DeepSeek API Tokens** | Agents use DeepSeek for reasoning and decision-making. Cheap, fast, uncensored enough for the job. Get your API key at [platform.deepseek.com](https://platform.deepseek.com). |
| 3 | **A VPS with Root Access** | You need a server you fully control. Root access is non-negotiable — the platform manages browser instances, proxies, cron jobs, and system-level automation. Any decent VPS provider works (Hetzner, OVH, Contabo, etc). |
| 4 | **Ubuntu (Linux)** | The platform is built for Ubuntu 22.04+. Other distros may work but are not officially supported. |

### Recommended VPS Specs

| Agents | RAM | CPU | Storage |
|---|---|---|---|
| 1-3 | 2 GB | 2 vCPU | 40 GB |
| 4-10 | 4 GB | 4 vCPU | 80 GB |
| 10-30 | 8 GB+ | 6+ vCPU | 120 GB+ |

Each agent runs a headless browser instance, so RAM is the main bottleneck.

## Quick Start

```bash
# 1. Clone the repo
git clone https://github.com/your-user/AI_disease.git
cd AI_disease

# 2. Run the installer (sets up Python, Playwright, system deps)
sudo ./install.sh

# 3. Configure your DeepSeek API key
cp .env.example .env
nano .env   # paste your DEEPSEEK_API_KEY

# 4. Launch the control panel
python3 main.py

# 5. Open the dashboard
# http://your-vps-ip:8080
```

## Architecture

```
AI_disease/
├── main.py                 # Entry point — starts the API server + dashboard
├── install.sh              # One-click installer for all system dependencies
├── .env.example            # Environment variable template
├── agents/
│   ├── agent.py            # Core agent class — lifecycle, state, scheduling
│   ├── brain.py            # DeepSeek integration — reasoning, decision-making
│   └── profiles/           # Stored agent personality & behavior configs
├── platforms/
│   ├── facebook.py         # Facebook automation — login, post, message, friend
│   ├── instagram.py        # Instagram automation — login, post, DM, follow
│   └── base.py             # Abstract platform interface
├── missions/
│   ├── mission.py          # Mission definition, state machine, progress tracking
│   ├── planner.py          # Breaks high-level missions into executable steps
│   └── templates/          # Pre-built mission templates
├── browser/
│   ├── session.py          # Browser session manager (Playwright)
│   ├── antidetect.py       # Fingerprint randomization, proxy rotation
│   └── captcha.py          # CAPTCHA detection and solving pipeline
├── dashboard/
│   ├── server.py           # Web UI backend (FastAPI)
│   ├── static/             # Frontend assets
│   └── templates/          # Dashboard HTML templates
├── scheduler/
│   ├── loop.py             # Main async event loop — orchestrates all agents
│   ├── queue.py            # Task queue — prioritization, deduplication
│   └── timing.py           # Smart timing — human-like delays, rate limit respect
├── storage/
│   ├── db.py               # SQLite/PostgreSQL — agent state, mission logs
│   └── models.py           # Database models
└── utils/
    ├── logger.py           # Structured logging — per-agent, per-mission
    ├── proxy.py            # Proxy management and rotation
    └── crypto.py           # Credential encryption at rest
```

## Key Design Principles

- **Token efficiency** — Agents don't call DeepSeek unless they need to make a decision. Routine actions (click button, fill form) are hard-coded. AI is only invoked for judgment calls (what to say, how to respond, whether to proceed).
- **Human-like behavior** — Random delays, realistic mouse movements, varied typing speed, session breaks. The goal is to look like a real person, not a bot.
- **Resilience** — If an action fails, the agent doesn't crash. It logs the failure, backs off, and retries with a different approach. Persistent state means agents survive server restarts.
- **Observability** — Every action is logged. The dashboard shows real-time feeds per agent. You always know what's happening.
- **Security** — Credentials are encrypted at rest. The dashboard is password-protected. No data leaves your VPS except to the platforms themselves and the DeepSeek API.

## Agent Lifecycle

```
CREATED ──> CONFIGURED ──> IDLE ──┐
                                   │
            ┌──────────────────────┘
            v
        EXECUTING ──> WAITING ──> EXECUTING ──> ...
            │                         │
            v                         v
        COMPLETED                  FAILED ──> RETRY
```

- **CREATED** — Agent exists but has no accounts linked
- **CONFIGURED** — Accounts linked, ready to receive missions
- **IDLE** — No active missions, standing by
- **EXECUTING** — Actively performing actions on a platform
- **WAITING** — Paused intelligently (waiting for reply, cooldown, rate limit)
- **COMPLETED** — Mission finished successfully
- **FAILED/RETRY** — Something went wrong, backing off and retrying

## Dashboard

The web dashboard gives you full control and visibility:

- **Agent Overview** — Status of all agents at a glance
- **Mission Control** — Create, assign, pause, cancel missions
- **Live Feed** — Real-time log of every action every agent takes
- **Account Health** — Login status, rate limit proximity, ban risk indicators
- **Analytics** — Messages sent, friends added, posts made, engagement metrics

## Roadmap

- [x] Core agent framework
- [x] Facebook automation (login, post, message, friend requests)
- [x] DeepSeek integration for agent reasoning
- [x] Mission system with async execution
- [ ] Instagram automation
- [ ] Web dashboard v1
- [ ] Proxy rotation and multi-IP support
- [ ] CAPTCHA solving pipeline
- [ ] Agent-to-agent coordination (avoid messaging the same person)
- [ ] Telegram platform support
- [ ] Twitter/X platform support
- [ ] Mission templates marketplace
- [ ] Mobile app for monitoring

## Disclaimer

This project is provided for **educational and research purposes only**. Automating interactions on social media platforms may violate their Terms of Service. The authors take no responsibility for how this software is used. You are solely responsible for your actions and any consequences that arise from using this tool.

**Use responsibly. Use at your own risk.**

## License

MIT
