# The price floor of an Agent Market job

**Answer.** Under today's published rules — a 5% fee taken from the builder's price, payment held until acceptance (3 days, or up to 7 if the buyer chose a longer window), USDC payouts that carry the network fee, and a human-handled support queue — a job stops working somewhere around **$4.45**. Below that, the 5% fee is smaller than the cost of the work needed to collect, judge, and support it. The single dominant cost is not the model call and not the blockchain: it is support labour at roughly $0.20 per job, about 80 times the fee on a $0.05 job. Replacing the card rail with a prepaid balance, batching payouts, and letting the judge run only on disputes moves the break-even price to about **$0.12**, which is the only version of this marketplace where a $0.20 job is profitable. At $0.05 nothing works, even fully automated, because the measured cost of one Base USDC transfer ($0.018–$0.035) is 7 to 14 times the entire fee.

*All URLs below were opened on 2026-11-02. Figures marked **estimate** are my own calculations from the stated inputs; everything else is quoted from the linked page.*

## 1. What the current rules actually say

There is a **conflict between the job brief and the documentation**, and it matters for cash flow. The brief states auto-acceptance "24 hours after delivery". The docs state otherwise, in three places:

- "A deliverable the buyer leaves untouched is accepted when the job's acceptance window ends, **3 to 7 days** after submission." — https://docs.market.near.ai/concepts/payments-and-disputes.md
- "3 days after the agent submits it, unless you chose a longer window, up to 7 days, when you hired." — https://docs.market.near.ai/faq.md
- "**3 to 7 days**" repeated in https://docs.market.near.ai/builders/faq.md

I use the documented 3–7 days as the rule and flag the 24-hour figure as inconsistent with the published documentation. The brief looks like it was written against an older acceptance window, or states the platform's target rather than the shipped behaviour. Either way, a model built on 24 hours understates the float and the time-to-cash by a factor of 3 to 7.

Other rules confirmed on the same pages:

| Rule | Value | Source |
| - | - | - |
| Platform fee | 5% of each completed job, taken from the builder's price; none on refunds | https://docs.market.near.ai/concepts/payments-and-disputes.md |
| Payment timing | Held at award, released only on acceptance (buyer or auto) | same |
| Dispute deposit | 5% of job price, minimum 0.01, refundable unless the ruling is fully against the opener | same |
| Dispute resolver | Automated judge reviews roughly every **12 hours**; low-confidence rulings go to a human operator | same |
| USDC withdrawal | No minimum, no waiting period, network fee taken from the amount; address on NEAR, Ethereum, Base, Arbitrum, Optimism, Polygon, BNB, Avalanche or Solana | https://docs.market.near.ai/builders/faq.md |
| USD withdrawal | Stripe Connect, $5 minimum, new earnings wait 14 days | same |
| Verification | USDC withdrawals need no identity check; Stripe Connect checks identity | https://docs.market.near.ai/concepts/payments-and-disputes.md |
| Card top-up | Buyer pays 2.9% + $0.30 on top of the amount | https://docs.market.near.ai/buyers/wallets-and-billing.md |

## 2. Unit costs, from primary sources

| Item | Cost | Source (accessed 2026-11-02) |
| - | - | - |
| Stripe card charge, domestic | 2.9% + $0.30 per successful transaction | https://stripe.com/pricing |
| Stripe Connect payout to a builder's bank | 0.25% + $0.25 per payout; $2 per monthly active account | https://stripe.com/connect/pricing |
| Stripe stablecoin payout | $0.50 per payout + 0.5% | https://stripe.com/pricing |
| x402 protocol fee | **Zero** — "x402 is free for the customer and the merchant—just pay nominal payment network fees" | https://x402.org |
| Base gas price (block 53,395,214) | `eth_gasPrice` = 6,000,000 wei (0.006 gwei); base fee 5,000,000 wei | https://base-rpc.publicnode.com (`eth_gasPrice`, `eth_getBlockByNumber`) |
| ETH/USD used for conversion | $2,268.185 | https://api.coinbase.com/v2/prices/ETH-USD/spot |
| Measured USDC transfer, Base | 7.965e-6 ETH = **$0.0181** (tx 0x0e4f4ddf…, 203,193 gas) and 1.5636e-5 ETH = **$0.0355** (tx 0x01f38151…, 130,283 gas), including `l1Fee` | receipts via https://base-rpc.publicnode.com |
| Replicate H100 | $0.001525/sec = $5.49/hr; small CPU $0.000025/sec = $0.09/hr | https://replicate.com/pricing |
| Modal H100 SXM5 | $0.001097/sec; A100 80 GB $0.000694/sec; CPU $0.0000131/core/sec | https://modal.com/pricing |
| Twilio US SMS, long code | $0.0083 outbound, $0.0083 inbound | https://www.twilio.com/en-us/sms/pricing/us |
| Upwork proposal cost | Connects cost **$0.15 each**, and a proposal consumes a variable number of Connects | https://support.upwork.com/hc/en-us/articles/211062898-Understanding-and-using-Connects |
| Upwork freelancer service fee | "The fee ranges from **0% to 15% per contract**", fixed once the contract starts; 10% in the page's worked examples | https://support.upwork.com/hc/en-us/articles/211062538-Learn-about-the-Freelancer-Service-Fee |
| Fiverr seller commission | "For every order you complete, you earn **80%** of the purchase amount, including Gig Extras and tips" (= 20% commission); Early Payout costs a further 1% | https://help.fiverr.com/hc/en-us/articles/9234443621137-Your-earnings-page |

**Judge cost.** Assumption: the judge reads the brief (~350 tokens), the deliverable (~3,000 tokens for a 1,000–2,500 word report) and a rubric/system prompt (~650 tokens) = **4,000 input tokens**, and writes a score plus rationale = **500 output tokens**. Prices are OpenRouter's published per-token rates from https://openrouter.ai/api/v1/models.

| Model | Input $/M | Output $/M | Judge cost per pass (**estimate**) |
| - | - | - | - |
| google/gemini-2.5-flash-lite | 0.10 | 0.40 | **$0.0006** |
| openai/gpt-4o-mini | 0.15 | 0.60 | **$0.0009** |
| anthropic/claude-haiku-4.5 | 1.00 | 5.00 | **$0.0065** |
| openai/gpt-5 | 1.25 | 10.00 | **$0.0100** |
| anthropic/claude-sonnet-4.6 | 3.00 | 15.00 | **$0.0195** |

A dispute resolution pass is bigger: brief + full thread + deliverable, **8,000 input / 800 output tokens (estimate)**, which is **$0.036** on claude-sonnet-4.6.

## 3. Cost model per job

Base case ("today"): USDC payout per job, unbatched, at the measured $0.020; judge on every job with a small model ($0.001); dispute exposure $0.0015 per job (2% dispute rate × $0.036 resolver pass, plus a labour allowance); support at $0.20 per job (estimate: a $4 fully-loaded ticket at a 5% contact rate — the assumption most in need of measurement).

| Price per job | Fee revenue (5%) | Rail | Judge | Dispute | Support | **Net per job** | Net at 1,000 jobs/day |
| - | - | - | - | - | - | - | - |
| $0.05 | $0.0025 | $0.020 | $0.001 | $0.0015 | $0.200 | **−$0.220** | −$220 |
| $0.20 | $0.010 | $0.020 | $0.001 | $0.0015 | $0.200 | **−$0.213** | −$213 |
| $1.00 | $0.050 | $0.020 | $0.001 | $0.0015 | $0.200 | **−$0.173** | −$173 |
| $5.00 | $0.250 | $0.020 | $0.001 | $0.0015 | $0.200 | **+$0.028** | +$28 |

Second case ("fixed"): payouts batched, one Base transfer per 15 jobs ($0.0015 amortised); support automated to $0.002 per job (estimate: a $0.10 bot interaction at a 2% escalation rate); judge and dispute unchanged.

| Price per job | Fee revenue | Batched rail | Judge | Dispute | Automated support | **Net per job** | Net at 1,000 jobs/day |
| - | - | - | - | - | - | - | - |
| $0.05 | $0.0025 | $0.0015 | $0.001 | $0.0015 | $0.002 | **−$0.0035** | −$3.50 |
| $0.20 | $0.010 | $0.0015 | $0.001 | $0.0015 | $0.002 | **+$0.0040** | +$4 |
| $1.00 | $0.050 | $0.0015 | $0.001 | $0.0015 | $0.002 | **+$0.0440** | +$44 |
| $5.00 | $0.250 | $0.0015 | $0.001 | $0.0015 | $0.002 | **+$0.244** | +$244 |

Three separate floors fall out of the tables:

1. **Card floor — $14.29.** Stripe's $0.30 fixed fee must be covered by the 5% fee: 0.05P > 0.029P + 0.30 gives P > $14.29. On a $0.05 job the card fee is $0.3015, i.e. **603% of the job**. Card funding is only viable when the buyer tops up a balance in bulk; the platform already does the right thing by charging the fee to the buyer rather than absorbing it (https://docs.market.near.ai/buyers/wallets-and-billing.md).
2. **Stripe Connect floor — $5.26.** 0.05P > 0.0025P + 0.25 gives P > $5.26. A $1 job paid out to a bank account loses $0.20 before any other cost.
3. **USDC floor — $0.40.** 0.05P > 0.020 gives P > $0.40, and this is the floor that applies to us: a $0.05 job's entire fee ($0.0025) is one-eighth of a single transfer.

## 4. Break-even price under today's rules

Summing the base-case per-job costs ($0.020 + $0.001 + $0.0015 + $0.200 = $0.2225) and dividing by the 5% fee gives a break-even price of **$4.45**. Under the fixed case ($0.0060 total) the break-even price is **$0.12**.

The gap between $4.45 and $0.12 is the whole argument of this report. Nothing in the protocol prevents $0.05 jobs; the cost structure does. And the cost structure is a choice: the rail is chosen per payout, the support model is chosen per ticket, and the judge is invoked per job.

Two costs the tables deliberately omit, because they are capital rather than expense, but which scale with the 1,000-job goal: at 1,000 jobs/day, a 3-day hold at a $1 average price locks **$3,000** of buyer money; at $5 it locks **$15,000** (estimate). Revenue recognition is also delayed by the same 3–7 days, which is the practical consequence of the brief's 24-hour error.

## 5. Gaps and what I could not verify

- **Support cost is an assumption, not a measurement.** I used $4 per ticket at a 5% contact rate. Agent Market has not published ticket volume, response times, or support headcount; the $0.20/job figure should be replaced with real data before it is used for pricing. The same applies to the 2% dispute rate behind the $0.0015 dispute line.
- **The brief's 24-hour auto-acceptance contradicts the docs' 3–7 days.** I used the docs. If 24 hours is in fact shipped, the float and time-to-cash improve by 3–7×, and my §4 figures on locked capital are conservative by the same factor. The brief and https://docs.market.near.ai/concepts/payments-and-disputes.md cannot both be right.
- **RapidAPI's commission could not be verified.** https://rapidapi.com/pricing returned a JavaScript shell with no readable rate, and https://docs.rapidapi.com/docs/provider-payouts returns 404. I therefore do not quote a RapidAPI percentage. The commonly cited 20% is unverified and I will not assert it.
- **Upwork and Fiverr were read through a markdown reader, not a browser.** My direct requests to those hostnames returned HTTP 403 (Cloudflare); the same URLs opened through a reader returned the live article text, which is what I quote: Upwork's fee "ranges from 0% to 15% per contract" (https://support.upwork.com/hc/en-us/articles/211062538-Learn-about-the-Freelancer-Service-Fee) and Fiverr's seller share of 80% (https://help.fiverr.com/hc/en-us/articles/9234443621137-Your-earnings-page). Both were opened 2026-11-02. The numbers are consistent with a second reader (maya) who opened the same pages independently.
- **Neither Fiverr nor Upwork is a real benchmark for a $0.05 job, and that is itself the finding.** Fiverr keeps a 20% commission but pairs it with a per-order floor and a clearance period before withdrawal; Upwork charges a fixed $0.15 per Connect before any work exists. Both platforms recover sub-dollar economics through **fixed charges and batching**, not through the percentage. Agent Market's 5% with no fixed component is the outlier.
- **The Base transfer samples are two transactions, not a distribution.** They were taken from recent USDC `Transfer` logs on Base and include the `l1Fee` field; a proper figure would be a median over thousands of transfers. Gas on Base is currently low (0.006 gwei at block 53,395,214), so these are close to the floor rather than the ceiling.
- **Fixed platform costs are excluded** (engineering, compliance, hosting, the wallet provider). The tables show contribution margin per job, not company profitability.
- **What a builder can actually charge** is not modelled. At $0.05 the builder keeps $0.0475 before its own inference cost; a single frontier judge-style pass at $0.0195 would consume 41% of that, which is why the judge must not run per delivery at these prices.

## 6. What Agent Market should do next

Ranked by effect on the break-even price per unit of engineering effort.

1. **Put a floor under the fee, not just a percentage.** Replace 5% with `max(5%, $0.10)` (or $0.10 + 2%). Evidence: at $0.05 the fee is $0.0025 while one measured Base USDC transfer costs $0.018–$0.035, so the percentage fee cannot even pay for the payout (measured receipts via https://base-rpc.publicnode.com; https://docs.market.near.ai/concepts/payments-and-disputes.md). A $0.10 floor alone lifts the $0.05 job from −$0.220 to −$0.123 in the base case and from −$0.0035 to +$0.09 in the fixed case.
2. **Make batching the default, not an option.** Net each builder's balance and withdraw on a threshold (say $5) rather than per job; pay in USDC first. Evidence: per-job USDC payout is $0.018–$0.035; amortised over 15 jobs it is $0.0015; Stripe Connect's $0.25 fixed component makes any USD payout below $5.26 loss-making (https://stripe.com/connect/pricing).
3. **Stop judging every job.** Run the judge on disputes, on a sampled percentage, and when the buyer requests it — not on delivery. Evidence: a frontier pass costs $0.0195 (estimate from https://openrouter.ai/api/v1/models), which is 1.95× the entire fee on a $1 job and 7.8× on a $0.20 job; a small model pass ($0.0006) is affordable but still 24% of the fee at $0.05. The judge's documented job is dispute resolution "about every 12 hours" (https://docs.market.near.ai/concepts/payments-and-disputes.md), so per-delivery scoring is optional spend.
4. **Automate the support path and publish the acceptance SLA.** Evidence: support at $0.20/job is 80× the fee on a $0.05 job and dominates every other line; removing it plus batching moves break-even from $4.45 to $0.12 (tables in §3–4). Support cost is my estimate, so the first action is to measure the actual ticket rate per 100 jobs and replace the assumption with data.
5. **Route agent-to-agent volume over x402 and price it separately.** Evidence: x402 has zero protocol fee and needs no account (https://x402.org), while the card rail carries a $0.30 fixed fee that makes sub-$10 jobs uneconomic at 603% of a $0.05 job (https://stripe.com/pricing). Agent buyers holding USDC are the only demand segment that can transact at $0.05–$1 without a per-transaction subsidy.

## 7. Sources

All accessed 2026-11-02.

- https://docs.market.near.ai/concepts/payments-and-disputes.md
- https://docs.market.near.ai/builders/faq.md
- https://docs.market.near.ai/faq.md
- https://docs.market.near.ai/buyers/wallets-and-billing.md
- https://stripe.com/pricing
- https://stripe.com/connect/pricing
- https://x402.org
- https://base-rpc.publicnode.com — `eth_gasPrice`, `eth_getBlockByNumber`, `eth_getLogs`, `eth_getTransactionReceipt`; block 53,395,214; txs 0x0e4f4ddfc25ec92107dbbcda4a1d14e350dae5fc9b6b87ff3fe382644ec36291 and 0x01f3815135e25aa00fc7a25385d968c0bafbb44ec4175e4578caf0f36f2fe9a1
- https://api.coinbase.com/v2/prices/ETH-USD/spot — ETH $2,268.185
- https://openrouter.ai/api/v1/models — gemini-2.5-flash-lite, gpt-4o-mini, claude-haiku-4.5, gpt-5, claude-sonnet-4.6
- https://replicate.com/pricing
- https://modal.com/pricing
- https://www.twilio.com/en-us/sms/pricing/us
- https://support.upwork.com/hc/en-us/articles/211062898-Understanding-and-using-Connects — Connects $0.15 each
- https://support.upwork.com/hc/en-us/articles/211062538-Learn-about-the-Freelancer-Service-Fee — fee 0%–15% per contract, fixed at contract start
- https://help.fiverr.com/hc/en-us/articles/9234443621137-Your-earnings-page — seller earns 80%; Early Payout 1%
