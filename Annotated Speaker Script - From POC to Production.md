# From POC to Production — Annotated Speaker Script

**NWA Tech Summit · 2026 · ~40 minutes + Q&A**

Each section below shows **what the audience sees on screen** first, then **what you say**. Rehearse straight down the page — eye goes from slide box to spoken text, same order as the real talk.

- ⏱ = time budget for that slide
- 🎯 = the one line that must land
- **[ASK]** = audience interaction · **[EXHIBIT]** = pause and let the visual land
- Running long? Slides marked ✂️ compress cleanly; see the compression guide at the end.

---

## Slide 1 — Title ⏱ ~2 min

> ### ON SCREEN
> **FROM POC TO PRODUCTION**
> *Engineering Talk · Northwest Arkansas Tech Community · 2026*
>
> Agentic AI has been demo-ready since 2023. Production-ready is an engineering property — orchestration, context, retries, caches, evals.
>
> Chips: Multi-agent orchestration · Context engineering · Reliability & loops · Cost control · Eval-driven delivery

Hello everyone — thanks for making time for this. Quick show of hands before we start: how many of you have seen an AI agent demo that was genuinely impressive? Keep your hand up if that same system is handling real production traffic today.

*(pause — let hands drop)*

Yeah. That gap between those two moments is what we're here to talk about.

Here's my promise for the next forty minutes: this is an **engineering talk**, not a market talk. I'm not selling anything, there's no vendor pitch, and every number has a source you can check — the last slide is nothing but citations. We're going to walk through why agentic AI has been demo-ready since 2023 but *production-ready* is still something you have to engineer — orchestration, context handling, reliability math, cost control, delivery discipline.

By the end you'll have eight concrete things to do Monday morning. Let's go.

---

## Slide 2 — The models got better. The outcomes didn't follow. ⏱ ~4 min

> ### ON SCREEN
> SWE-bench Verified curve: **~33% (Aug 2024) → 80.9% (Jan 2026)** — Opus 4.5 first over 80%.
>
> | Stat | |
> |---|---|
> | **~95%** | enterprise GenAI pilots, no measurable P&L impact, $30–40B invested *(MIT NANDA)* |
> | **>40%** | agentic projects predicted canceled by end-2027 *(Gartner)* |
> | **−19%** | measured productivity for experienced OSS maintainers — while self-reporting +20% faster *(METR RCT)* |
>
> ⚑ Day-one move: instrument end-to-end success rate + cost per completed task before scaling.

🎯 *"Capability stopped being the bottleneck in 2024. Where outcomes stay flat is exactly where systems discipline is missing."*

Let's start with the tension that motivates this whole talk.

Look at this curve. SWE-bench Verified — real GitHub issues resolved end-to-end. August 2024: around thirty-three percent. January 2026: **eighty point nine**. Capability crossing bar after internal bar, year over year.

Now look at what happened to outcomes:

- MIT's NANDA group: roughly **95% of enterprise GenAI pilots showed no measurable P&L impact**, despite thirty to forty billion dollars invested.
- Gartner predicts **more than 40% of agentic-AI projects get canceled** by end of 2027.
- And my favorite uncomfortable one: METR ran a randomized controlled trial with experienced open-source maintainers. With AI they **felt about 20% faster — and measured 19% slower.** The perception gap IS the PoC-to-prod gap in miniature.

Steelman beat — say it before someone else does. METR's null is one study of sixteen experts on familiar code; Copilot, BCG, and writing RCTs show real gains for non-experts. MIT's methodology is contested too. The claim isn't "AI doesn't help" — it's "**helping individuals ≠ shipping reliable systems.**"

Why do outcomes stall? Four things the demo never had to survive: **context handling, reliability, cost, delivery discipline.** Everything after this slide is about those four.

Day-one move: instrument end-to-end success rate and cost per completed task **before** you scale. If you can't name those two numbers, you have a demo, not a system.

---

## Slide 3 — Two incidents every builder should know by heart ✂️ ⏱ ~5 min

> ### ON SCREEN
> **Case 01 — "The agent ate the database" · Replit × SaaStr · Jul 2025**
> Code freeze acknowledged → violated anyway ("it panicked"); wiped prod DB (~1,200 client companies); confronted with failing tests, fabricated 3,605 fake users + fake pass reports; CEO: "unacceptable"; fixes in days — dev/prod isolation by default, one-click restore, emergency off-switch.
>
> **Case 02 — "The chatbot invented a policy" · Air Canada tribunal · May 2024**
> Chatbot invented a bereavement-fare policy; passenger booked, refund refused; airline claimed bot was "a separate legal entity"; tribunal rejected it — CA$650.88.
>
> Monday morning: ① least-privilege creds for agents ② idempotency keys + dry-run gates on mutating tools ③ immutable audit log + tested restore ④ kill switch owned by on-call

🎯 *Regulators and courts treat agent output as company speech.*

Theory is cheap. Two stories — one engineering failure, one legal one.

**Case one: Replit and SaaStr, July 2025.** A founder used Replit's AI agent to build a database app. During an explicit code freeze, the agent said it understood "no more changes" — then ran anyway, described itself as "panicking," and **wiped the production database covering roughly twelve hundred client companies.** When confronted with failing tests afterward, it didn't fail honestly — it **fabricated around three thousand six hundred fake users and fake passing test reports** to cover the failure. CEO called it unacceptable, and fixes shipped within days: dev/prod isolation by default, one-click restore, an emergency off-switch.

I'll scope this honestly if anyone presses: weakest-case proxy — a non-engineer pointing a consumer tool at production with zero credential separation. It earns its place here for exactly two lessons: **credential isolation and kill switches.** And hold onto the shape: failing tests → fabrication → building on the lie. We come back to it later.

**Case two: Air Canada, May 2024.** A passenger asked the website chatbot about bereavement fares. The chatbot confidently described a policy that **did not exist.** Passenger booked on that advice; airline refused the refund. Air Canada's defense — I'm not joking — was that the chatbot was **"a separate legal entity"** responsible for its own statements. The BC tribunal rejected that and awarded CA$650.88. Small number, lasting precedent.

The takeaway from both: **courts and regulators now treat agent output as company speech.** "The model said it" is not a defense — it never was for any other piece of your software.

Neither was a model-capability failure. Both were prevented by boring controls: least-privilege credentials for agents, idempotency keys plus dry-run gates on anything mutating, immutable audit log with a *tested* restore, kill switch owned by on-call rather than a support ticket.

**[ASK]** Who here has separate credentials for your agent versus your own deploy account?

---

## Slide 4 — Klarna, fifteen months apart ✂️ ⏱ ~3–4 min

> ### ON SCREEN
> | FEB 2024 · SCALE-UP | MAY 2025 · WALK-BACK |
> |---|---|
> | 2.3M conversations month one — ⅔ of support chats; work of 700 full-time agents; resolution 11 min → <2 min; −25% repeat inquiries; projected $40M profit improvement | CEO Siemiatkowski: *"We were so cost-focused… what you end up having is lower quality."* Rehiring human phone support |
>
> Honest footnote: headcount fell ~44% over a different window; quality attribution contested.
> `score = w₁·resolution + w₂·CSAT + w₃·cost + w₄·escalation` — weights set by humans, in code.
>
> ⚑ Day-one move: define the plug-pulling metric set before launch; wire into dashboards.

🎯 *Reported value is a function of what you chose to measure.*

This story isn't a failure story — that's exactly why I like it.

**February 2024:** Klarna's AI assistant handles **2.3 million conversations in its first month** — two-thirds of support chats, the work of 700 full-time agents, resolution time from eleven minutes to under two across 35-plus languages, repeat inquiries down a quarter, projected **$40 million profit improvement**. Massive win.

**May 2025, fifteen months later,** the same CEO says — quote — *"we were so cost-focused… what you end up having is lower quality."* And they started rehiring human phone support.

Read both quotes carefully — neither is insincere. In 2024 the numbers were real as reported. In 2025 they'd lived with it long enough to see what wasn't captured. Honest footnote: total headcount fell about 44% over a different window, quality attribution is contested — but the $40M was a projection, never publicly validated. **The reported value reflected what they chose to measure.**

That formula on screen is the lesson in one line: score equals w-one times resolution, plus w-two times CSAT, plus w-three times cost, plus w-four times escalation. Every weight set by a human, in code. Weight cost heavily and quality lightly, you get exactly the system you asked for — and fifteen months later you discover what the weights left out.

Day-one move: **before launch, write down the metric set that would convince YOU to pull the plug** — then wire those exact metrics into dashboards before traffic does.

---

## Slide 5 — The multi-agent question — argued live in public ⏱ ~5 min

> ### ON SCREEN
> | EVIDENCE FOR — Anthropic · Jun 2025 | EVIDENCE AGAINST — Cognition · Jun 2025 |
> |---|---|
> | Orchestrator + subagents scored **90.2% higher** than single-agent Opus (internal research eval; self-reported). Parallel tool calls cut research time up to 90%. Works because subtasks are READ-ONLY | "Don't build multi-agents": share full context; actions carry implicit decisions; parallel writers compound conflicting decisions. Claude Code sub-agents work only because read-only |
>
> Stat strip: `90.2%` gain · `~15×` tokens vs chat · `80%` of variance = token usage alone
> Shared ground: start simple · earn complexity · LangChain deep-agents as pragmatic middle

🎯 **Decision rule: fan out only for read-only or independently verifiable breadth. Write paths stay single-threaded with full context. Subagents return compressed findings — never transcripts.**

Now let's get technical. Should you build multi-agent systems? There's a genuine public disagreement between two labs, both from June 2025, staged side by side.

**Evidence FOR — Anthropic.** Their multi-agent research system: an orchestrator spawning parallel subagents. It scored **90.2% higher** than single-agent Opus on their internal research eval. Parallel tool calls cut research time up to ninety percent. Why does it work? Because the subtasks were **read-only** — gather evidence, return compressed findings, no conflicting writes.

Fair caveats: self-reported, methodology unpublished, they sell the tokens it consumes, and it burned roughly **fifteen times the tokens of a chat.**

**Evidence AGAINST — Cognition.** Walden Yan's post "Don't Build Multi-Agents." Two principles: **share full context** — every part of the system needs the complete picture; and **actions carry implicit decisions** — parallel writers make conflicting implicit decisions that compound into bad results.

And Cognition's own concession proves the mechanism: Claude Code uses subagents successfully *because* they're read-only. Both sides agree on why it works. They disagree on where to draw the line.

One more number for the skeptics: Anthropic attributed **80% of performance variance to token usage alone.** Sometimes "more agents" just means "more compute."

So the decision rule, and if you remember one thing from this section make it this: **fan out only for read-only or independently verifiable breadth. Write paths single-threaded, full context. Subagents return compressed findings, never transcripts.**

---

## Slide 6 — Delegation patterns — matched to task shape ⏱ ~5 min

> ### ON SCREEN
> Quad diagram:
> - **SUPERVISOR** (dynamic decomposition) — fails: bottleneck/context bloat · fix: turn/token budgets, compressed returns
> - **PIPELINE** (known DAG — most production value) — fails: drift poisons downstream · fix: typed contracts, retry-from-checkpoint, dead-letter queue
> - **BLACKBOARD** (parallel read-only research) — fails: write races/duplication · fix: additive versioned writes, leases, quorum
> - **DEBATE** (verification by contention) — fails: sycophancy collapse, 3× cost/2× latency · fix: adversarial roles, capped rounds, human escalation
>
> Delegation contract: deadline + token budget per delegation · structured summaries back · mutating steps never fan out · one owner of the final answer

[EXHIBIT] Walk the quad clockwise. Tie back to the slide-5 debate.

If you do delegate to multiple agents, four shapes — and each has a named production failure mode, because they all break characteristically.

**Supervisor** — dynamic decomposition. The orchestrator decides who does what, on the fly. Fails via **bottleneck and context bloat**: everything routes through one agent whose window fills with everyone else's transcripts. Fix: hard turn and token budgets, compressed returns.

**Pipeline** — known DAG. Steps known in advance, typed artifacts passed down a chain. **Most production value lives here**, precisely because known steps are testable steps. Fails via **drift poisoning downstream**: garbage early poisons everything late. Fix: typed contracts, retry-from-checkpoint, dead-letter queues.

**Blackboard** — parallel read-only research. Multiple agents attack corners of a question simultaneously. Fails via **write races and duplication** — two agents solving the same subproblem twice, no completion signal. Fix: additive versioned writes, leases, explicit quorum.

**Debate** — verification by contention. Agents argue opposing positions. Fails via **sycophancy collapse** — agreeing too easily — at three-times cost and double latency per decision. Fix: genuinely adversarial roles, capped rounds, human escalation.

Notice supervisor and blackboard are safe at scale exactly when workers are **read-only** — Anthropic's finding meets Cognition's caveat.

Under all four sits the **delegation contract**: every delegation gets a deadline, a token budget, structured summaries back. Mutating steps never fan out. Someone always owns the final answer.

**[ASK]** Which failure mode have you already lived? *(take 1–2 answers, protect timing)*

---

## Slide 7 — The cheapest latency win is remembering correctly ⏱ ~5 min

> ### ON SCREEN
> KV-cache exhibit: same 20-turn session, two layouts.
> Stable prefix → **93% hit rate** → ~$0.20/session ≈ 6× cheaper
> Volatile head → **33% hit rate** → ~$0.86/session — **same session, 4× the bill**
> Rules: ① stable top-of-prompt block ② volatile data at tail ③ append-only history ④ mask don't remove tools
> Cached: $0.30 vs $3 per MTok (~10×)

⚑ Day-one move: cache-hit rate on dashboard next to error rate.

New section: context engineering. Starting with money.

Manus published their context-engineering learnings last July. Headline stat: their agent runs roughly **100 input tokens per output token** — input dwarfs output. That makes one metric disproportionately important, and Manus calls cache-hit rate **"the single most important metric"** for an agent company.

Here's the experiment: same twenty-turn conversation, two prompt layouts.

Stable layout — big immutable block up top, system prompt, tool definitions, docs, then append-only history, volatile stuff pushed to the tail: **93% hit rate**, about twenty cents a session.

Volatile head — a timestamp near the top invalidating everything above it every turn: hits collapse to **33%**, about eighty-six cents. Same session, same model, **four times the bill, purely from prompt layout.** Cached tokens bill around ten times cheaper and serve faster, so p50 latency drops too.

Four rules: one, stable top-of-prompt block. Two, volatile data — timestamps, session IDs — at the tail, never the head. Three, append-only history; edits invalidate everything after them. Four, mask unavailable tools via logits rather than removing them — removal shifts bytes and busts the prefix.

Day-one move: put **cache-hit rate next to error rate on the dashboard**. A falling hit rate is the earliest signal of prompt drift — and of money burning quietly.

---

## Slide 8 — When the window fills: four field-tested moves ✂️ ⏱ ~4–5 min

> ### ON SCREEN
> **MOVE 1 · Compact** — compress, don't truncate; regenerate state from artifacts. *In practice:* drop oldest turns → keep todo + key facts → rebuild from files
> **MOVE 2 · Note-take** — memory outside the window (NOTES.md). *In practice:* write-once-per-milestone · notes survive compaction · failures kept deliberately
> **MOVE 3 · Recite** — keep the goal on top; rewrite todo.md every few turns. *In practice:* goal stays near attention peak
> **MOVE 4 · Quarantine** — subagents as airlocks; fresh-context worker reads the haystack, returns only the conclusion. *In practice:* read-only workers · compressed findings back
>
> ⚑ Day-one move: context-budget check per turn — tokens-in trending up warns of drift, loops, runaway cost

Context windows feel infinite until they aren't. Long-horizon agents die of **context rot** long before they die of model limits — attention is a finite budget. Four field-tested moves when it fills:

**Compact.** Summarize old turns and regenerate state from artifacts. Naive truncation loses constraints mid-task — the model forgets the rules.

**Note-take.** Persist findings outside the window — the NOTES.md pattern. Environment feedback, milestones, failures to avoid, re-readable regardless of conversation length. Claude Code ships this today.

**Recite.** Rewrite the objective into recent context every few turns. Fights lost-in-the-middle; Manus credits it for coherence across fifty-tool-call tasks.

**Quarantine.** Fresh-context subagent reads the haystack and returns only the conclusion — main thread pays for the answer, not the reading. Notice: that's the read-only fan-out rule again, now earning its keep.

Day-one move: add a **context-budget check per turn**. Tokens-in trending upward is the earliest warning of drift, loops, and runaway cost — one number catching three failure families.

✂️ *(Short on time: name the four moves + the banner, skip "in practice" lines — 90 seconds.)*

---

## Slide 9 — Reliability compounds — and decays ⏱ ~5 min

> ### ON SCREEN
> `P = rᴺ`
> r=0.99, N=10 → **90%** · r=0.95, N=10 → **60%** · r=0.90, N=10 → **35%** · r=0.95 at N=20 → **36%**
> Buy the exponent back: ① retry w/ backoff+jitter (r′=1−(1−r)²) ② fallback/degraded path ③ circuit breaker + timeout ④ idempotency keys on mutations
> Dead zone: ≥97% earns autonomy · ≤90% keeps reviewer on every run · 90–97% partial autonomy w/ sampled review

[EXHIBIT] Let the curves land before talking.

Reliability section. Core idea fits in one formula: for an N-step chain at per-step reliability r, end-to-end success is **P equals r to the N**.

At ten steps: ninety-nine percent per step gives ninety percent end-to-end. Ninety-five percent gives sixty. Ninety percent gives thirty-five. And here's the kicker — same ninety-five percent but twenty steps instead of ten: **thirty-six percent. Depth hurts as much as per-step quality.** This is why demos die in production: demos are shallow chains, production is deep ones.

But the exponent isn't destiny. You buy it back:

One — **retries with backoff and jitter.** One retry takes 95% to 99.75%. Cheap exponent recovery. Two — **fallbacks and degraded paths**: safe partial answers instead of dying. Three — **circuit breakers and timeouts**: stop cascades, stop hangs. Four — **idempotency keys on mutations** — without them, a retry IS a double-delete.

Fine print, because engineers deserve fine print: retries assume independent failures — correlated failures laugh at this math, cap and jitter them. Fallbacks assume a safe partial answer exists, which means designing one ahead of time.

Where should you aim? **Ninety-seven percent and up earns autonomy. Ninety and below keeps a reviewer on every run. The 90–97 dead zone — where most teams live for months — is partial autonomy with sampled review**, sample rate set by blast radius, tightened after every incident.

**[ASK]** What's your longest agent chain in prod today? What's its per-step rate?

---

## Slide 10 — Hallucination loops: watching drift happen ✂️ ⏱ ~5–6 min

> ### ON SCREEN
> Four drift trajectories: unguarded / grounding / validation / reflection
> Loop: ① small factual error → ② plan built on error → ③ tool results read as confirmation → ④ confidence ↑ divergence compounds
> Structural breaks: grounding gate (fails closed) · validation (schema/sanity checks) · reflection (critic pass) · keep failures in context
> Callout: "Replit was this loop"
> Disclaimer: illustrative runs, not benchmark data — shape is the finding

Open by recalling the Replit cover-up — this loop at incident scale.

Remember the fabricated users, the fake test reports? This slide is that story drawn as a system diagram. The loop has four steps. One: a small factual error enters the context. Two: the plan gets built on top of it. Three: tool results get read as confirmation. Four: confidence rises while divergence compounds. **Nobody decided to be wrong. The loop did it.**

Watch the trajectories. Unguarded drift compounds geometrically and crosses into confabulation by turn six. Grounding — claims must cite retrieved context — flattens immediately. Validation bends the curve back. Reflection catches late drift but costs a second pass. One honest note: these are illustrative runs, not benchmark data — **the shape is the finding.**

Three structural breaks: a **grounding gate** where claims must cite context and *fail closed* — no citation means blocked or flagged, not warn-and-continue. **Validation** — schema and sanity checks on tool outputs at the boundary, not downstream. **Reflection** — a critic pass; selective because it costs latency.

And one counterintuitive practice: **keep failures in context deliberately.** Don't scrub the error away — the agent should see that step twelve failed so it doesn't rebuild the same broken path. Scrubbed errors invite repetition.

Callout says it plainly: **Replit was this loop** — failing tests, fabricate users, build on the lie. A grounding gate fails closed at step one and stops the cascade. Loops are fixed **structurally** — gates and schemas — not with sternly-worded prompts.

**[ASK]** Where in YOUR pipeline could an error survive three turns unnoticed?

---

## Slide 11 — Tokens are your quality dial AND your burn rate ⏱ ~5 min

> ### ON SCREEN
> Three levers, ROI order:
> **1 · Route** — classifier routes by complexity; ~60–80% blended savings reported (directional, measure your own mix); routing in config not code. Watch: misroute rate
> **2 · Cache & reuse** — prefix/KV caching first (~10× input price), then semantic caches. Watch: hit rate
> **3 · Govern** — quotas, per-run budget caps, admission control, batch windows; kill switch = load-bearing infrastructure. Watch: token burn rate
>
> Banner: capacity is a product decision — turns × tokens × price × traffic × hit-rate reviewed like an SLO

Last technical section: cost. Frame with Anthropic's numbers: multi-agent research ran **~15× chat tokens**, and token usage alone explained **80% of performance variance**. Tokens are simultaneously your quality dial and your burn rate — governance decides which one you're turning. Three levers, ROI order.

**Route.** Most traffic doesn't need your biggest model. A small classifier sends extraction and formatting to fast models, escalates planning and synthesis. Practitioners report **sixty to eighty percent blended savings at equal quality** — flagging clearly: that's directional folklore, not a sourced benchmark. Measure your own mix. Keep routing decisions in config, not code, so you tune against evals without deploys. Watch misroute rate.

**Cache and reuse.** Prefix/KV caching first — free discipline, Manus measured ~10× on input price — then semantic response caches for repeated intents. Remember, multi-turn agents pay for the same system prompt thousands of times. Watch hit rate.

**Govern.** Per-team quotas, per-run budget caps enforced by the orchestrator, admission control under load, batch windows. Without governance, one runaway loop can consume a fleet budget overnight — and the kill switch, which we met in the Replit story, is **load-bearing infrastructure**, not a nice-to-have. Watch token burn rate.

Banner: **capacity is a product decision.** Ship-cost model — expected turns, tokens per turn, price, traffic, cache-hit-rate — reviewed like any other SLO, not opened after the invoice scares you.

**[ASK]** Who owns inference cost in your org today? *(pause)* Silence is the finding.

---

## Slide 12 — The production control plane ⏱ ~5 min

> ### ON SCREEN
> Nine-layer grid: 01 Gateway & AuthN/Z · 02 Orchestrator · 03 Agent Fleet · 04 Tool Layer · 05 Memory & State · 06 Observability · 07 Eval Harness · 08 Deploy & Rollback · 09 Guardrails
> Left rail: request journey · incidents→layers (Replit = creds+guardrails+restore; Air Canada = guardrails+eval) · build order: ① observability ② tool contracts ③ eval harness before autonomy ④ deploy/rollback last, before scale

🎯 Take-home artifact — tell people to screenshot it.

Let's assemble everything into one picture. Nine layers — the control plane of a production agent system. Each layer answers a question a PoC never had to ask. Walking the grid as a request journey:

**Gateway and auth** — rate limits, quotas, identity. Question answered: *who is allowed to spend money?*

**Orchestrator** — planning, retries, timeouts, budgets. *Who enforces all these contracts we keep writing?*

**Agent fleet** — specialized workers, sandboxed executors. *Where does the actual work run safely?*

**Tool layer** — typed contracts, circuit breakers, idempotency keys. *What happens when the outside world misbehaves?*

**Memory and state** — vector stores, session state, durable checkpoints. *How does the system survive restarts and share context?*

**Observability** — traces per span, token and cost metrics, replay. *When it breaks at 2am, how do we know what happened?*

**Eval harness** — golden sets, LLM judges, regression gates. *Is tomorrow's change better than today's?*

**Deploy and rollback** — canary percentage, shadow mode, instant revert. *How do we ship without betting the service?*

**Guardrails** — PII filters, policy engine, human-in-the-loop queue. *What stops the worst day?*

Left rail insight: map our incidents onto layers. Replit was missing tool creds, guardrails, restore. Air Canada was missing guardrails and evals. **Incidents cluster where layers are absent.**

Build order beats full stack: observability first, then tool contracts, then eval harness — *before* autonomy. Deploy/rollback last, but before scale. **Screenshot this one — it's the take-home artifact.**

---

## Slide 13 — Testing what refuses to be deterministic ✂️ ⏱ ~4 min

> ### ON SCREEN
> Four-tier stack:
> TIER 1 deterministic — contract tests (schema-valid, required fields, bounded args, no PII) — CI gate
> TIER 2 model-judged — golden-set scoring by rubric; judges correlate once calibrated
> TIER 3 statistical — simulation & canaries vs staging world; auto-rollback on breach
> TIER 4 human — expert review queues; feedback flows back into golden sets
>
> Rule: *"If a change isn't covered by an eval, it isn't a change — it's a coin flip."* No diff, no merge.
> Anti-pattern: "we tried it on ten examples and it seemed fine."

"How do you test something nondeterministic?" — the most common question I get. Answer: in tiers, matching test type to what can be checked mechanically. Key reframe: **you cannot unit-test creativity — test the contract, not the prose.**

**Tier one, deterministic contract tests.** Schema-valid outputs, required fields present, args bounded, no PII leaks. Pure code, seconds to run, blocks merges in CI. Teams skip this because it feels basic — it's the highest-ROI tier you have.

**Tier two, model-judged golden sets.** Curated pairs scored by rubric — factuality, groundedness, tone. Judges correlate with humans *once calibrated against human labels.* Known biases: verbosity preference, self-preference. Calibrate first, trust after.

**Tier three, statistical.** Synthetic journeys against a staging world, live canary slices with automatic rollback on guardrail breach.

**Tier four, human.** Sampled audits and escalation queues — and crucially, feedback flows back into golden sets, so the eval suite learns from the reviewers instead of just consuming them.

The rule that changes behavior: **"If a change isn't covered by an eval, it isn't a change — it's a coin flip."** Every prompt edit, tool swap, and model upgrade lands with a diff of eval scores attached. No diff, no merge.

Anti-pattern, named out loud: *"We tried the new model on ten examples and it seemed fine."* Seemed fine is how agents reach production unevaluated. Replit's agent had no eval gate at all. Sample sizes and score deltas — not vibes.

And connect it back: the MIT 95% stat is largely an **eval-discipline stat** — pilots that couldn't demonstrate value were usually pilots that never measured it.

---

## Slide 14 — Ship like it's software — because it is ✂️ ⏱ ~4 min

> ### ON SCREEN
> Promotion path: 01 Offline evals (gate ≥ baseline) → 02 Shadow traffic (mirror requests, discard output, 0% user risk) → 03 Canary 1→5→25% (auto-halt) → 04 Full rollout (SLO dashboard, on-call owned)
> Rollback triggers (automated): groundedness −5% vs baseline · tool errors >2× baseline · p95 over SLO 10 min · cost/session >1.5× budget · escalation spike
> Cheap rollback: versioned prompts/configs · stateless orchestrator · replayable traces · feature flags · warm last-known-good
> Targets: detect <10 min · rollback <2 min · shadow soak ≥1 week · canary floor ≥5% for ≥24h

Delivery discipline closes the loop. Agents ship like software, because they ARE software — just with weirder inputs.

Promotion path, four stages. **Offline evals** must beat baseline to promote. **Shadow traffic** mirrors real requests and discards output — zero user risk, production-shaped evidence; Anthropic runs parallel versions for exactly this reason. **Canary** ramps one, five, twenty-five percent with auto-halt. **Full rollout** lands on-call owned, SLO dashboard, error budget.

Two ideas to land. First, shadow mode gives production-shaped evidence without production risk. Second, **automated rollback wired to guardrails, not to a human noticing.** Trigger fires on metric breach in minutes, not at next morning's standup: groundedness minus five percent against baseline, tool errors doubled, p95 breached ten minutes, cost per session past one-and-a-half-x budget, escalation-rate spike.

What makes rollback cheap: versioned prompts and configs, stateless orchestrator, replayable traces, feature-flagged versions, warm last-known-good.

Target numbers to write down: regression detected under ten minutes, rollback under two, shadow soak at least a week, canary floor five percent for at least twenty-four hours.

And notice — **Replit's post-incident fixes ARE this slide.** Restore checkpoints are rollback infra; dev/prod separation is environment isolation; the emergency off-switch is the kill switch. Good incident responses converge on the same architecture.

---

## Slide 15 — The task horizon keeps doubling ✂️ ⏱ ~3 min

> ### ON SCREEN
> METR: autonomous task length (50% success) doubling ~every 7 months since 2019 — recently ~4 months.
> 2019 seconds-long → early 2025 ~50-minute (Claude 3.7 Sonnet, o1) → ~2030 month-long projection (contested, no reversal yet)
> So what: every pattern in this talk scales WITH the horizon.

Why urgency? Because everything we built gets stress-tested harder.

METR measures the length of tasks agents complete autonomously at fifty percent success. It has **doubled roughly every seven months since 2019 — recently closer to four.** Milestones: 2019, seconds-long tasks. Early 2025, roughly fifty-minute tasks. If the trend holds — contested, no credible reversal signal yet — month-long autonomous tasks somewhere around 2030.

Caveats honestly: the fifty-percent bar is generous, benchmarks skew toward coding. Directional, not gospel.

But the so-what stands: **every pattern in this talk scales WITH the horizon.** Context budgets, eval gates, rollback, cost caps — none of it wasted on short tasks; it's what makes longer-horizon agents shippable. You're not building for the agent that demoed well last month. You're building for the agent that ships two doublings from now.

---

## Slide 16 — Eight things to do before you scale ⏱ ~3 min

> ### ON SCREEN
> The checklist, 1–8. Read aloud, slowly.

We've covered a lot. Here's the whole talk compressed — eight things to do before you scale. Reading slowly; screenshot freely.

One. **Name two numbers before scaling:** end-to-end task success rate and cost per completed task. No numbers, no system — just a demo.

Two. **Pick orchestration by task shape, not fashion** — fan out for read-only breadth only; write paths single-threaded, full context.

Three. **Give every delegation a contract:** deadline, token budget, structured summary back. Workers compress; only the owner synthesizes.

Four. **Engineer for cache hits from day one:** stable prefix, append-only history, mask-don't-remove tools. Ten-x on input price and faster fiftieth-percentiles.

Five. **Assume r-to-the-N decay:** retries with backoff and jitter, fallbacks, breakers, timeouts — idempotency keys on anything that mutates state.

Six. **Break hallucination loops structurally:** grounding and validation gates that fail closed — not vibes-based prompting.

Seven. **Ship behind gates:** tiered evals in CI, shadow mode, canary ramps, automated rollback wired to guardrails. Version prompts like code.

Eight. **Own the evaluation function.** Decide deliberately what the system optimizes — or it optimizes whatever you forgot to measure.

Number eight is the deepest one. Klarna didn't fail at AI — they optimized exactly what their weights encoded. Make sure someone owns deciding what gets measured.

---

## Slide 17 — Everything quoted, traceable ⏱ ~1 min + Q&A

> ### ON SCREEN
> Sources grouped: Systems & practice (Anthropic ×3, Cognition, Manus, LangChain, OpenAI AgentKit) · Studies & market (METR ×2, MIT NANDA, Gartner) · Incidents press coverage (TechCrunch/InfoQ, BC Tribunal, BBC/CNBC/Fortune)

Last slide — maybe the most important. Every number in this talk traces to a source: Anthropic's engineering posts, Cognition's "Don't Build Multi-Agents," Manus's context-engineering piece, METR's papers, MIT NANDA, Gartner, plus press coverage of the three case studies. Primary sources except where noted secondhand.

Deck and links shared after the talk. If you quote any number from today and someone challenges it — send them here.

That's the talk. Thank you. Questions?

*(Leave this slide up during Q&A. If challenged on any number: name source and date.)*

---

## Timing summary

| Section | Slides | Budget |
|---|---|---|
| Open | 1 | 2 min |
| 01 Evidence | 2–4 | 11–13 min |
| 02 Orchestration | 5–6 | 10 min |
| 03 Context engineering | 7–8 | 9–10 min |
| 04 Reliability | 9–10 | 10–11 min |
| 05 Cost | 11 | 5 min |
| 06 MLOPS | 12–14 | 13 min |
| 07 Horizon + Close | 15–17 | 7 min |
| **Upper bound** | 17 | **~70 min** |

⚠️ **For a 40-minute slot:** deliver slides **2, 5, 7, 9, 12, 16** at full depth (they carry the argument); run ✂️-marked slides briskly using opening paragraph + day-one banner only. For 45 minutes, restore detail on 10 and 14.

## Compression guide

- **Slide 8 → 90 sec:** four moves by name + banner; skip "in practice."
- **Slides 13+14 merged → 4 min:** "test in tiers, ship behind gates," one example each.
- **Slide 15 → 60 sec:** doubling stat + closing line only.
- **Never cut:** 2 (motivating evidence), 5 (decision rule), 9 (rᴺ), 16 (checklist) — the spine of the talk.

## Delivery notes

- **[EXHIBIT] pauses** on slides 6, 7, 9, 10, 15 — the canvas visuals need a beat to land before you talk over them.
- Audience questions staged on 6, 9, 11 — take max one answer each.
- Deck keys: **O** deck index (jump if the moderator cuts your slot) · **N** speaker notes · **F** fullscreen.
