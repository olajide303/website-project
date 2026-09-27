# MEGAMIND AI ASSISTANT - IMPLEMENTATION PLAN

Prepared: 2026-09-27
Repository: `github.com/olajide303/website-project`
Status: Phase 0 (foundation) - no application code written yet

---

## 1. WHERE THE PROJECT ACTUALLY IS

### What exists right now

| Item | Detail |
|---|---|
| Files in repo | 1 file: `MEGAMIND A.I ASSISTANT.md` (5,132 bytes) |
| Git history | 1 commit: `6e8eb15 first commit - megamind ai assistant` |
| Branch | `main`, tracking `origin/main`, working tree clean |
| Remote | `git@github.com:olajide303/website-project.git` (SSH) |
| Auth | SSH key `My Dell Laptop` - verified working, no password needed |

### What is missing

| Area | Status |
|---|---|
| Application code | None. No HTML, CSS, JS, no backend |
| `package.json` / any project manifest | Missing |
| `.gitignore` | Missing (will break once `node_modules` exists) |
| `README.md` | Missing |
| `.env` / secrets handling | Missing |
| Database | None |
| Tests / CI | None |
| Hosting | Not chosen |
| AI provider key | Not obtained |
| Meta / WhatsApp app | Not created (steps 1-3 of setup guide not confirmed done) |

### Tooling audit on this PC

| Tool | Installed? | Note |
|---|---|---|
| Git | Yes | Working, SSH auth verified |
| Node.js | **No** | Required for Phase 1 - must install |
| npm | **No** | Comes with Node.js |
| Python | **No** | Not needed if we use Node.js |
| GitHub CLI (`gh`) | **No** | Optional, nice for PRs/releases later |
| Docker Desktop | Unknown | Optional, useful if self-hosting |

**Recommendation:** Install Node.js LTS (v22 or newer) and GitHub Desktop. Skip Python entirely - one language keeps this simple for a small business. Install Docker only if you go the VPS route in Phase 2.

---

## 2. PRODUCT REQUIREMENTS (from `MEGAMIND A.I ASSISTANT.md`)

Confirmed with you in the original planning conversation:

| Requirement | Source | Phase |
|---|---|---|
| Reply on WhatsApp | `:14` | 2 |
| Reply on website chat | `:14` | 3 |
| Reply on Facebook | `:14` | 5 |
| Reply on X (Twitter) | `:38` | 5 |
| Answer questions (FAQ) | `:23` | 1 |
| Take orders | `:23` | 6 |
| Track delivery / order status | `:23` | 7 |
| Human-in-the-loop approval before any order is final | `:38`, `:44-51` | 4 |
| One shared "brain", separate channel connectors | `:18` | 1 |
| Owner runs it on their phone | `:38` | 4 |
| Runs while owner is unavailable | `:38` | 1 |

### The 5-step customer flow (from `:44-51`)

1. Customer messages on any channel while you are away
2. Assistant replies instantly: greets, answers questions, collects order details
3. Assistant notifies you: *"Customer [name] wants to order [X]. Approve?"*
4. You tap Yes / No / Edit from your phone
5. Only then does the assistant confirm the order to the customer

**This flow is the heart of the product. Nothing in this plan may break it.**

---

## 3. ARCHITECTURE

### The core idea (already agreed in `:18`)

One brain, many channel connectors. Never build three separate bots.

```
                 ┌─────────────────────────────┐
   WhatsApp ────▶│                             │
   Website   ────▶│      MEGAMIND BRAIN        │──▶ Answer sent back
   Facebook  ────▶│  (intent -> retrieve ->     │    on same channel
   X         ────▶│   compose -> act)          │
                 │                             │
                 └──────────┬──────────────────┘
                            │ needs approval?
                            ▼
                   ┌─────────────────┐
                   │  APPROVAL QUEUE │
                   │  (draft orders) │
                   └────────┬────────┘
                            │ notify
                            ▼
                   ┌─────────────────┐
                   │  YOUR PHONE     │
                   │ Approve/Edit/No │
                   └─────────────────┘
```

### Key architectural decisions

| # | Decision | Choice | Why |
|---|---|---|---|
| 1 | Language | **TypeScript on Node.js 22+** | Single language for backend + frontend. Largest AI SDK support. Cheap to host. |
| 2 | Web framework | **Fastify** | Fast, small, built-in schema validation. |
| 3 | Database | **SQLite** (`better-sqlite3`) | Zero-config, single file, perfect for one small business. Can upgrade to Postgres later without rewriting queries. |
| 4 | AI provider | **Anthropic Claude API** (Sonnet for replies, Haiku for cheap classification) | Already know it. Best instruction-following for customer tone. Haiku keeps per-message cost tiny. |
| 5 | Hosting | **Cheap Linux VPS + Docker** | Webhooks need a machine that is on 24/7. Your Windows PC sleeps, loses power, and Nigeria has outages. |
| 6 | Customer-facing website | **Next.js** (one project, API + widget) | Serves the public site and the chat API from the same app. |
| 7 | Your approval interface | **Telegram bot (primary) + PWA dashboard (secondary)** | Telegram is free, installs in seconds, buttons work everywhere, pushes are instant. Dashboard gives you history and editing. |
| 8 | Knowledge source | **Versioned JSON + Markdown in the repo** | Prices and policies live in git, so every change is reviewed and reversible. |
| 9 | Secrets | **`.env` locally, secrets manager on the server** | Never commit API keys. |
| 10 | Monorepo | **npm workspaces** (`apps/web`, `apps/brain`, `packages/shared`, `packages/knowledge`) | Brain stays independent of any channel. |

### Why not "just run it on my PC"

Be aware of this tradeoff, because it is the most common early mistake:

| | Your PC | Small VPS |
|---|---|---|
| Cost | Free | ~$4-6/month (estimate) |
| Uptime | Poor (sleep, shutdown, power cuts) | ~99.9% |
| Public address | Needs tunneling, breaks often | Fixed IP |
| Setup effort | Low | Moderate |
| **Verdict** | Dev/testing only | **Production** |

Recommended path: develop and test on your PC, deploy to a VPS when WhatsApp goes live in Phase 2.

---

## 4. PHASED IMPLEMENTATION PLAN

Each phase ends with something you can **see or test**. Do not start a phase before the previous one is verified.

---

## PHASE 0 - FOUNDATION AND ACCOUNTS
**Goal:** Install tools, make every account decision, gather the business information the AI needs. **No code yet.**
**Time:** 1-3 days, mostly waiting and typing.
**Cost:** $0

### Tasks

| # | Task | Detail |
|---|---|---|
| 0.1 | Install Node.js LTS | nodejs.org, LTS build, accept defaults. Verify with `node --version` |
| 0.2 | Install GitHub Desktop | desktop.github.com - optional, gives you drag-and-drop commits |
| 0.3 | Install Docker Desktop | Only if you will deploy in Phase 2 |
| 0.4 | Create Anthropic API account | console.anthropic.com, add credits, create API key |
| 0.5 | Create Meta Developer account | developers.facebook.com (steps 1-3 of your setup guide) |
| 0.6 | Create Meta Business App | Add WhatsApp product, note the Phone Number ID and token |
| 0.7 | Buy / confirm dedicated SIM | Must not be your personal WhatsApp number |
| 0.8 | Register WhatsApp Business app | On the dedicated SIM |
| 0.9 | Choose a VPS | Hetzner (cheapest, EU), or DigitalOcean/Linode. Ubuntu 24.04, 1 vCPU / 1GB RAM is enough to start |
| 0.10 | Register a domain | `megamind.com.ng` or similar. Optional in Phase 0, needed by Phase 3 |
| 0.11 | Create Telegram account | Needed for approval notifications in Phase 4 |
| 0.12 | **Write the business knowledge file** | See Section 5 below. **This is the single most important input to the whole project.** |
| 0.13 | Create `.gitignore` | Blocks `node_modules`, `.env`, build output |
| 0.14 | Create `README.md` | What the project is, how to run it, where docs live |
| 0.13b | Create `.env.example` | Every variable name, empty values, comments |
| 0.15 | Set up branch protection | Settings → Branches → protect `main`, require PR |

### Deliverables
- [ ] `node --version` prints v22 or higher
- [ ] `MEGAMIND KNOWLEDGE.md` written and committed
- [ ] `.gitignore`, `README.md`, `.env.example` committed
- [ ] Meta app created, phone number linked, test message sent from sandbox
- [ ] Anthropic API key created and working

### Verify
Run a single test prompt against the Anthropic API from a scratch file. If you get a text reply, your key and internet are fine.

---

## PHASE 1 - THE BRAIN (no channels yet)
**Goal:** Build the core engine that decides what to say, and test it entirely on your PC.
**Time:** 4-7 days
**Cost:** ~$1-3 in API credits (estimate)

### Tasks

| # | Task | Detail |
|---|---|---|
| 1.1 | Scaffold monorepo | npm workspaces: `apps/brain`, `apps/web`, `packages/shared`, `packages/knowledge` |
| 1.2 | Build `Message` type | `channel`, `customerId`, `customerName`, `text`, `timestamp`, `attachments` |
| 1.3 | Build conversation store | SQLite tables: `conversations`, `messages`. Index by customer |
| 1.4 | Build the knowledge loader | Reads `packages/knowledge/*.json` and `*.md` into memory at boot |
| 1.5 | Build intent classifier | Haiku model, fixed label set: `GREETING`, `FAQ`, `ORDER_START`, `ORDER_STATUS`, `COMPLAINT`, `HUMAN_REQUEST`, `UNKNOWN` |
| 1.6 | Build FAQ responder | Retrieves matching knowledge, composes reply with Sonnet, **always** cites that it is AI if asked |
| 1.7 | Build the system prompt | Megamind's tone, spelling (Nigerian English is fine), what it must never promise, escalation rule |
| 1.8 | Build the order state machine | States: `DRAFT → COLLECTING → PENDING_APPROVAL → CONFIRMED / REJECTED / EXPIRED`. Enforce illegal transitions |
| 1.9 | Build a CLI test harness | `npm run chat` opens a terminal you can type customer messages into. **This is how you test everything in Phase 1** |
| 1.10 | Conversation memory | Last N messages to the model, plus structured facts already collected |
| 1.11 | Safety rails | Never invent prices or delivery dates. If unknown, collect details and flag for owner |
| 1.12 | Structured logging | JSON logs per turn: intent, tokens used, latency, cost. Needed for Phase 7 cost control |

### Deliverables
- [ ] `npm run chat` lets you have a full conversation about Megamind products
- [ ] Correctly refuses to invent a price it does not know
- [ ] Order flow reaches `PENDING_APPROVAL` and stops
- [ ] 30+ scripted test conversations pass

### Verify
Write 30 realistic customer messages (greetings, price questions, complaints, "let me speak to a human", order requests, typos, Nigerian Pidgin). Feed each through the CLI. Record where it fails. Fix, re-test.

### Recommendation
Do not skip the CLI harness. Without it you are testing through WhatsApp, which is slow, rate-limited, and makes debugging painful. The CLI makes iteration 10x faster.

---

## PHASE 2 - WHATSAPP GOES LIVE
**Goal:** First real customers, first real messages.
**Time:** 3-5 days
**Cost:** VPS + Meta conversation charges

### Tasks

| # | Task | Detail |
|---|---|---|
| 2.1 | Build WhatsApp connector | Verify webhook signature, dedupe by message ID, mark messages read |
| 2.2 | Handle the 24-hour session window | Free-text messages expire after 24h. Send approved templates outside it |
| 2.3 | Rate limiting | Meta allows ~80 messages/second. Queue outgoing sends, never fire in parallel |
| 2.4 | Deploy brain to VPS | Docker Compose, HTTPS with Caddy (auto TLS), systemd restart on crash |
| 2.5 | Point Meta webhook to VPS | Public URL, verify token, subscribe to `messages` field |
| 2.6 | Deploy with Cloudflare Tunnel as fallback | Replaces public IP if the VPS ever has DNS issues |
| 2.7 | Swap sandbox number for real SIM | Only after everything is verified in sandbox |
| 2.8 | Build template messages | Order received, order approved, order ready, out for delivery, delivered |
| 2.9 | Dead-man's-switch | If brain is down 5 minutes, alert you on Telegram |
| 2.10 | Daily backup job | `pg_dump`-equivalent (SQLite file copy) to offsite storage, keep 30 days |

### Deliverables
- [ ] Real WhatsApp number replies in under 3 seconds
- [ ] Messages survive a brain restart without duplication
- [ ] Customer never sees a raw error message

### Verify
Have a friend message the number 20 times from a different phone. Check every reply is correct, nothing is duplicated, nothing is lost.

### Recommendation
Keep the WhatsApp Business app logged in on your phone as a manual override. If the brain is down or says something wrong, you can always type to the customer yourself. Never give away your human fallback.

---

## PHASE 3 - WEBSITE AND CHAT WIDGET
**Goal:** Customers who find you on the web get the same brain.
**Time:** 5-8 days
**Cost:** domain + hosting

### Tasks

| # | Task | Detail |
|---|---|---|
| 3.1 | Build Next.js site | Home, About, Services/Products, Contact. Mobile-first, fast, Nigerian-friendly |
| 3.2 | Build the chat widget | Floating button, expandable panel, message history, typing indicator |
| 3.3 | Reuse the same conversation logic | Widget talks to the same `/api/chat` endpoint the brain uses. **Do not write a second brain** |
| 3.4 | Visitor identity | Anonymous session ID, then captured on order or by asking name |
| 3.5 | Hand off to WhatsApp | Button that moves a website conversation into WhatsApp, carrying context |
| 3.6 | Add SEO basics | Meta tags, Open Graph, sitemap, `robots.txt`, Google verification |
| 3.7 | Analytics | Privacy-respecting, page views and chat-start rate |
| 3.8 | Accessibility and low bandwidth | Works on 2G/3G. Compress images, lazy-load. Customers are on mobile data |

### Deliverables
- [ ] Public site live on own domain with HTTPS
- [ ] Chat widget answers an FAQ correctly
- [ ] Widget conversation continues seamlessly into WhatsApp

### Verify
Open the site on your phone on mobile data, not WiFi. It must feel fast. If it does not, customers leave.

---

## PHASE 4 - THE APPROVAL LOOP (human-in-the-loop)
**Goal:** Orders are drafted and confirmed by you, from your phone. This is the phase that makes it trustworthy.
**Time:** 5-7 days
**Cost:** $0 (Telegram is free)

### Tasks

| # | Task | Detail |
|---|---|---|
| 4.1 | Build the approval queue | Table `pending_actions`: type, payload, status, timestamps, customer ref |
| 4.2 | Telegram notification | Send order summary + inline `Approve` / `Edit` / `Reject` buttons |
| 4.3 | Handle button callbacks | Idempotent. Double-tap must not create two orders |
| 4.4 | Implement `Edit` | You correct items/address/phone, it re-asks the customer to confirm the corrected version |
| 4.5 | Timeout handling | 12h no response → auto-expire and politely tell the customer you will follow up |
| 4.6 | Quiet hours | No notifications 22:00-07:00, except if you set an override |
| 4.7 | Build the PWA dashboard | Approve queue, full conversation history, search customers, edit knowledge base |
| 4.8 | Audit log | Every approval/rejection with who, when, what changed. Protects you in disputes |
| 4.9 | Kill switch | One Telegram command stops all AI replies, hands everything back to you |
| 4.10 | "Talk to a human" intent | Hands off immediately, tells customer you will reply personally |

### Deliverables
- [ ] Order is drafted, sent to you, you tap Approve, customer gets confirmation
- [ ] Edit flow works and customer sees the corrected order
- [ ] Kill switch instantly stops the AI
- [ ] You can review any past conversation from your phone

### Verify
Run 20 fake orders end-to-end including rejection, edit, timeout, and double-tap. Every outcome must be correct. A duplicate order in production is a real business loss.

### Recommendation
Treat the kill switch as the most important feature in this phase. Build it first, test it, then build the rest. It is your insurance policy and it takes under an hour.

---

## PHASE 5 - FACEBOOK MESSENGER AND X
**Goal:** Full channel coverage as promised in `:14`.
**Time:** 4-6 days
**Cost:** free per message on current Meta and X free tiers

### Tasks

| # | Task | Detail |
|---|---|---|
| 5.1 | Build Messenger connector | Same pattern as WhatsApp, plus 24h messaging window and tag policies |
| 5.2 | Connect the Facebook Page | Link the business Page, enable the messenger API |
| 5.3 | Build X connector | Direct messages require an approved developer account and elevated access. **Apply early - approval can take weeks** |
| 5.4 | Channel adapter interface | Refactor to a common `Connector` contract: `send`, `parse`, `verifyWebhook`, `markRead` |
| 5.5 | Per-channel tone profiles | X is shorter and more casual. Same brain, different voice. |
| 5.6 | Unified conversation view | Dashboard shows all channels per customer, with identity linked by phone number |

### Deliverables
- [ ] Facebook Page DMs answered by the brain
- [ ] X DMs answered by the brain, or documented as blocked pending approval
- [ ] One customer with accounts on all channels is treated as one person

---

## PHASE 6 - ORDER TAKING
**Goal:** Collect real orders end-to-end, with human approval.
**Time:** 5-8 days
**Cost:** $0 incremental

### Tasks

| # | Task | Detail |
|---|---|---|
| 6.1 | Order schema | Items, quantity, unit price, total, delivery address, phone, notes, preferred time |
| 6.2 | Interactive list messages | WhatsApp product list + quick replies. Much better conversion than free text |
| 6.3 | Item collection flow | Add, remove, change quantity, with a running total shown after each step |
| 6.4 | Address handling | Text address for now. Add location pin support in 6.6 |
| 6.5 | Validate and price | Totals computed by code, never by the model. This prevents wrong prices |
| 6.6 | Location and payment options | Share-location button, COD vs transfer, transfer details from the knowledge base |
| 6.7 | Export | CSV and PDF for your records. Even if you do not automate fulfillment yet |
| 6.8 | Order numbering | `MM-YYYY-####`, searchable, unique forever |

### Deliverables
- [ ] Customer completes an order purely through chat
- [ ] Every price on the summary matches the knowledge base exactly
- [ ] You approve from Telegram and get a clean order record

### Recommendation
This is where AI discipline matters most: **the model collects, code calculates.** Never let the model do arithmetic or decide a price. One mispriced order loses customer trust permanently.

---

## PHASE 7 - DELIVERY TRACKING AND OPERATIONS
**Goal:** Customers can ask "where is my order?" and get a real answer, and the system runs itself safely.
**Time:** 5-8 days
**Cost:** monitoring ~$0-10/month (estimate)

### Tasks

| # | Task | Detail |
|---|---|---|
| 7.1 | Order status model | `PENDING → CONFIRMED → PREPARING → DISPATCHED → DELIVERED`, plus `CANCELLED` |
| 7.2 | Owner status controls | Update status in the dashboard or by Telegram command |
| 7.3 | Status notifications | Push to customer on every status change |
| 7.4 | Proactive check-in | "How is your order?" 24h after dispatch |
| 7.5 | Cost dashboard | Tokens and API cost per day, per channel, per customer. Alert on spikes |
| 7.6 | Uptime monitoring | External monitor, not self-reported. Page you if down |
| 7.7 | Backups and restore test | **Actually test restoring.** An untested backup is not a backup |
| 7.8 | Failover plan | Written down: what happens if the VPS dies, if your phone is lost, if your SIM dies |
| 7.9 | Compliance and privacy | Data retention policy, what is stored, how to delete a customer's data on request, NDPR (Nigeria's data protection law) basics |
| 7.10 | Secrets rotation schedule | Rotate API keys and tokens quarterly |

### Deliverables
- [ ] Customer asks for status, gets the correct real status in one message
- [ ] Daily cost is known and within budget
- [ ] A restore from backup is demonstrated successfully
- [ ] Written failover plan exists

---

## 5. THE KNOWLEDGE FILE (do this in Phase 0)

This is the AI's entire understanding of your business. Everything in Section 4 depends on it being complete and accurate.

Create `MEGAMIND KNOWLEDGE.md` covering:

1. **What Megamind does** - plain language, 3 sentences
2. **Products or services** - name, description, exact price, unit, availability
3. **How ordering works** - steps the customer goes through
4. **Delivery** - areas covered, fees, delivery time, what is not deliverable
5. **Payment** - accepted methods, account details, payment deadlines
6. **Returns and refunds** - policy in full
7. **Opening hours** - days and times, timezone (WAT, UTC+1)
8. **Contact** - phone, email, address, social handles
9. **Tone and values** - how you want to sound. Nigerian English? Formal? Friendly?
10. **Hard rules** - things the AI must never say. Examples: never promise a delivery time, never offer a discount, never give health or legal advice
11. **Escalation** - when to hand to a human immediately
12. **Common questions** - the 20 questions customers ask most, with approved answers
13. **Do not say** - words, phrases, or promises to avoid

Put machine-readable pricing in `packages/knowledge/catalog.json` so code can calculate totals, and keep prose in Markdown for the AI to read.

---

## 6. TIMELINE AND COST SUMMARY

Estimates only. Actual depends partly on external approvals.

| Phase | Duration | Running cost | Cumulative |
|---|---|---|---|
| 0 Foundation | 1-3 days | $0 | $0 |
| 1 Brain | 4-7 days | $1-3 | ~$3 |
| 2 WhatsApp live | 3-5 days | ~$5-10/mo | ~$13 |
| 3 Website + widget | 5-8 days | ~$2-5/mo | ~$18 |
| 4 Approval loop | 5-7 days | $0 | ~$18 |
| 5 Facebook + X | 4-6 days | $0 | ~$18 |
| 6 Order taking | 5-8 days | $0 | ~$18 |
| 7 Tracking and ops | 5-8 days | ~$0-10/mo | ~$25/month |
| **Total build** | **~5-9 weeks** | | |

Meta charges per conversation on WhatsApp. Check current Meta pricing, as it changes. This is usually the largest ongoing cost of the whole system, so monitor it in Phase 7.

### What you get at the end

A system that answers customer questions instantly on WhatsApp, your website, Facebook, and X, drafts orders while you are away, asks you for one tap of approval from your phone, tracks deliveries, and never makes a promise you did not approve.

---

## 7. RISKS AND HOW WE HANDLE THEM

| Risk | Impact | Mitigation |
|---|---|---|
| AI invents a price or promise | High - loses customer trust | Knowledge base is the only source. Code does arithmetic. Approval gate before anything binding |
| Meta rejects or delays the app | High - blocks Phase 2 | Apply in Phase 0. Sandbox first, so delays are cheap |
| X API access denied | Medium - one channel missing | Apply in Phase 5 early. Ship without X if needed |
| VPS or network fails in Nigeria | High - assistant goes silent | Dead-man's-switch alert, WhatsApp Business app as manual fallback, failover plan in 7.8 |
| Power outage at your house | Medium | System lives on the VPS, not your PC. Local backups of orders |
| Customer data leak | High - legal and reputational | Secrets never in git, least-privilege tokens, retention policy, NDPR awareness |
| API cost surprise | Medium | Per-turn cost logging from Phase 1, daily dashboard and alerts in Phase 7 |
| Bot sends a damaging message | High | Kill switch, tone testing, escalation rules, approval on anything binding |
| You stop maintaining it | Medium | Simple stack, written runbook, one-command deploy and rollback |

---

## 8. WHAT TO DO RIGHT NOW

**Immediate, in this order:**

1. Install Node.js LTS, verify with `node --version`
2. Create the Anthropic API account and key
3. Create the Meta Developer account and Business App, link the WhatsApp number, send one sandbox test message
4. Write `MEGAMIND KNOWLEDGE.md` using Section 5
5. Create `.gitignore`, `README.md`, `.env.example` and commit them
6. Then we start Phase 1 and build the brain

**One decision needed from you before Phase 1:** which VPS provider you want to use, or whether to develop locally first and choose hosting at the end of Phase 1. My recommendation: develop locally, choose hosting when we start Phase 2. It keeps Phase 0 free and lets the architecture settle before you spend money.

---

## 9. DOCUMENT MAP

| Document | Purpose |
|---|---|
| `MEGAMIND A.I ASSISTANT.md` | Original planning conversation and requirements source |
| `IMPLEMENTATION PLAN.md` | This file. Phases, tasks, costs, risks |
| `MEGAMIND KNOWLEDGE.md` | To be created in Phase 0. The AI's business knowledge |
| `README.md` | To be created in Phase 0. How to run the project |
| `CHANGELOG.md` | To be created in Phase 0. What changed per release |
| `docs/ARCHITECTURE.md` | To be created in Phase 1. Diagrams and data flow |
| `docs/RUNBOOK.md` | To be created in Phase 2. Deploy, backup, restore, incident steps |
