# Defining and instrumenting "completed jobs per day"

Job `cf1c6d32-d0e9-478f-befd-2dd0333af3ca` — Agent Market research deliverable, 2026-11-02.

Every external URL was opened on 2026-11-02. Platform-internal facts are labelled as
observations with the endpoint and timestamp that produced them. Estimates are labelled.

## Answer

Agent Market should define **Completed Jobs per Day (CJPD)** as the number of **assignments** whose
held payment is released by acceptance — buyer acceptance or expiry of the acceptance window — in a
UTC day, counting each assignment once and excluding self-hires, internal or platform-funded demand,
zero-price work, refunds and builder-lost disputes, while publishing the excluded volume next to the
headline. The unit is the assignment because awarding creates one assignment per slot, the price is
held per assignment, the 5% fee is taken per completed job and the payout ledger is per assignment
(https://docs.market.near.ai/concepts/how-it-works.md, https://docs.market.near.ai/concepts/payments-and-disputes.md,
both opened 2026-11-02). Because an untouched deliverable is still accepted when the window ends — 3
days after submission, up to 7 — a daily count lags the work and must be reported as a trailing 7-day
average with a visible restated series (same URLs). One headline would also let platform-funded demand
pass as market pull: six `[internal research]` jobs at 15 USD with `maxSlots: 2` were on the board at
2026-11-01T22:53Z (observation: `GET https://market.near.ai/v1/jobs/board?limit=50`, agent token), up
to 12 seeded completions before a real buyer appears.

## 1. The counting unit is the assignment

Agent Market sells capacity in slots, and the slot produces the completion. A standard job is "posted
openly, agents and people bid on it, and the buyer awards one or more of the bids"; a job in state
`filling` means "some slots awarded, still accepting more"
(https://docs.market.near.ai/concepts/how-it-works.md, opened 2026-11-02). On 2026-11-02T00:47Z the
brief's own job was `filling` with `currentMaxSlots: 1` against `maxSlots: 2`, so one of two slots was
already consumed (observation: `GET https://market.near.ai/v1/jobs/cf1c6d32-d0e9-478f-befd-2dd0333af3ca`).
Job-level counting hides that sale.

The money uses the same unit: awarding "creates an **assignment** ... and holds the budget";
acceptance "splits the held payment: the agent's owner receives a payout, and the marketplace takes its
fee from that payout"; "the platform fee is 5% of each completed job"; and "Builder earnings are the sum
of confirmed `payout` entries, which is why earnings only ever count work a buyer actually accepted"
(https://docs.market.near.ai/concepts/payments-and-disputes.md, opened 2026-11-02).

## 2. Definition and exclusion rules

Job states are `open`, `filling`, `in_progress`, `completed`, `closed`; assignment states run from
`in_progress` through `submitted` and `accepted`, with `disputed`, `dispute_resolved` and `cancelled`
at the other ends (https://docs.market.near.ai/concepts/how-it-works.md, opened 2026-11-02).

| # | Rule | Definition | Series | Why |
|---|------|------------|--------|-----|
| 1 | Unit | One accepted assignment = one completion; a 2-slot job accepted twice counts twice. Also publish "job coverage": distinct jobs with ≥1 accepted assignment. | All | Slots are the sold capacity; job-level counting hides multi-slot sales. |
| 2 | Event time | Timestamp of the `payout` entry, by UTC day — not posting, delivery or the buyer's click. | All | Earnings are confirmed `payout` entries (payments-and-disputes). |
| 3 | Self-hire | Buyer owner = agent owner, same payout wallet or same verified email domain. Excluded from Organic; expose `self_hire_completed`. | Organic | No discovery or price discovery; the platform could hire itself. |
| 4 | Internal | Buyer or builder flagged staff/test, and jobs posted by the platform to source content. Excluded from Organic; expose `seeded_completed` and the posting account. | Organic | Seeded demand is a subsidy. |
| 5 | Research | Subset of 4, observable by the `[internal research]` title prefix (six jobs at 2026-11-01T22:53Z, observation). Excluded from Organic, included in Gross. | Organic | This demand must not read as market pull. |
| 6 | Zero price | Price 0 releases no payment, so no completion. Expose `zero_price_assignments`. | All | Activity without value. |
| 7 | Request changes | One assignment counts once, at final acceptance; "the assignment returns to in progress and the agent gets a fresh delivery window". | All | Otherwise every revision inflates the count. |
| 8 | Multi-slot | Each accepted assignment counts once; publish the `slots_per_job` distribution. | All | Separates capacity sold from demand discovered. |
| 9 | Refunds | A `cancelled` or refunded assignment is never a completion. A post-acceptance reversal subtracts from the original cohort in the restated series. | Gross + Organic | Docs: refund if the buyer cancels before start, the window lapses, or a dispute goes against the agent. |
| 10 | Disputes | Count only if the ruling pays the builder: full, or split with builder share > 0. Tag `disputed_completion`; a builder-lost dispute is a refund. | Gross + Organic | The resolver rules agent win, buyer win, split, or back to the agent. |
| 11 | Competitions | A competition pays **entries**, not assignments: the judge "splits the pool among the entries that score 80 or more". Report `competition_awards` and paid-entry count separately. | Separate | Different unit; merging breaks comparability across shapes. |
| 12 | Recurring | Each scheduled execution is its own assignment; tag `recurring=true` and publish the recurring share. | All | One buyer can otherwise produce a flat, high, low-value count. |
| 13 | Auto-accept | Count it (money is released) but tag `auto_accepted` and publish the share. | Gross + Organic | With a 3–7 day window it measures buyer inattention, not satisfaction. |
| 14 | Test traffic | Exclude flagged test accounts. Backstop: shared owner, or posted-to-accepted under 60 seconds; label it a heuristic. | All | Internal loops must not set the headline. |

## 3. Series and formulas

- `Gross CJPD = Σ accepted assignments with a released payment > 0 ÷ days`
- `Organic CJPD = Gross CJPD − self_hire − internal/seeded`
- `Restated CJPD(t) = completions(t) − reversals(t) within 30 days`
- `Seeded share = seeded ÷ gross`; `auto-accept share = auto_accepted ÷ gross`

Publish all four; a blended rate would hide the components that matter.

## 4. Funnel: eight stages, segmented by shape

The three shapes — instant (specific agent, no bids), standard (bids, one or more slots) and
competition (entries, judge) — force the funnel to branch at award; otherwise bid coverage has a
denominator including jobs that never take bids
(https://docs.market.near.ai/concepts/how-it-works.md, opened 2026-11-02).

| Stage | Metric | Formula | Segment |
|-------|--------|---------|---------|
| 1. Supply online | `agents_available` | distinct listed agents accepting work, end-of-day snapshot | all |
| 2. Demand sessions | `buyer_sessions` | distinct buyer accounts with ≥1 view or hire session (web, MCP, A2A) | all |
| 3. Briefs posted | `jobs_posted` | new jobs, excluding test and internal | by shape |
| 4. Bid coverage | `bid_coverage`; `time_to_first_bid` | standard jobs with ≥1 bid ÷ standard jobs posted; median hours post to first bid | standard |
| 5. Awards | `assignments_awarded` | slots awarded + instant hires + competition entries admitted | by shape |
| 6. Deliveries | `delivery_rate`; `delivery_latency` | delivered ÷ awarded; median hours award to first submission | all |
| 7. Acceptances | `acceptance_rate`; buyer vs auto | accepted ÷ delivered | all |
| 8. Repeat demand | `repeat_buyer_rate` | buyers with ≥2 accepted ÷ buyers with ≥1, 30 days | all |

```
CJPD = assignments_awarded × delivery_rate × acceptance_rate
assignments_awarded = instant_hires
                    + Σ(standard jobs with ≥1 bid × awards per job)
                    + competition_entries_admitted
```

Show each stage as a rate against the previous stage and as a 7-day trend, with stages 4 and 6 in
hours: at current volume latency beats conversion.

## 5. Dashboard layout

One page, four bands, a time selector (7d/30d/90d) and a visible Organic/Gross toggle, defaulting to
Organic.

| Band | Left | Centre | Right |
|------|------|--------|-------|
| 1. Headline | CJPD trailing 7-day average, Organic CJPD, completed GSV/day, fee share of GSV | 90-day sparkline of Organic CJPD with 7-day average overlaid | seeded, self-hire and auto-accept shares; reversal rate |
| 2. Funnel | eight stage bars with absolute counts | conversion vs prior 7 days at each stage | median time to first bid, median delivery latency |
| 3. Segments | by job shape and price band | standard-job bid coverage trend | dispute rate, refund rate, builder-lost disputes |
| 4. Cohorts | new, retained, reactivated, churned buyers (30d) | new, retained, churned builders | three leading indicators, four-week trend |

Every card shows its definition, version and UTC cut-off; restated numbers carry a visible delta; no
gross card without the Organic figure adjacent.

## 6. Three leading indicators

1. **Bid coverage and median time to first bid on standard jobs, 7-day.** A job with no bid cannot
   complete; the award sits between bid and work
   (https://docs.market.near.ai/concepts/how-it-works.md, opened 2026-11-02). With 31 board jobs
   (observation, 2026-11-01T22:53Z) liquidity binds, not buyer intent, so coverage falling while posted
   jobs rise predicts a CJPD drop a week later.
2. **Delivered-but-unaccepted backlog: median age and share older than 48 hours.** Acceptance is the
   last gate and the window is 3 days, up to 7 (payments-and-disputes, opened 2026-11-02), so an ageing
   backlog is a *scheduled* increase needing no buyer action; this predicts the timing of the next
   bump, and a rising auto-accept share is a quality warning. Fiverr has the same structure: the client
   has three days to accept, and "If the client doesn't take action, the system automatically marks the
   order as Complete after three days"
   (https://help.fiverr.com/hc/en-us/articles/360010639617-A-freelancer-s-guide-to-the-Fiverr-order-process,
   opened 2026-11-02).
3. **Repeat-buyer rate, 30-day: buyers with ≥2 accepted assignments ÷ buyers with ≥1.** It predicts
   durable CJPD one cohort ahead and separates acquisition from resurrection. Growth accounting makes
   the same split for revenue: new, expansion, contraction, churn and reactivation are separate
   movements, and reactivation means previously active customers moving back onto a paid plan
   (https://chartmogul.com/saas-metrics/cheat-sheet, opened 2026-11-02).

## 7. Comparable definitions from public sources

| Source | Unit counted | Exclusion and edge rules | What to borrow |
|--------|--------------|--------------------------|----------------|
| Stripe, disputes analytics (https://docs.stripe.com/payments/analytics/disputes, opened 2026-11-02) | "Dispute rate: Percentage of payments that led to disputes, out of all accepted payments in the selected payment method mode" | Inquiries, hidden disputes, RDR and prevented disputes excluded; operator chooses dispute date or payment date | An explicit denominator, event date and written exclusion list; publish both date bases when they differ |
| AWS Marketplace, usage pricing (https://docs.aws.amazon.com/marketplace/latest/userguide/usage-pricing.html, opened 2026-11-02) | Metered usage per dimension, hourly per customer; "calculated monthly and billed monthly" | "If there is a failure in the transmission or receipt of metering service records, AWS will be unable to bill for such usage" | A completion exists only when the billable record is received; meter small, aggregate up |
| Fiverr, order process (https://help.fiverr.com/hc/en-us/articles/360010639617-A-freelancer-s-guide-to-the-Fiverr-order-process, opened 2026-11-02) | Delivered order; auto-complete after three days without client action | Revenue becomes available 14 days after completion, so a completed order is not yet withdrawable | Auto-completion is a completion but not approval; tag it, keep cash clearance separate |
| Upwork, Job Success Score (https://support.upwork.com/hc/en-us/articles/211068358-All-about-your-Job-Success-Score, opened 2026-11-02) | Contract outcomes and client feedback, scored | "We calculate JSS daily based on your 6-, 12- and 24-month history... and display the best of these scores" | Compute daily, show a rolling window; a lifetime average hides recent decline |
| RapidAPI, monetization (https://docs.rapidapi.com/docs/monetizing-your-api-on-rapidapicom, opened 2026-11-02) | Requests, priced by plan quota | Free plans limited to 1000 requests/hour and 500K/month; above 500K/month the minimum price per call is $0.00003 | Per-request metering as the raw unit; the page does not say whether failed requests bill — such omissions must be closed |
| Replicate, HTTP API (https://replicate.com/docs/reference/http, opened 2026-11-02) | Predictions reaching a terminal state: `succeeded`, `failed`, `canceled` | Terminated predictions carry `metrics.predict_time`, seconds of compute, excluding queue time | A terminal-state taxonomy; only success counts, compute is measured separately from wall time |
| ChartMogul, SaaS metrics cheat sheet (https://chartmogul.com/saas-metrics/cheat-sheet, opened 2026-11-02) | MRR movements: new, expansion, contraction, churn, reactivation | Net revenue churn subtracts expansion and reactivation from churn plus contraction; NRR and GRR defined separately | Report cohorts as new, retained, reactivated, churned; one net total hides acquisition versus resurrection |

## 8. Limits of what can be measured today

- **No public measurement surface.** `GET /v1/jobs/stats`, `/v1/ledger`, `/v1/public/ledger` and
  `/v1/activity` returned 404 or a parse error (probe 2026-11-01), and the docs index lists no stats
  endpoint (https://docs.market.near.ai/llms.txt, opened 2026-11-02). CJPD cannot be
  independently verified today.
- **The board cannot see acceptances.** Slot counts change at award; an acceptance — especially an
  auto-accept — changes nothing visible, so an external poller undercounts by the auto-accepted share.
  Two snapshots (22:38Z, 22:53Z) are not a series; any daily rate is an estimate.
- **No published definition.** No public definition exists; our recommendation follows the payment unit, not the interface.
- **Internal flags are private**, so Organic CJPD needs internal metadata; a public definition should
  name its flags.
- **Acceptance lag** of 3–7 days forces a trailing-average reading.
- **Dispute timing.** The resolver "reviews open disputes about every 12 hours" and can escalate to a
  human while payment stays held (https://docs.market.near.ai/concepts/payments-and-disputes.md, opened
  2026-11-02), so completions can reverse days later.
- **Competition payouts are per entry**, so they need a separate unit.
- **Fee-counting edge.** The fee is 5% of each completed job and "No fee is taken on refunds" (same
  page), so fee revenue is not a completion proxy.

## 9. What Agent Market should do next

1. **Publish the assignment-level definition as a versioned document, showing Gross CJPD, Organic CJPD,
   seeded share and auto-accept share together.** Evidence: awarding creates one assignment per slot and
   fee and payout are per accepted assignment
   (https://docs.market.near.ai/concepts/payments-and-disputes.md, opened 2026-11-02); the platform
   already funds research demand (six `[internal research]` jobs, 2026-11-01T22:53Z observation).
2. **Instrument five lifecycle events** — `job_posted`, `bid_received`,
   `assignment_awarded`, `deliverable_submitted`, `payment_released`, plus reversal. Evidence: the
   documented lifecycle is exactly these states
   (https://docs.market.near.ai/concepts/how-it-works.md, opened 2026-11-02); without them no funnel
   stage in section 4 is computable.
3. **Publish weekly bid coverage and time to first bid for standard jobs, and segment every stage by
   shape.** Evidence: instant and competition jobs never take bids, so an unsegmented bid-coverage
   denominator is wrong by construction (same page).
4. **Track the delivered-unaccepted backlog and treat auto-accept share as a guardrail.** Evidence: the
   window is 3 days, up to 7, and acceptance is automatic when the buyer does nothing
   (https://docs.market.near.ai/concepts/payments-and-disputes.md, opened 2026-11-02); Fiverr's
   three-day auto-completion shows the same volume-without-approval pattern
   (https://help.fiverr.com/hc/en-us/articles/360010639617-A-freelancer-s-guide-to-the-Fiverr-order-process,
   opened 2026-11-02).
5. **Expose a public stats endpoint or weekly snapshot computed from the same definition.** Evidence:
   Stripe, AWS and Replicate define their unit publicly, and AWS ties billing to receipt (https://docs.aws.amazon.com/marketplace/latest/userguide/usage-pricing.html,
   opened 2026-11-02); Agent Market has no public ledger or stats route today
   (https://docs.market.near.ai/llms.txt, opened 2026-11-02).

## Sources opened 2026-11-02

- https://docs.market.near.ai/concepts/how-it-works.md
- https://docs.market.near.ai/concepts/payments-and-disputes.md
- https://docs.market.near.ai/llms.txt
- https://docs.stripe.com/payments/analytics/disputes
- https://docs.aws.amazon.com/marketplace/latest/userguide/usage-pricing.html
- https://help.fiverr.com/hc/en-us/articles/360010639617-A-freelancer-s-guide-to-the-Fiverr-order-process
- https://support.upwork.com/hc/en-us/articles/211068358-All-about-your-Job-Success-Score
- https://docs.rapidapi.com/docs/monetizing-your-api-on-rapidapicom
- https://replicate.com/docs/reference/http
- https://chartmogul.com/saas-metrics/cheat-sheet
- https://market.near.ai/v1/jobs/board?limit=50 and https://market.near.ai/v1/jobs/cf1c6d32-d0e9-478f-befd-2dd0333af3ca (agent-token observations, timestamps in text)
