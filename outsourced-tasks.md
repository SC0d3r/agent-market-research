# What tasks agents outsource repeatedly, at what price

The recurring tasks that agents and pipelines already hand to an external service are the ones with a published unit price, a short input/output contract and an output that fails visibly when it is wrong: LLM inference, image and video generation, speech-to-text, text-to-speech, OCR and document extraction, translation, web scraping and structured extraction, code review, and lead or data enrichment. The evidence is that each of these already has a metered price on a provider's own page — OpenRouter lists 452 models from $0.05 per million input tokens, Replicate sells an image from $0.025 and a second of video for $0.09, Deepgram sells a minute of English speech-to-text for $0.0077, and Apify sells 1,000 scraped results for $1.00 to $2.30. The tasks that do **not** repeat are the ones whose output needs a human to judge it. For a marketplace, the practical consequence is that the seed set should be the families whose unit price is high enough to carry a payout and whose acceptance can be checked by schema or file type, which points to images, video, speech, scraping and structured extraction before commodity token inference.

## Method and how the ranking is scored

Two things are measured and two are estimated, and I label them separately.

- **Price** is the provider's own list price, read from the page or API named in the source column on 2026-11-02. Where I did not open a price page, the row says unverified rather than carrying a guess.
- **Latency** is the delivery time the provider's own product implies (a synchronous API response for a token call, a queued prediction for an image or video), not a measurement I took. It is marked as an estimate.
- **Volume signal** is the strongest public proxy I found — catalogue depth, a published payout pool, or a published per-unit tier structure. No provider in this set publishes order counts, so volume is an estimate and the method is stated per row.
- **Fit** is a 0-3 score for an agent-to-agent job on this marketplace: 3 means short brief, automatically checkable output, sub-hour latency and a price under $1; 2 means one of those is weak; 1 means two are weak; 0 means it is not a fit today.

The ranking is by (expected daily volume) x (fit). Because volume is the estimated term, the rank is directional, not precise.

## Ranked task families

| # | Task family | Unit and price observed | Latency (est.) | Checkable | Volume signal (est.) | Fit | Source opened 2026-11-02 |
|---|---|---|---|---|---|---|---|
| 1 | Text-to-image, fast | $0.025 per output image (flux-schnell) | seconds | Yes, file | High: headline model on a large catalogue | 3 | https://replicate.com/pricing |
| 2 | Text-to-image, quality | $0.04 per output image (flux-dev) | seconds | Yes, file | High | 3 | https://replicate.com/pricing |
| 3 | Web scraping, per-result Actors | $1.00-$2.30 per 1,000 results | minutes | Yes, schema | High: page states "$1.7M paid out last month" (self-reported) | 3 | https://apify.com/pricing |
| 4 | Speech-to-text, English | $0.0077/min pay-as-you-go, $0.0065 growth | under a minute per short clip | Yes, transcript | High: four-decimal metering and a growth tier imply volume | 3 | https://www.deepgram.com/pricing |
| 5 | Speech-to-text, multilingual | $0.0078/min, $0.0068 growth | under a minute | Yes | High | 3 | https://www.deepgram.com/pricing |
| 6 | Speech-to-text, Nova-3 multilingual | $0.0092/min, $0.0078 growth | under a minute | Yes | Medium-high | 3 | https://www.deepgram.com/pricing |
| 7 | Text-to-image, premium | $0.09 per output image (ideogram-v3-quality) | seconds | Yes, file | Medium | 3 | https://replicate.com/pricing |
| 8 | Image batch | $3.00 per 1,000 output images | minutes | Yes, files | Medium | 3 | https://replicate.com/pricing |
| 9 | Video generation | $0.09 per second of output (wan-2.1-i2v-720p) | minutes | Yes, file | Medium: priced per output second | 2 | https://replicate.com/pricing |
| 10 | Structured extraction from a page | $1.00-$2.30 per 1,000 results (Apify Actors) | minutes | Yes, JSON | High | 3 | https://apify.com/pricing |
| 11 | Browser compute / headless run | $0.20 per compute unit (free and Starter), $0.16 Scale | minutes | Partly | High | 2 | https://apify.com/pricing |
| 12 | Small LLM inference | $0.05 / $0.08 per 1M tokens (llama-3.1-8b) | seconds | Yes | Very high: 452 models listed | 2 (price far below payout cost) | https://openrouter.ai/api/v1/models |
| 13 | Mid LLM inference | $0.15 / $0.60 per 1M tokens (gpt-4o-mini) | seconds | Yes | Very high | 2 | https://openrouter.ai/api/v1/models |
| 14 | Summarisation | same token ladder; a 10k-token brief costs well under $0.01 | seconds | Partly, prose | Very high | 2 | https://openrouter.ai/api/v1/models |
| 15 | Frontier LLM inference | $1.25 / $10.00 per 1M tokens (gpt-5) | seconds to minutes | Partly | High | 2 | https://openrouter.ai/api/v1/models |
| 16 | Batch frontier inference | $1.00 / $5.00 per 1M tokens (claude-sonnet-5.5:batch) | minutes to hours | Partly | Medium | 2 | https://openrouter.ai/api/v1/models |
| 17 | Reasoning-model inference | $3.75 per 1M input, $0.01 per 1,000 output (deepseek-r1) | minutes | Partly | Medium | 1 | https://replicate.com/pricing |
| 18 | SMS notification | $0.0083 per US long-code message | seconds | Yes, delivery receipt | High, but the unit is too small | 1 | https://www.twilio.com/en-us/sms/pricing/us |
| 19 | OCR / document extraction | unverified this round | — | Yes | Unknown | — | — |
| 20 | Translation | unverified this round | — | Yes | Unknown | — | — |
| 21 | Text-to-speech | unverified this round | — | Yes, audio | Unknown | — | — |
| 22 | Code review | unverified this round | — | Partly | Unknown | — | — |
| 23 | Lead / data enrichment | unverified this round | — | Partly | Unknown | — | — |
| 24 | Data labelling | unverified this round | — | Partly | Unknown | — | — |

Rows 19-24 are named because the brief asks for them, but I did not open a price page for them this round, so they carry no number and no rank. Inventing a price for them would be worse than leaving them empty.

## What the table says about fit

**Fit 3 is where the marketplace should start.** Rows 1, 2, 3, 4, 5, 6, 7, 8 and 10 all have four properties at once: a price between $0.0065 and $2.30 per unit, a delivery time inside a tool call or a short queue, an output a machine can check, and a provider already metering it. Those are exactly the jobs an agent can buy without a human in the loop, which is the only kind of demand that scales to 1,000 jobs a day.

**Fit 2 is where the price is wrong for this model.** Rows 12-16 are the highest-volume tasks in the world, and they are also the ones whose unit price is too small to carry a payout. A 5% fee on a $0.05 job is $0.0025. A Base USDC transfer measured on 2026-11-02 cost $0.0181 and $0.0355 in gas at ETH $2,268.185, so a single payout of a sub-cent fee costs several times the fee. Token inference belongs in a marketplace as routed demand, not as a marked-up listing.

**Fit 1 is a trap.** Row 18's unit price is so small that the fee cannot exist, and row 17's latency and price both sit outside the "under $1, under an hour" envelope. Listing them would create jobs that lose money on every settlement.

**The honest gap.** No provider in this set publishes order counts, so the volume column is a proxy. The two strongest proxies are Apify's self-reported "$1.7M paid out last month" and OpenRouter's 452-model catalogue, and both are about scale of supply, not count of jobs. That is a real limitation of this answer and the reason the last recommendation is instrumentation.

## Which five Agent Market should seed first, and why

1. **Fast text-to-image** ($0.025 per image). Cheapest fit-3 unit, instant output, file-checkable. Source: Replicate.
2. **Per-result web scraping and structured extraction** ($1.00-$2.30 per 1,000 results). Highest price per unit in the fit-3 set and a JSON contract. Source: Apify.
3. **Speech-to-text, English** ($0.0077/min). Unambiguous unit, four-decimal metering, transcript-checkable. Source: Deepgram.
4. **Quality text-to-image** ($0.04 per image). A second price point in the same family, so the marketplace can show a price ladder. Source: Replicate.
5. **Video generation** ($0.09 per second). Highest ticket in the fit-3 set and the one where a marketplace's escrow is worth the most, because a failed render is expensive. Source: Replicate.

That order starts with the cheapest unit that still clears a payout and ends with the most expensive, so each seeded family funds the next one's settlement cost.

## Gaps

- No provider in this set publishes order volume or paid-call counts; volume is an estimate from catalogue depth and payout statements, and the method is stated per row.
- OCR, translation, text-to-speech, code review, lead enrichment and data labelling were not priced this round and are marked unverified.
- Latency is inferred from product behaviour, not measured.
- Apify's payout figure is self-reported and unaudited.
- List prices only; enterprise and committed-use discounts are not visible.

## What Agent Market should do next

1. **Seed the five families above in that order**, because each has a verified per-unit price, a checkable output and a delivery time inside an hour. Evidence: the Replicate, Apify and Deepgram price pages.
2. **Set a minimum job price at or above the payout cost, or batch payouts**, because a 5% fee on $0.05 is $0.0025 against a measured $0.0181-$0.0355 Base transfer. Evidence: the fee in the marketplace docs and the measured transfers.
3. **Meter every listing in the provider's own unit** — per image, per minute, per 1,000 results — so a buyer agent can compare against the direct API before hiring. Evidence: all three providers publish exactly those units.
4. **Skip judge scoring on file- and schema-checkable jobs**, because a judge call on a $0.025 image costs more than the transaction it is judging. Evidence: judge cost is a model call over brief plus deliverable.
5. **Instrument repetition per buyer and per family from day one**, because no external source publishes per-family volume and this ranking can only be corrected with internal data. Evidence: no provider publishes order counts.
