<div align="center">

<img src="https://img.shields.io/badge/n8n-Workflow-EA4B71?style=for-the-badge&logo=n8n&logoColor=white" />
<img src="https://img.shields.io/badge/Groq-Llama_3.3_70B-F55036?style=for-the-badge" />
<img src="https://img.shields.io/badge/Google-Gemini-4285F4?style=for-the-badge&logo=google&logoColor=white" />
<img src="https://img.shields.io/badge/Stability_AI-Image_Gen-7C3AED?style=for-the-badge" />
<img src="https://img.shields.io/badge/LinkedIn-Auto_Post-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white" />
<img src="https://img.shields.io/badge/Status-Production_Ready-22c55e?style=for-the-badge" />

<br /><br />

# 🤖 LinkedIn AI Auto-Poster

### A fully automated, AI-powered LinkedIn content pipeline built on n8n.
### Researches trends → Writes posts → Generates images → Publishes. Every 2 days. Zero manual effort.

<br />

[📖 Documentation](#-table-of-contents) · [🚀 Quick Start](#-quick-start) · [🏗️ Architecture](#️-system-architecture) · [⚙️ Configuration](#️-configuration-reference) · [🤝 Contributing](#-contributing)

<br />

---

</div>

## 📋 Table of Contents

- [Overview](#-overview)
- [Key Features](#-key-features)
- [System Architecture](#️-system-architecture)
- [Pipeline Stages](#-pipeline-stages)
- [Technology Stack](#-technology-stack)
- [Prerequisites](#-prerequisites)
- [Quick Start](#-quick-start)
- [Configuration Reference](#️-configuration-reference)
- [API Credentials Setup](#-api-credentials-setup)
- [Workflow Node Reference](#-workflow-node-reference)
- [Prompt Engineering](#-prompt-engineering)
- [Error Handling & Resilience](#-error-handling--resilience)
- [Monitoring & Observability](#-monitoring--observability)
- [Scaling & Multi-Account](#-scaling--multi-account)
- [Cost Analysis](#-cost-analysis)
- [Security Considerations](#-security-considerations)
- [Troubleshooting](#-troubleshooting)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🔭 Overview

**LinkedIn AI Auto-Poster** is a production-grade, autonomous content publishing system that eliminates manual LinkedIn posting entirely. Built on [n8n](https://n8n.io), it orchestrates a multi-model AI pipeline that:

1. Fetches **real-time AI news** from the web via Tavily Search
2. Generates a **professionally written LinkedIn post** using Groq (Llama 3.3 70B)
3. Extracts a **punchy visual headline** using Google Gemini
4. Creates a **custom banner image** via Stability AI
5. **Publishes everything to LinkedIn** — as a personal profile or company page

The system runs on a configurable schedule (default: every 2 days at 16:00) with no human intervention required after initial setup.

> **Business Impact:** Consistent LinkedIn presence drives 3–5x more profile views, inbound leads, and follower growth compared to irregular manual posting — without hiring a social media manager.

---

## ✨ Key Features

| Feature | Description |
|---|---|
| 🕐 **Fully Scheduled** | Cron-based trigger fires every 2 days at 4PM automatically |
| 🌐 **Real-Time Research** | Tavily API fetches live web data — posts are always current, never stale |
| 🧠 **Multi-Model AI** | Groq for speed & writing quality; Gemini for headline refinement |
| 🖼️ **AI Image Generation** | Stability AI v2 creates a custom visual for every post |
| 🏢 **Dual Publishing** | Supports both personal profiles and organization pages |
| 🧹 **Robust Parsing** | Code nodes handle malformed LLM outputs gracefully with fallback logic |
| 🔌 **Modular Design** | Each stage is independently replaceable with alternative APIs |
| 💰 **Cost Efficient** | Full pipeline runs for under $5/month at current API pricing |

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        LinkedIn AI Auto-Poster Pipeline                      │
└─────────────────────────────────────────────────────────────────────────────┘

  ┌──────────────┐
  │   SCHEDULER  │
  │              │
  │  Every 2     │
  │  days @ 4PM  │
  └──────┬───────┘
         │ trigger
         ▼
  ┌──────────────────────────────────────────────────────┐
  │              STAGE 1: CONTENT GENERATION             │
  │                                                      │
  │  ┌─────────────────────┐    ┌──────────────────┐    │
  │  │   AI Agent (n8n)    │◄───│  Groq LLM        │    │
  │  │   Trending Topic    │    │  Llama 3.3 70B   │    │
  │  │   Post Writer       │    └──────────────────┘    │
  │  └──────────┬──────────┘                            │
  │             │  tool call                            │
  │             ▼                                       │
  │  ┌─────────────────────┐                            │
  │  │   Tavily Search     │  ← Real-time web data      │
  │  │   HTTP Tool         │    (AI news, last 24-48h)  │
  │  └─────────────────────┘                            │
  └──────────────────────────┬───────────────────────────┘
                             │ raw JSON output
                             ▼
  ┌──────────────────────────────────────────────────────┐
  │              STAGE 2: OUTPUT PARSING                 │
  │                                                      │
  │  ┌─────────────────────┐                            │
  │  │   Code Node         │  strips ```json wrappers   │
  │  │   (JavaScript)      │  validates & extracts post │
  │  └─────────────────────┘                            │
  └──────────────────────────┬───────────────────────────┘
                             │ clean post text
                             ▼
  ┌──────────────────────────────────────────────────────┐
  │              STAGE 3: HEADLINE EXTRACTION            │
  │                                                      │
  │  ┌─────────────────────┐    ┌──────────────────┐    │
  │  │   AI Agent (n8n)    │◄───│  Google Gemini   │    │
  │  │   Headline          │    │  Chat Model      │    │
  │  │   Generator         │    └──────────────────┘    │
  │  └──────────┬──────────┘                            │
  │             │                                       │
  │             ▼                                       │
  │  ┌─────────────────────┐                            │
  │  │   Code Node 2       │  parses headline JSON      │
  │  │   (JavaScript)      │  with manual fallback      │
  │  └─────────────────────┘                            │
  └──────────────────────────┬───────────────────────────┘
                             │ clean headline string
                             ▼
  ┌──────────────────────────────────────────────────────┐
  │              STAGE 4: IMAGE GENERATION               │
  │                                                      │
  │  ┌─────────────────────┐                            │
  │  │   Stability AI      │  POST /v2beta/stable-image │
  │  │   HTTP Request      │  output_format: webp       │
  │  │                     │  responseFormat: file ⚠️   │
  │  └─────────────────────┘                            │
  └──────────────────────────┬───────────────────────────┘
                             │ binary image file
                             ▼
  ┌──────────────────────────────────────────────────────┐
  │              STAGE 5: LINKEDIN PUBLISH               │
  │                                                      │
  │         ┌──────────────────────────────┐            │
  │         │  LinkedIn Node               │            │
  │         │  shareMediaCategory: IMAGE   │            │
  │         │                              │            │
  │         │  ┌─────────────┐  ┌────────┐│            │
  │         │  │  Personal   │  │  Org   ││            │
  │         │  │  Profile    │  │  Page  ││            │
  │         │  └─────────────┘  └────────┘│            │
  │         └──────────────────────────────┘            │
  └──────────────────────────────────────────────────────┘
```

### Data Flow Diagram

```
Schedule Trigger
      │
      ▼
[Groq + Tavily Agent] ──────── fetches web → writes post (JSON)
      │
      ▼
[Code Node 1] ──────────────── strips markdown, parses JSON, extracts .post
      │
      ├──────────────────────────────────────────────► [LinkedIn Post Text]
      │
      ▼
[Gemini Agent] ─────────────── extracts 8-word billboard headline
      │
      ▼
[Code Node 2] ──────────────── parses headline, fallback regex cleaning
      │
      ▼
[Stability AI] ─────────────── generates WebP banner image (binary)
      │
      ▼
[LinkedIn Node] ────────────── publishes post + image to profile/org page
```

---

## 🔄 Pipeline Stages

### Stage 1 — Content Generation

The core AI agent uses **Groq's Llama 3.3 70B** model for its exceptional speed and writing quality. The agent is equipped with a Tavily web search tool, which it calls autonomously to retrieve the most recent AI developments before drafting the post.

The system prompt enforces strict output constraints:
- Post length: 250–300 words
- Format: Plain text only (no markdown bold — LinkedIn ignores it)
- Structure: Hook → Insight → Business translation → CTA
- Tone: Expert-to-peer, confident, conversational
- Output: Valid JSON with a single `post` key

### Stage 2 — Output Parsing

LLMs frequently wrap JSON in markdown code fences (` ```json ... ``` `). The JavaScript Code node strips these, attempts `JSON.parse()`, and falls back to the raw string if parsing fails — ensuring the pipeline never stalls on formatting inconsistencies.

### Stage 3 — Headline Extraction

A second AI agent, powered by **Google Gemini**, reads the full post and distills it into a single 8-word billboard-style headline. This headline serves as the image generation prompt. Gemini was chosen for this stage due to its strength in condensed creative copywriting tasks.

A second Code node applies regex-based manual cleaning as a safety net in case the LLM returns improperly formatted output.

### Stage 4 — Image Generation

The cleaned headline is sent to **Stability AI's v2 beta** endpoint (`/stable-image/generate/core`) as a multipart form. The response is a binary WebP image file.

> ⚠️ **Critical configuration:** The HTTP Request node's response format **must** be set to `file`. Omitting this causes n8n to attempt JSON parsing of a binary payload, producing a silent failure with no error thrown.

### Stage 5 — LinkedIn Publishing

The LinkedIn node receives both the post text (referenced from Code Node 1) and the binary image (from Stability AI). It publishes with `shareMediaCategory: IMAGE`, supporting both personal profiles and organization pages via OAuth2.

---

## 🛠 Technology Stack

| Component | Technology | Version | Purpose |
|---|---|---|---|
| **Orchestration** | n8n | ≥1.30.0 | Workflow automation engine |
| **LLM — Writing** | Groq / Llama 3.3 70B | Latest | Fast, high-quality post generation |
| **LLM — Extraction** | Google Gemini | 1.5 Flash | Headline distillation |
| **Web Search** | Tavily API | v1 | Real-time content research |
| **Image Generation** | Stability AI | v2 Beta | Banner image creation |
| **Publishing** | LinkedIn API | v2 | Post + media upload |
| **Runtime** | Node.js | ≥18.x | n8n runtime environment |

---

## 📦 Prerequisites

Before deploying this workflow, ensure you have:

- [ ] **n8n instance** — self-hosted or [n8n Cloud](https://n8n.io/cloud). Minimum version: `1.30.0`
- [ ] **Groq API key** — Free tier available at [console.groq.com](https://console.groq.com)
- [ ] **Google Gemini API key** — Available via [Google AI Studio](https://aistudio.google.com)
- [ ] **Tavily API key** — Free tier at [tavily.com](https://tavily.com)
- [ ] **Stability AI API key** — Available at [platform.stability.ai](https://platform.stability.ai)
- [ ] **LinkedIn Developer App** — OAuth2 credentials from [LinkedIn Developer Portal](https://www.linkedin.com/developers/)
- [ ] **LinkedIn Organization ID** (optional) — Required only for company page posting

---

## 🚀 Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/your-org/linkedin-ai-autoposter.git
cd linkedin-ai-autoposter
```

### 2. Import the Workflow into n8n

**Option A — Via n8n UI:**
1. Open your n8n instance
2. Navigate to **Workflows → Import from file**
3. Select `LinkedIn_Post_Auto_tutorial.json`

**Option B — Via n8n CLI:**
```bash
n8n import:workflow --input=./LinkedIn_Post_Auto_tutorial.json
```

### 3. Configure Credentials

Open the imported workflow and configure the following credential slots:

| Node | Credential Type | Where to Create |
|---|---|---|
| Groq Chat Model | `Groq API` | n8n → Credentials → New → Groq |
| Google Gemini Chat Model | `Google PaLM API` | n8n → Credentials → New → Google PaLM |
| HTTP Request (Tavily) | Header Auth (Bearer) | Inline in node parameters |
| HTTP Request (Stability AI) | Header Auth (Bearer) | Inline in node parameters |
| LinkedIn nodes | `LinkedIn OAuth2` | n8n → Credentials → New → LinkedIn |

### 4. Update Node Parameters

In the **LinkedIn "Create a post"** node, update:
```
organization: "YOUR_LINKEDIN_ORG_ID"    # For company page posting
person: "YOUR_LINKEDIN_PROFILE_ID"       # For personal profile posting
```

In the **HTTP Request (Tavily)** node, update the `Authorization` header:
```
Bearer YOUR_TAVILY_API_KEY
```

In the **HTTP Request (Stability AI)** node, update the `authorization` header:
```
Bearer YOUR_STABILITY_API_KEY
```

### 5. Test the Workflow

1. Click **"Test Workflow"** in the n8n editor
2. Monitor each node's output in the execution panel
3. Verify the LinkedIn post appears on your profile/page
4. Confirm the image was attached correctly

### 6. Activate

Once testing passes, toggle the workflow to **Active**. It will now run automatically on schedule.

---

## ⚙️ Configuration Reference

### Schedule Configuration

The Schedule Trigger node controls execution frequency. Modify in the n8n UI or directly in the JSON:

```json
"rule": {
  "interval": [
    {
      "daysInterval": 2,        // Run every N days
      "triggerAtHour": 16       // Hour in 24h format (UTC)
    }
  ]
}
```

**Recommended posting schedules by goal:**

| Goal | Frequency | Time (Local) |
|---|---|---|
| Brand awareness | Every 2 days | 09:00 or 17:00 |
| Thought leadership | Daily | 07:00 |
| Lead generation | 3x per week | 10:00 |
| Community engagement | 5x per week | 08:00 |

### Customizing the Content Niche

Edit the Tavily search query in the **HTTP Request** tool node to target any content niche:

```json
{
  "query": "latest trend and news in the field of AI that can not be overlooked."
}
```

**Example alternatives:**
```json
{ "query": "top B2B SaaS product launches and funding rounds this week" }
{ "query": "latest breakthroughs in climate tech and green energy" }
{ "query": "most important developments in cybersecurity last 48 hours" }
{ "query": "trending topics in digital marketing and growth hacking" }
```

### Adjusting Post Tone & Style

Modify the system prompt in the **Trending Topic Post** agent node. Key parameters to tune:

```
Tone options: Professional / Conversational / Bold & Opinionated / Educational
Word count: 150–300 words (shorter = more engagement on mobile)
CTA style: Question / Statement / Poll prompt / Comment bait
Hashtag count: 3–5 recommended for LinkedIn algorithm
```

---

## 🔑 API Credentials Setup

### Groq API

1. Visit [console.groq.com](https://console.groq.com) and sign in
2. Navigate to **API Keys → Create API Key**
3. Copy the key and add it to n8n under **Credentials → Groq API**
4. Free tier provides ~14,400 requests/day — sufficient for this workflow

### Google Gemini (PaLM API)

1. Visit [Google AI Studio](https://aistudio.google.com)
2. Click **Get API key → Create API key in new project**
3. Add to n8n under **Credentials → Google PaLM API**

### Tavily Search API

1. Register at [tavily.com](https://tavily.com)
2. Navigate to **Dashboard → API Keys**
3. Copy your key and paste it into the HTTP Request node's `Authorization` header as `Bearer YOUR_KEY`

### Stability AI

1. Sign up at [platform.stability.ai](https://platform.stability.ai)
2. Navigate to **Account → API Keys → Create Key**
3. Paste into the Stability AI HTTP Request node's `authorization` header

### LinkedIn OAuth2

1. Go to [LinkedIn Developer Portal](https://www.linkedin.com/developers/apps)
2. Create a new app with the following OAuth 2.0 scopes:
   - `w_member_social` — Post as a person
   - `w_organization_social` — Post as an organization
   - `r_basicprofile` — Read profile info
3. Set redirect URI to: `https://YOUR_N8N_INSTANCE/rest/oauth2-credential/callback`
4. Create credentials in n8n under **Credentials → LinkedIn OAuth2 API**
5. Authenticate via the OAuth flow in n8n

**Finding your LinkedIn Organization ID:**

Navigate to your company page on LinkedIn. The URL will contain the numeric ID:
```
https://www.linkedin.com/company/110742135/
                                  ─────────
                                  This is your Organization ID
```

---

## 📘 Workflow Node Reference

| Node | Type | Role | Key Parameters |
|---|---|---|---|
| `Schedule Trigger` | Trigger | Fires the pipeline on a cron schedule | `daysInterval: 2`, `triggerAtHour: 16` |
| `Trending Topic Post` | AI Agent | Writes the LinkedIn post using web-sourced context | Groq LLM, Tavily tool, custom system prompt |
| `HTTP Request (Tavily)` | HTTP Tool | Searches the web for fresh AI news | POST to Tavily API, connected as AI tool |
| `Code` | Code (JS) | Parses and extracts post text from LLM output | JSON parse with markdown strip + fallback |
| `Image Generation` | AI Agent | Extracts a billboard-style headline from the post | Gemini LLM, max 8 words, no punctuation |
| `Google Gemini Chat Model1` | LLM | Language model for headline agent | Default options, PaLM API credentials |
| `Code1` | Code (JS) | Parses and cleans headline from LLM output | JSON parse + regex fallback cleaning |
| `HTTP Request1` | HTTP | Generates WebP image via Stability AI | Multipart form, `responseFormat: file` ⚠️ |
| `Create a post` | LinkedIn | Publishes to organization page | `shareMediaCategory: IMAGE`, org ID |
| `Create a post1` | LinkedIn | Publishes to personal profile | `shareMediaCategory: IMAGE`, person ID |
| `Groq Chat Model` | LLM | Language model for writing agent | Llama 3.3 70B, Groq credentials |

---

## 🧠 Prompt Engineering

### Post Generation Prompt (Stage 1)

The prompt instructs the model to act as an expert social media strategist. Core constraints enforced:

```
✅ Under 250–300 words
✅ Plain text only (no markdown **bold**)
✅ 3–4 hashtags
✅ Max 2 emojis (hook or CTA only)
✅ Strong fact/claim hook on line 1
✅ Business translation included
✅ Ends with a CTA
✅ No em-dashes
✅ Output: { "post": "..." } JSON
```

### Headline Extraction Prompt (Stage 3)

Designed for maximum visual impact:

```
✅ Maximum 8 words
✅ Active voice, strong verbs
✅ Billboard / magazine cover style
✅ No hashtags, emojis, or punctuation
✅ Output: { "headline": "..." } JSON
```

**Example outputs:**

| Post Topic | Generated Headline |
|---|---|
| GPT-5 efficiency improvements | AI Cuts Enterprise Costs By Half |
| Agentic AI adoption surge | Autonomous Agents Are Taking Over Workflows |
| Open-source model milestone | Open Source Just Beat The Big Labs |

---

## 🛡️ Error Handling & Resilience

### Parsing Fallback Strategy

Both Code nodes implement a two-tier fallback to prevent pipeline failures:

```javascript
// Tier 1: Standard JSON parse
try {
  const parsed = JSON.parse(cleaned);
  result = parsed.post || parsed.headline || "";
} catch (e) {
  // Tier 2: Manual regex cleaning
  result = cleaned
    .replace(/^{|}$/g, '')
    .replace(/"post"|"headline":/gi, '')
    .replace(/[*{}"]/g, '')
    .trim();
}
```

### Common Failure Points & Mitigations

| Failure Point | Symptom | Mitigation |
|---|---|---|
| Stability AI response format | Empty binary output, no error | Set `responseFormat: file` in HTTP node options |
| LLM markdown wrapping | JSON parse error in Code node | Regex strip before parse (implemented) |
| Tavily rate limit | Empty search results | Upgrade plan or add retry logic |
| LinkedIn token expiry | 401 on publish | Re-authenticate OAuth2 credentials in n8n |
| Gemini quota exceeded | Agent node error | Switch to alternative LLM (e.g., GPT-4o-mini) |

### Adding a Retry Layer (Recommended for Production)

Wrap HTTP Request nodes with n8n's built-in retry settings:
1. Open the node settings
2. Enable **"On Error: Retry on fail"**
3. Set **Max tries: 3**, **Wait between tries: 5000ms**

### Adding Human-in-the-Loop Review (Optional)

Insert a **Slack** or **Email** node between the `Code` node and the `Image Generation` node to review posts before publishing:

```
[Code Node] → [Send to Slack for approval] → [Wait for approval] → [Image Generation] → [LinkedIn]
```

---

## 📊 Monitoring & Observability

### n8n Execution Logs

All workflow runs are logged in n8n's **Executions** panel. For each run, inspect:
- Node-level input/output
- Execution duration per node
- Error stack traces on failure

### Recommended Alerting Setup

Configure **Error Workflow** in n8n settings to notify on failure:

1. Create a separate "Alert" workflow triggered by `Workflow Error`
2. Connect a Slack or email notification node
3. Set it as the global error workflow under **Settings → Error Workflow**

### Key Metrics to Track

| Metric | How to Track | Target |
|---|---|---|
| Pipeline success rate | n8n execution history | >95% |
| Post generation time | Node execution duration | <30s |
| Image generation time | HTTP Request1 duration | <15s |
| LinkedIn publish success | LinkedIn node output | 100% |
| API cost per run | External API dashboards | <$0.08/run |

---

## 📈 Scaling & Multi-Account

### Running for Multiple LinkedIn Profiles

Duplicate the workflow and change the `person` or `organization` parameter in the LinkedIn nodes. Each instance can target a different niche, language, or account.

### Multi-Tenant Architecture

For agencies managing multiple clients:

```
Master Scheduler
       │
       ├──► Client A Workflow (AI/Tech niche, EN)
       ├──► Client B Workflow (Marketing niche, EN)
       ├──► Client C Workflow (Finance niche, AR)
       └──► Client D Workflow (HR niche, DE)
```

Each client workflow can have its own:
- Topic query
- Post language and tone
- Posting schedule
- LinkedIn credentials

### Environment-Based Configuration

For team deployments, use n8n environment variables to manage API keys centrally:

```bash
# .env (n8n instance)
GROQ_API_KEY=your_key_here
GEMINI_API_KEY=your_key_here
TAVILY_API_KEY=your_key_here
STABILITY_API_KEY=your_key_here
```

Reference in n8n nodes using `{{ $env.GROQ_API_KEY }}`.

---

## 💰 Cost Analysis

Estimated cost per workflow execution (as of 2025):

| Service | Model / Plan | Cost per Run | Monthly (15 runs) |
|---|---|---|---|
| Groq | Llama 3.3 70B (Free tier) | $0.00 | $0.00 |
| Google Gemini | Gemini 1.5 Flash (Free tier) | $0.00 | $0.00 |
| Tavily | Free tier (1,000 searches/mo) | $0.00 | $0.00 |
| Stability AI | Core (pay-per-image) | ~$0.03 | ~$0.45 |
| LinkedIn API | Free | $0.00 | $0.00 |
| n8n Cloud | Starter plan | Fixed | ~$20/mo |
| **Total** | | **~$0.03/run** | **~$20.50/mo** |

> All free tiers are subject to provider limits. For high-volume or commercial use, review each provider's paid pricing.

---

## 🔐 Security Considerations

### API Key Management

- **Never commit API keys** to version control. The workflow JSON in this repository has all credentials removed.
- Use n8n's built-in credential store — keys are encrypted at rest.
- For self-hosted n8n, set `N8N_ENCRYPTION_KEY` in your environment for additional security.

### LinkedIn OAuth2 Token Security

- OAuth2 tokens are stored in n8n's credential vault
- Tokens expire periodically — monitor for `401` errors from LinkedIn nodes
- Rotate credentials immediately if a breach is suspected

### Network Security

- Run n8n behind a reverse proxy (nginx/Caddy) with HTTPS
- Restrict n8n admin access via IP allowlist
- Use environment variables for all sensitive configuration

### Credential Rotation Checklist

```bash
# Run this checklist quarterly
□ Rotate Groq API key
□ Rotate Gemini API key
□ Rotate Tavily API key
□ Rotate Stability AI API key
□ Re-authenticate LinkedIn OAuth2
□ Audit n8n execution logs for anomalies
```

---

## 🔧 Troubleshooting

### Pipeline produces no image / empty binary

**Cause:** Stability AI HTTP node response format not set to `file`.

**Fix:** In the `HTTP Request1` node → Options → Response → Response Format → select **File**.

### LinkedIn post publishes without image

**Cause:** Binary data not correctly passed to LinkedIn node.

**Fix:** Ensure `HTTP Request1` immediately precedes the LinkedIn node with no intermediate transformation nodes that might lose the binary context.

### Code node throws JSON parse error

**Cause:** LLM returned output wrapped in markdown code fences or with extra text.

**Fix:** The fallback logic handles this, but if it persists, log `$json.output` and inspect the raw string. Update the regex pattern accordingly.

### Tavily returns empty results

**Cause:** API key expired, quota exceeded, or network issue.

**Fix:**
1. Test the key directly via `curl`:
```bash
curl -X POST https://api.tavily.com/search \
  -H "Authorization: Bearer YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "AI news today"}'
```
2. If `401`, regenerate the key on the Tavily dashboard.

### LinkedIn OAuth2 authentication fails

**Cause:** Token expired or scope mismatch.

**Fix:**
1. Open n8n **Credentials → LinkedIn OAuth2**
2. Click **Reconnect** and re-authenticate
3. Ensure your LinkedIn app has `w_member_social` and `w_organization_social` scopes enabled

---

## 🤝 Contributing

Contributions are welcome and encouraged. Please follow this process:

### Development Setup

```bash
# Fork and clone the repo
git clone https://github.com/your-username/linkedin-ai-autoposter.git

# Create a feature branch
git checkout -b feature/your-feature-name

# Make changes to the workflow JSON or documentation

# Test in your own n8n instance before submitting
```

### Pull Request Guidelines

- All PRs must include a description of what changed and why
- Workflow JSON changes must be tested end-to-end
- Documentation must be updated if behavior changes
- Remove all personal API keys and credentials before committing

### Areas Open for Contribution

- [ ] Support for Twitter/X cross-posting
- [ ] Slack approval step (human-in-the-loop)
- [ ] Multi-language post generation
- [ ] Google Sheets logging of published posts
- [ ] A/B testing prompt variants
- [ ] Webhook trigger as alternative to schedule
- [ ] Notion content calendar integration

### Reporting Issues

Please use [GitHub Issues](https://github.com/your-org/linkedin-ai-autoposter/issues) with the following template:

```
**Describe the bug**
A clear description of the issue.

**Node where failure occurs**
Which n8n node fails and what the error output is.

**Expected behavior**
What should have happened.

**n8n version**
Output of `n8n --version`
```

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

You are free to use, modify, and distribute this workflow for personal or commercial purposes. Attribution appreciated but not required.

---

<div align="center">

**Built with ❤️ using n8n, Groq, Gemini, Stability AI & LinkedIn API**

⭐ If this project saves you time, please star the repository — it helps others discover it.

[🐛 Report Bug](https://github.com/your-org/linkedin-ai-autoposter/issues) · [💡 Request Feature](https://github.com/your-org/linkedin-ai-autoposter/issues) · [📧 Contact](mailto:your@email.com)

</div>
