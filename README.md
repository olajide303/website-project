# MEGAMIND AI ASSISTANT

An AI customer representative for **Megamind**. It answers customer questions, takes
orders, and tracks deliveries on WhatsApp, the website, Facebook, and X — while you are
away. Nothing is finalised without your approval.

> **Human-in-the-loop by design.** The assistant replies instantly and drafts orders,
> but every order waits for your one-tap approval from your phone before the customer
> is told it is confirmed.

---

## Status

**Phase 0 — Foundation.** No application code yet.

| Phase | Scope | Status |
|---|---|---|
| 0 | Foundation and accounts | In progress |
| 1 | The brain (core engine) | Not started |
| 2 | WhatsApp live | Not started |
| 3 | Website and chat widget | Not started |
| 4 | Approval loop | Not started |
| 5 | Facebook Messenger and X | Not started |
| 6 | Order taking | Not started |
| 7 | Delivery tracking and operations | Not started |

See [`IMPLEMENTATION PLAN.md`](./IMPLEMENTATION%20PLAN.md) for full task lists, costs,
and timelines.

---

## Requirements

- [Node.js](https://nodejs.org) 22 or newer
- [Git](https://git-scm.com) with SSH access to GitHub (already configured)
- An Anthropic API key
- A Meta Developer account with a WhatsApp Business number (from Phase 2 onward)
- A Telegram account (from Phase 4 onward)

## Setup

```bash
git clone git@github.com:olajide303/website-project.git
cd website-project
npm install
```

Copy `.env.example` to `.env` and fill in your keys. Never commit `.env`.

## Project layout

```
website project/
├─ IMPLEMENTATION PLAN.md   Phased build plan, costs, risks
├─ MEGAMIND A.I ASSISTANT.md Original requirements conversation
├─ MEGAMIND KNOWLEDGE.md    The assistant's business knowledge (Phase 0)
├─ README.md                This file
├─ CHANGELOG.md             What changed per release
├─ apps/
│  ├─ brain/                Core engine: intents, retrieval, replies
│  └─ web/                  Next.js site + chat widget
├─ packages/
│  ├─ shared/               Types shared across apps
│  └─ knowledge/            catalog.json + business docs
└─ docs/                    Architecture and runbook
```

## Design principles

1. **One brain, many channels.** Channel connectors are thin adapters. Never write a
   second brain.
2. **The model collects, code calculates.** Prices and totals are computed in code from
   the catalog. The model never does arithmetic or invents a price.
3. **Nothing binding without approval.** Orders, refunds, and promises queue for you.
4. **A kill switch always exists.** One command hands every conversation back to you.
5. **The knowledge base is the single source of truth.** If it is not in
   `MEGAMIND KNOWLEDGE.md` or `packages/knowledge/catalog.json`, the assistant says it
   does not know and escalates.

## Documentation

| Document | Purpose |
|---|---|
| [`IMPLEMENTATION PLAN.md`](./IMPLEMENTATION%20PLAN.md) | Phases, tasks, costs, risks |
| [`MEGAMIND A.I ASSISTANT.md`](./MEGAMIND%20A.I%20ASSISTANT.md) | Original requirements |
| `MEGAMIND KNOWLEDGE.md` | Business knowledge — create in Phase 0 |
| `docs/ARCHITECTURE.md` | Data flow — Phase 1 |
| `docs/RUNBOOK.md` | Deploy, backup, restore — Phase 2 |

## Contributing

Work on a branch, open a pull request, get it reviewed, then merge to `main`.
`main` is protected.

## License

Private. All rights reserved.
