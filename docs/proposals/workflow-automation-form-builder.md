# FlowScout — AI Workflow Intake & Automation Blueprint Builder

*A James Networks subsite proposal (e.g. `flowscout.jamesnetworks.net`)*

## The idea in one sentence

An AI-guided intake experience that interviews a business owner about one of
their workflows the way a seasoned automation consultant would, then hands back
an **Automation Blueprint**: a plain-English plan for turning that process into
an automation, including which middleware platform to use (n8n, Rewst, Zapier,
Make, Power Automate, …) and why.

## Why this fits James Networks

- It *is* the "Do It With You" model in software form. The free blueprint is
  the first half of the conversation; the paid engagement ("let's build this
  together") is the natural second half.
- It's a lead-generation engine: every completed intake is a qualified lead who
  has already described their pain point, their systems, and their volume.
- It showcases the 30-years-of-IT-ops credibility — the tool asks the questions
  an operator would ask, not a generic form.

## How it works (user experience)

### Phase 1 — The AI Interview (adaptive form)

Instead of a static 40-field form nobody finishes, the intake is a short seed
form plus an AI-driven conversation:

1. **Seed form** (static, 4 fields): name/email, industry, company size, and a
   free-text "describe the process that eats your time."
2. **Adaptive follow-ups**: the AI reads the description and asks only the
   follow-ups that matter, 1–3 at a time, rendered as real form controls
   (chips, dropdowns, sliders — not a wall of chat). Typical dimensions it
   probes:
   - **Trigger** — what starts the process? (email arrives, form submitted,
     invoice received, ticket opened, schedule)
   - **Systems** — what tools touch it? (QuickBooks, Outlook/Gmail, Excel,
     a CRM, a PSA/RMM if they're an MSP…)
   - **Volume & frequency** — how many times a day/week, how long per run
   - **Decision points** — where does a human judge/approve something?
   - **Exceptions** — what goes wrong, how often
   - **Data sensitivity** — PII, payment data, HIPAA, etc.
   - **Team & tech comfort** — who maintains this, can anyone read JSON?
   - **Budget posture** — per-task pricing OK, or flat/self-hosted preferred?
3. The interview caps at ~8–12 questions. A progress meter and a running
   "what we've understood so far" sidebar keep trust high.

### Phase 2 — The Automation Blueprint (the deliverable)

When the interview completes, the AI synthesizes everything into a rendered,
shareable blueprint page (plus PDF/email copy):

1. **Process map** — the workflow as they described it, drawn as a diagram
   (Mermaid), with the automatable steps highlighted.
2. **Automation verdict** — what's fully automatable, what needs
   human-in-the-loop, what should stay manual, stated honestly.
3. **Platform recommendation** — a ranked pick from the middleware matrix
   (below) with the *why*: cost at their volume, fit with their stack, skill
   required to maintain it.
4. **Build plan** — numbered steps mapped to the recommended platform's
   vocabulary (e.g. "n8n: Webhook node → IF node → QuickBooks node" or
   "Zapier: Gmail trigger → Formatter → QuickBooks action").
5. **ROI estimate** — hours/week reclaimed × their volume answers, and rough
   platform cost at that volume.
6. **Gotchas & prerequisites** — API access needed, plan tiers required,
   rate limits, data-sensitivity flags.
7. **CTA** — "Want it built? We'll do it with you." → booking link.

## The middleware knowledge layer (the moat)

The AI isn't asked to recall platforms from memory — it's grounded with a
maintained, structured **capability matrix** injected into the synthesis
prompt. Curating this matrix is where the domain expertise lives:

| Platform | Sweet spot | Pricing model | Skill floor | Notes |
|---|---|---|---|---|
| **Zapier** | Non-technical teams, SaaS-to-SaaS glue | Per-task, gets pricey at volume | Lowest | Largest connector library (~7,000+ apps) |
| **Make** | Visual builders, branching logic, mid budgets | Per-operation, cheaper than Zapier at volume | Low-mid | Strong scenario visualizer |
| **n8n** | Technical teams, high volume, self-host/data control | Flat self-hosted or cloud tiers — cheap at scale | Mid-high | Code nodes, AI-agent nodes, on-prem option |
| **Rewst** | MSPs specifically | MSP subscription | Mid | Deep PSA/RMM hooks (ConnectWise, Datto, Halo, Microsoft CSP); ROC prebuilt crates |
| **Power Automate** | Microsoft 365-centric shops | Bundled w/ M365 + premium connectors | Low-mid | Wins by default when the stack is all-Microsoft |
| **Pipedream / Tines / Workato** | Dev-heavy, security, or enterprise niches | Varies | Varies | Mentioned when the fit is right |

Recommendation logic keys off interview answers: M365-heavy → Power Automate
first; MSP with a PSA → Rewst; high volume + any technical comfort → n8n;
zero technical staff + modest volume → Zapier/Make; sensitive data or
data-residency concerns → self-hosted n8n.

The matrix lives as versioned JSON/YAML in the repo so updating platform facts
(pricing, connectors) is a content edit, not a code change.

## Architecture (right-sized MVP)

- **Frontend**: a static single-page app on the subsite, same visual language
  as jamesnetworks.net (dark ink + gold). No framework needed at MVP — or a
  light Preact/Vue if preferred. Renders the seed form, the adaptive question
  cards, and the final blueprint (with Mermaid for diagrams).
- **Backend**: one serverless function (Cloudflare Workers / Netlify
  Functions) that proxies to the Claude API — keeps the API key off the
  client and enforces rate limits.
- **AI**: Claude with structured outputs / tool use in two roles:
  1. *Interviewer* — given transcript so far, returns the next question(s) as
     JSON (question text, control type, options) or `interview_complete`.
  2. *Synthesizer* — given the full transcript + capability matrix, returns
     the blueprint as structured JSON the frontend renders.
- **Capture & delivery**: email required before the full blueprint unlocks
  (summary visible free). Blueprint emailed as PDF; lead lands in a simple
  store (Airtable/Sheet/DB) with the whole transcript attached — gold for the
  follow-up sales conversation.

## Roadmap

- **Phase 1 (MVP)**: seed form → adaptive interview → on-page blueprint +
  email capture. One serverless function, no accounts, no database beyond
  lead storage.
- **Phase 2**: PDF export, booking-link integration, admin view of leads and
  transcripts, analytics on where people drop off.
- **Phase 3**: *starter exports* — generate an importable n8n workflow JSON
  skeleton or a Zapier step outline directly from the blueprint. This is the
  showstopper demo: "we didn't just tell you how — here's the file."
- **Phase 4**: saved blueprints/accounts, multi-workflow "automation roadmap"
  for a whole business, white-label mode for MSP partners.

## Name candidates

- **FlowScout** — scouts your workflow, reports back with a plan (working title)
- **BlueprintAI / The Automation Blueprint** — leans on the deliverable
- **FlowForge** — leans on the build phase

## Risks & mitigations

- **AI over-promises feasibility** → synthesis prompt requires an honest
  "stays manual" section; matrix grounds claims about connectors/pricing.
- **API cost abuse on a public form** → rate limiting per IP/email at the
  serverless layer; interview capped at ~12 turns.
- **Platform facts go stale** → matrix is versioned content with a review
  cadence; blueprint footer dates its recommendations.
