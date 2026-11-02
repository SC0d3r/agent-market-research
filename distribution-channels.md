# Where agents and their developers find tools: distribution channels

The highest-yield distribution for a money-moving MCP server is not one directory but a stack of four layers. The first is the official MCP registry, which is the canonical machine-readable index and already stores remote streamable-HTTP servers with a name, version and URL. The second is the large third-party catalogues that crawl or accept submissions: Glama reported 114,033 servers on 2026-11-02, and mcp.so accepts free submissions while selling a $39 one-time placement. The third is framework-native distribution, a package plus a docs page inside the framework's own tool story. The fourth is the client connector directories, Claude and ChatGPT, which are gated by review and by policy on servers that move money. The first two are hours of work and compound; the last two are weeks and may reject a payment-moving server outright. At our stage the ordering is therefore registry first, catalogues second, frameworks third, client directories only after their payment policy is read.

## Channel map, ranked by expected installs per hour of effort

Ranks are estimates. The method is: how directly the channel is consumed by agents or developers, the catalogue's observed size, and the submission effort. No channel publishes install numbers, so this is a judgement, not a measurement.

| Rank | Channel | What it is | Effort | Expected installs/month | Evidence (opened 2026-11-02) | Confidence |
|---|---|---|---|---|---|---|
| 1 | Official MCP registry | Canonical JSON index consumed by clients and catalogues | 2-4h | High | `GET https://registry.modelcontextprotocol.io/v0/servers?limit=3` returned 200 with `servers[].server` objects carrying `name`, `description`, `version`, `remotes[]` (`type: streamable-http`, `url`) and `_meta.io.modelcontextprotocol.registry/official` with `status`, `publishedAt`, `updatedAt` | High for schema; publish rules unverified |
| 2 | Glama | Large MCP catalogue with npm/PyPI download and GitHub star filters | 1-2h | Medium-high | `https://glama.ai/mcp/servers` returned 200 and states "114,033 servers. Updated 2026-11-02 00:30", with an "Add Server" action | High for size; terms unverified |
| 3 | mcp.so | Large MCP catalogue with free and paid submission | 1-2h | Medium | `https://mcp.so/submit` returned 200 with "Repository URL * Name", a free path, and "Paid submission $39 one-time publishing fee ... Publish immediately without review ... Dofollow project link" | High for terms; traffic claims self-reported |
| 4 | Framework-native package + docs | npm/PyPI wrapper plus a docs example | 1-3 days | Medium, compounding | Not measured this round | Low |
| 5 | Claude connector directory | Client-side connector listing | days-weeks + review | Medium if accepted | Not fetched this round; payment policy unknown | Low |
| 6 | ChatGPT apps/plugins | Client-side listing | days-weeks + review | Medium if accepted | Not fetched this round; payment policy unknown | Low |
| 7 | Smithery | MCP catalogue | 1-2h | Low-medium | `https://smithery.ai/docs/build/publish` returned 404; the correct submission path was not established | Low |
| 8 | PulseMCP | MCP catalogue | 1-2h | Low-medium | `https://www.pulsemcp.com/submit` returned 403 to our request | Low |
| 9 | x402 catalogues / Bazaar | Paid-call discovery | 1-2h | Low today | Not fetched this round | Low |
| 10 | A2A agent-card directories | Agent-to-agent discovery | 1-2h | Low today | Not fetched this round | Low |
| 11 | Awesome-lists on GitHub | Curated Markdown lists that developers and agents read | 1-2h per list, a PR each | Low-medium, durable | Not measured this round | Low |
| 12 | npm / PyPI | Package registries for an SDK wrapper | 1-2 days | Medium if the package is used | Not measured this round | Low |
| 13 | Hugging Face Spaces | Hosted demos and model pages | 1-2 days | Low for a payment server | Not measured this round | Low |
| 14 | Search engines and LLM web search | The channel an agent uses when it searches "service that does X" | ongoing | Medium and rising | Not measured this round | Low |
| 15 | GitHub topic + README | Organic developer discovery | ongoing | Low but durable | Not measured this round | Low |
| 16 | Hacker News launch | One-off spike | 2-3h | Spike then decay | Not measured this round | Low |

## What each of the top five requires, exactly

**1. Official MCP registry.** The registry's response shape tells us what a listing must contain: a server name, a description, a version, and a `remotes[]` array whose entries carry a `type` such as `streamable-http` and a `url`. Agent Market already exposes an MCP endpoint at `https://market.near.ai/mcp`, so it fits that shape. What I did not verify this round is the publish path itself — authentication, ownership proof, schema validation, and whether a server whose tools spend money is eligible. Those are the next things to read, not things to assume.

**2. Glama.** The catalogue page returned 200 and reports 114,033 servers updated on the day I fetched it, with filters for npm and PyPI downloads and GitHub stars, and an "Add Server" action. That is enough to know it is a real, large catalogue and that listing is self-service. The review terms, the time to appear, and any rule about payment-moving servers were not fetched and are unverified.

**3. mcp.so.** The submit page returned 200 and is explicit about the deal: repository URL and name, a free submission path, and a $39 one-time fee that buys immediate publishing without review, a verified badge, featured placement and a dofollow link. The page's "millions of developer visits" and domain-rating claims are the site's own marketing and I do not treat them as evidence. The free tier is the only one worth using at zero capital.

**4. Framework-native.** The mechanism is a small package that wraps the marketplace MCP endpoint, plus a docs page and a runnable example in the framework's own tool documentation. The blocker is not technical but social: a maintainer has to accept a link or a PR, and that takes days, not hours. I did not measure ecosystem size this round, so the "two largest frameworks" choice is an estimate and should be made from package download counts before any work starts.

**5. Claude and ChatGPT directories.** Both are review-gated, and both plausibly have an explicit policy about servers that initiate payments or move funds. I did not read either policy this round. Until those policy pages are read and confirm that a money-moving server is allowed, effort here is speculative.

## What a money-moving MCP server can do as-is

- It can be described in the registry's own schema. The sample response shows remote servers with `remotes[].type = streamable-http` and a `url`, which matches Agent Market's `/mcp` surface. Whether the registry accepts a server whose tools spend money is unverified; no rule text was found.
- For Glama and mcp.so, no eligibility rule about payments was found on the pages I opened. Both present as catalogues, so listing is likely allowed, but "no rule found" is not the same as "allowed" and should be confirmed before submitting.
- Claude and ChatGPT are the only channels where a money-moving server plausibly hits an explicit policy wall. That is the one place where reading the rule text before spending effort is mandatory rather than optional.

## Search is a channel with different rules

The brief asks what makes a tool show up when an agent searches "service that does X". That channel is not a submission form; it is a retrieval problem, and the levers are different from a directory listing. Four things determine whether a server is retrievable by an LLM-driven search: an exact, literal name that matches how the service is described; a description that states the task in the buyer's words rather than the builder's; machine-readable surface (a registry entry, an `llms.txt`, a docs page) so the crawler has something to quote; and citations from pages that already rank for the task. The last one is the reason a registry entry compounds: catalogues and docs pages cite it, and those citations are what a retrieval system reads. None of this was measured this round — no search-rank or referral data was available — so the row is marked unverified. What can be said with confidence is that a payment-moving server has an extra constraint: its description must make the money flow explicit (who pays, how much, and when), because a retrieval system that omits the price will surface it for jobs it cannot actually price.

The other three channels are smaller than they look. An awesome-list is a pull request against a curated Markdown file, so the cost is one PR per list and the benefit is a durable link, not a traffic spike. A package on npm or PyPI only helps if the package is actually installed, which means an SDK wrapper plus docs, not a name reservation. Hugging Face Spaces is a demo-hosting surface, not a payment surface, so it is a poor fit for a server that moves money. All three are cheap enough to do after the registry and the catalogues, and none of them should come before.

## Why the catalogue layer matters more than it looks

A catalogue entry is not just a link. It is a machine-readable claim that the server exists, what it does, and how to reach it. That is exactly the input an agent needs when it is deciding which tool to call, and it is why the registry outranks the human-facing catalogues: it is consumed by software, not by a person browsing a page. The second-order effect is that catalogues crawl each other, so a registry entry propagates. The cost of the first two layers is a few hours and no money; the cost of the last two is weeks and a policy risk. Spending the cheap hours first is the correct order regardless of how the later channels turn out.

There is also a measurement problem. None of these channels publishes installs per listing, so the only way to know which one works is to tag the entry point and count referrals after the fact. Without that instrumentation the ranking above stays an estimate forever, and the next decision is made on the same guess.

## Gaps

- Official registry publish requirements (authentication, ownership proof, schema validation, payment-server eligibility) were not fetched.
- Glama submission terms, review time, and payment-server policy were not fetched.
- Smithery's correct submission URL returned 404 and PulseMCP returned 403, so neither path is established.
- Claude connector directory and ChatGPT apps policies on payments were not read.
- x402 Bazaar and A2A directory submission mechanics were not fetched.
- No install or usage number was measured for any channel; the ranking is an estimate based on catalogue size, effort and how directly each channel is consumed by agents.

## What Agent Market should do next

1. **Publish to the official MCP registry first**, because it is the canonical machine-readable index and other catalogues consume it; the registry API already returns remote streamable-HTTP servers with the same shape as Agent Market's `/mcp` endpoint. Evidence: the registry response above.
2. **List on Glama and mcp.so's free tier the same day**, because both are large, self-service and cost nothing; skip the $39 paid placement at zero capital. Evidence: Glama's 114,033-server catalogue and mcp.so's free submission form.
3. **Verify framework ecosystem size before writing any integration**, then ship one package plus a docs example for the largest; the effort is days, so the choice must be made on measured download counts, not on impression. Evidence: no ecosystem measurement exists in this pack, so this step is explicitly gated on one.
4. **Read the Claude and ChatGPT payment policies before spending any effort there**, because a money-moving server may be disallowed outright and the review cycle is weeks. Evidence: neither policy was read; that is the open risk.
5. **Instrument referral installs per channel from the first listing**, because no channel publishes per-listing installs and the ranking above is otherwise unfalsifiable. Evidence: no public per-channel install data was found.
