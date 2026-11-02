# How "hire a specialist agent" becomes a tool inside agent frameworks

**Answer.** The shortest path is a **remote MCP server**, because MCP is the only interface present in every row and none requires a bespoke plugin, but MCP solves transport only — it does not solve discovery or payment. For reach per unit of work, the best order is: (1) one remote MCP endpoint plus a Claude Code plugin repository and an entry in the official MCP Registry — two commands (`marketplace add`, then `plugin install`), no review gate, on a client with 12.1M weekly npm downloads [4] (a reach proxy, not a user count) [A]; (2) an n8n community node, because 14,113 packages already ship under the community-node keyword [20] and n8n's built-in MCP Client Tool can call the same endpoint with no node at all; (3) documentation-level support for OpenAI Agents SDK and Vercel AI SDK, where the developer pastes a URL and a header. Payment cannot ride on any framework primitive — none of them bills third-party tool calls — so the endpoint must return an HTTP 402 quote or debit a prepaid balance and report it in the tool result. Finally, the call must return a `job_id` rather than block, because default client timeouts are seconds and a marketplace job takes hours.

*All URLs were opened on 2026-11-02. Every number was fetched from the cited source; **[A]** marks analysis or judgement rather than a vendor statement.*

## 1. Method

I read each framework's documentation for install, MCP transport, OAuth, secrets and discovery. Size figures come from live APIs on 2026-11-02: GitHub stargazers, npm last-week downloads (2026-10-23 to 10-29), PyPI `pypistats` and the npm keyword total.

## 2. Comparison table

| Framework | Install mechanism | Remote MCP | OAuth | Secret / payment config | Discovery path | Ecosystem size |
|---|---|---|---|---|---|---|
| **OpenAI Agents SDK** | `pip install openai-agents` [1] | Yes: `HostedMCPTool`, `MCPServerStreamableHttp`, `MCPServerSse`, `MCPServerStdio` [2] | Not documented on the MCP page (no "OAuth" string in the extracted text); hosted tools forward *connector metadata* to the Responses API, so connector auth is OpenAI-side; local servers use an `Authorization` header [2] | Python args: `headers`, `params["auth"]`; `OPENAI_API_KEY` for the model [1] | None in-SDK; the developer must already know the URL [A] | 30,440 stars (py), 3,958 (js); PyPI 4,453,132/wk; npm `@openai/agents` 2,384,378/wk [3][4][5] |
| **Claude Agent SDK + Claude Code skills** | `pip install claude-agent-sdk` [6]; a skill is a `SKILL.md` at `.claude/skills/<name>/SKILL.md` [7] | Yes: `mcp_servers` with `type:"http"`, stdio, or in-process servers; tools appear as `mcp__<server>__<tool>` [8] | Spec-level OAuth 2.1, but **the SDK does not open a browser or run an interactive flow** — a 401 marks the server `needs-auth`/`failed`; you must pass a token [8] | Env vars (`${API_TOKEN}` expands at runtime) or headers; no billing primitive [8] | Plugin marketplaces: `claude plugin marketplace add <owner>/<repo>`, then `claude plugin install <name>@<marketplace>` — a git repo, no central review [9]. Official marketplace ships 12 plugins [10] | Claude Code 154,171 stars, npm CLI 12,083,369/wk; SDK npm 17,096,839/wk, PyPI 8,093,087/wk [3][4][5] |
| **LangChain / LangGraph** | `pip install "langchain[mcp]"` [11] | Yes: `MCPAdapter` with `list_tools()`, used as a context manager; docs warn the API may change and describe migrating from `langchain-mcp-adapters` [11] | Yes — docs list Bearer tokens, OAuth 2.1, and per-user server auth [11] | Constructor args; no billing primitive [11] | Integration docs + GitHub; no first-party paid-tool marketplace verified [A] | 148,839 / 44,286 stars; PyPI 37,058,402 and 13,602,612/wk; npm `langchain` 4,151,182/wk [3][4][5] |
| **CrewAI** | `uv add mcp`, or `pip install 'crewai-tools[mcp]'` for the adapter [12] | Yes: `mcps` field on agents (`MCPServerStdio`, `MCPServerSSE`, streamable HTTP); advanced `MCPServerAdapter` [12] | Not documented there; auth shown as `headers={"Authorization":"Bearer ..."}` [12] | Crew YAML/Python; no billing primitive [12] | Docs tool catalog; no paid marketplace verified [A] | 60,393 stars; PyPI 589,367/wk [3][5] |
| **AutoGen** (archived) | `pip install -U "autogen-ext[mcp]"` [13] | Yes: `mcp_server_tools()` plus stdio/SSE/`StreamableHttpServerParams` adapters [13] | Not verified | `server_params` (command/URL, headers, timeouts) [13] | None | 61,663 stars; PyPI `autogen-agentchat` 102,985/wk [3][5]. README: "transitioning to archived… advise against production use"; new users are sent to Microsoft Agent Framework [14] |
| **Microsoft Agent Framework** (AutoGen successor) | Python/.NET packages in `microsoft/agent-framework` | Yes: `MCPStreamableHTTPTool`, OpenAPI, native functions [15] | A2A + MCP hosting samples; OAuth specifics not verified | Plugin/extension model; no billing primitive verified | None; PyPI/NuGet + docs [A] | 14,482 stars [3]; README describes a 1.0 production release with long-term support [15] |
| **Google ADK** | `pip install google-adk` [16] | Yes: `McpToolset` with `StdioConnectionParams`, `StreamableHTTPConnectionParams`, `SseConnectionParams` [16] | Yes, a documented browser flow: "grant the agent access" in the browser; a token header path also exists [16] | Python tool config, headers/env vars [16] | None in ADK; docs demonstrate third-party hosted MCP servers (PayPal, Google Maps) [16] | 22,074 stars; PyPI 2,448,880/wk; npm `@google/adk` 282,962/wk [3][4][5] |
| **Vercel AI SDK** | `npm install @ai-sdk/mcp` (v2.0.88 published); docs show the import [17] | Yes: `createMCPClient`, HTTP transport recommended, SSE and official `StreamableHTTPClientTransport`; session reattach via `initialSessionId` [18] | Yes: optional `authProvider` "for automatic authorization" [18] | Transport `headers`; no billing primitive [18] | None; npm + docs [A] | `ai` 41,889,127/wk; `@ai-sdk/mcp` 7,965,102/wk; 27,680 stars [3][4] |
| **n8n** | Self-host (npm/Docker) or cloud; community node installed from npm [19] | Yes, both directions: **MCP Client**, **MCP Client Tool** for AI Agents, and **MCP Server Trigger** [19] | Yes — the MCP Client node supports Bearer, generic/multiple headers and OAuth2 [19] | n8n credential store + node fields; billing is the n8n plan [19] | npm keyword `n8n-community-node-package`: **14,113 packages** [20] | 209,300 stars; npm `n8n` 159,754/wk [3][4] |
| **Zapier** (hosted MCP server) | Nothing to install; connect a client to Zapier's hosted endpoint and sign in [21] | Yes: all clients share one endpoint; **Streamable HTTP only, no SSE** [21] | Yes: "Zapier handles OAuth, credentials, and rate limiting" [21] | OAuth into the user's Zapier account; **each successful tool call costs two tasks; failed calls are free** [21] | Zapier's own catalog: 9,000+ apps, 40,000+ actions [21]. A third party cannot publish a tool into that server without building a Zapier app [A] | Free 100 tasks/mo; Professional from $19.99/mo [22] |
| **Make** (official MCP server) | Nothing to install client-side; connect an MCP client [23] | Yes: Claude and ChatGPT are named clients; **scenarios become tools** and must be active with on-demand scheduling [23] | Yes: OAuth consent with scopes; scenario-run scopes on all plans, management scopes on paid plans; only OAuth can restrict to one organization [23] | OAuth scopes; Make operations/credits billing [23] | Make's own scenario/app catalog; publishing a third-party tool means building a Make app [A] | 2,000+ apps [26] |

Two facts cut across the table: MCP is the only interface in all eleven rows (nine are MCP clients; Zapier and Make are hosted MCP servers) [A], and **none ships a payment primitive** — the tool must handle money itself (API key + prepaid balance, or HTTP 402/x402 at the boundary) [A].

## 3. Ecosystem numbers

Every size figure used for ranking is in the Size column of §2: stars from the GitHub API, last-week downloads from the npm API (window 2026-10-23 to 2026-10-29) and PyPI `pypistats`, plus the n8n community-keyword total of 14,113 [3][4][5][20].

## 4. Paid third-party tool precedent

**Apify MCP server** (`https://mcp.apify.com`) is the closest analogue of "hire a specialist, paid per job" inside MCP. Billing, verbatim: "Connecting to the MCP server is free. Actor runs bill to your Apify account according to each Actor's pricing, which `fetch-actor-details` shows before a run. The Apify free plan includes $5 of monthly platform usage." It also ships an experimental agent-native path: a prepaid, **spend-capped** token sold for an x402, MPP, or Skyfire payment, used as a normal bearer token [24]. Adoption: 14,260 stars, 30,525 npm downloads/wk for `@apify/actors-mcp-server` [3][4]. A second precedent is Firecrawl's MCP server: credit-based paid API behind a key, 7,743 stars, 170,113 npm downloads/wk [3][4].

The pattern is **free connection, metered work, price shown before the run, hard spend cap** — not subscription, not per-seat [A].

Per-framework paid-tool evidence is thin. Verified: ADK documents a PayPal MCP server with browser OAuth [16]; Zapier and Make expose paid app catalogs of 9,000+ and 2,000+ apps [21][26]; Firecrawl's credit-billed MCP server is the paid example used with Claude-family clients (170,113 npm downloads/wk) [4]. For OpenAI, LangChain, CrewAI, AutoGen, Vercel and n8n I found no framework-specific adoption figure for a paid third-party tool — those frameworks provide transport, not a billing or listing layer [gap].

## 5. Three integrations with the best reach-to-effort ratio

All three reuse one remote MCP endpoint; only the packaging differs.

### 5.1 Claude Code plugin + MCP Registry entry — least work, no review gate
**Reach:** 154,171 stars, 12,083,369 npm CLI downloads/wk [3][4]; two-command install, no app review [9].

**What installs**
```
repo/
  .claude-plugin/marketplace.json        # {name, owner, plugins:[{name, source}]}
  plugins/hire-specialist/.claude-plugin/plugin.json
  plugins/hire-specialist/skills/hire-specialist/SKILL.md
  plugins/hire-specialist/.mcp.json      # {"mcpServers":{"market":{"type":"http","url":"https://market.near.ai/mcp"}}}
```
Developer runs `claude plugin marketplace add <owner>/repo` then `claude plugin install hire-specialist@<owner>` [9]. A free record in the official MCP Registry (`registry.modelcontextprotocol.io`, schema `2025-12-11`, entries carry `remotes[].url`) is the listing path MCP clients can read [25][A].

**Skill text** is the real discovery mechanism: frontmatter `name` + `description` stating *when* to use it — "delegate research, translation, extraction, or review to a marketplace agent; costs USDC per job; confirm budget with the user first."

**Tool contract** [A]
- `hire_specialist({specialty, brief, deadline_minutes, budget_usdc})` → `{job_id, state, price_usdc, escrow:"held", eta_minutes}`
- `get_job({job_id})` → `{state, result_inline?, result_url?, worker_id?, refunded?}`
- `list_jobs({since, state?})`, `cancel_job({job_id})` (refund only while `posted`)

**Developer configures:** the plugin, or `mcp_servers` + `allowed_tools:["mcp__market__*"]` without it [8].
**Payment:** 402 quote on first call, paid in Base USDC and replayed with a receipt, or prepaid credit debited per call with `balance_usdc` returned [A]. The SDK cannot run OAuth in-process [8], so bearer-token-first is the only unattended path.
**Async:** `hire_specialist` returns within a second with `job_id`; the agent polls `get_job` on later turns.

### 5.2 n8n community node — widest low-code surface
**Reach:** 14,113 packages use the community-node keyword [20]; 209,300 stars, 159,754 npm downloads/wk [3][4].

**What installs:** npm package `n8n-nodes-agent-market` with `keywords:["n8n-community-node-package"]`, a node exposing `hire` / `status` / `cancel`, and a credential `marketApi` (`serverUrl`, `apiKey`, `walletAddress`). Install via Settings → Community Nodes or `npm install` on self-hosted [19]. A no-code path already exists: the built-in **MCP Client Tool** node points at the same endpoint with no custom node [19].

**Developer configures:** credential plus three fields; output maps one-to-one to the tool result.
**Payment:** prepaid credit or 402; the node surfaces `balance_usdc` because n8n has no billing hook [A].
**Async:** `hire` returns a `job_id`; a second node polls, or the workflow parks on a Wait node [A].

### 5.3 Zero-install remote MCP for code SDKs (OpenAI Agents SDK, Vercel AI SDK)
**Reach:** OpenAI 4,453,132 PyPI + 2,384,378 npm downloads/wk; Vercel `ai` 41,889,127 and `@ai-sdk/mcp` 7,965,102 [4][5].

**What installs:** nothing of ours; snippets in each framework's idiom: `MCPServerStreamableHttp(params={"url":..., "headers":{"Authorization":"Bearer ..."}})` [2] and `createMCPClient({transport:{type:"http", url:..., headers:{Authorization:"Bearer ..."}}})` [18]; Vercel also accepts `authProvider` [18].

**Developer configures:** URL + header. **Tool contract:** `hire_specialist` / `get_job` / `list_jobs` / `cancel_job`, arguments as §5.1; the snippet only sets transport. **Result:** `job_id` on the first call, then `get_job` returns `{state, result_inline|result_url}` [A]. **Payment:** 402/credit; every result carries `price_usdc` and `balance_usdc` because these SDKs have no billing layer [A].

## 6. What the developer's agent sees when the task outlives a tool call

| Moment | Returned to the agent | Design rule |
|---|---|---|
| Accepted | `{job_id, state:"posted", price_usdc, eta_minutes}` | Never hold the call open: AutoGen's adapter default is 30 s, and sessions can end mid-call [13][A] |
| Picked up | `state:"running"`, `worker_id`, `accepted_at` | `job_id` is the only handle the agent needs |
| Later turn | `get_job` re-poll, or a webhook registered by the host app | Do not assume the session survives |
| Delivered | `state:"delivered"` + `result_inline` (≤4 KB) or `result_url` + `sha256` | Inline small results; agents cannot fetch URLs unaided |
| Failed/expired | `state:"refunded"`, `refund_usdc`, `reason` | Visible refunds stop double-payment retries |
| Retry | Same `job_id` for the same idempotency key | Agents retry aggressively [A] |

## 7. Gaps

- **No framework hosts a paid-tool marketplace.** The MCP Registry lists servers with no payment or ranking and exposes no corpus count (`limit=1` returns `metadata.count:1`) [25].
- **OAuth is uneven**: spec support is not unattended support (Claude SDK [8], ADK [16], Vercel [18] all differ).
- **Install paths move**: LangChain warns its MCP API may change [11].
- **AutoGen is archived**, advises against production use, and points to Microsoft Agent Framework [14][15].
- **No revenue-share evidence** exists for third-party paid tool calls; absence of evidence, not proof of absence.
- **Not measured:** SKILL.md context cost, tool-schema token overhead, installed-to-called conversion.
- **Make**: app count verified, third-party publish path not [A].
- **Our payment path is untested**: x402 on Base is an assumption; only Agent Market's own USDC withdrawal has been exercised.

## 8. What Agent Market should do next

1. **Ship one remote MCP endpoint that works with a pasted token and an HTTP 402 quote, and treat OAuth as an add-on.** Evidence: Claude Agent SDK will not open a browser and reports `needs-auth` on a 401 [8]; Vercel adds `authProvider` [18]; ADK documents a browser flow [16]. A tool that requires interactive OAuth cannot be called by half these clients unattended.
2. **Publish a Claude Code plugin repository plus an MCP Registry record first.** Evidence: two commands to install, no review gate [9], on a client with 12.1M weekly CLI downloads [4]; Anthropic's own marketplace ships 12 plugins [10], so third-party supply is unmeasured [gap].
3. **Publish an n8n community node second, and make the endpoint work with n8n's built-in MCP Client Tool on day one.** Evidence: 14,113 existing community packages [20] and a 209k-star project; the client tool needs no custom node [19].
4. **Make the tool contract asynchronous and idempotent: return `job_id`, accept a client-supplied key, and report `balance_usdc` in every response.** Evidence: default timeouts of ~30 s [13] are shorter than any real job; agents retry on timeout [A].
5. **Do not spend early effort on Zapier or Make as distribution.** Evidence: both expose *their own* app catalogs to MCP clients, so a third-party tool must become a Zapier or Make app; Zapier also bills two tasks per successful call and needs a Team plan to share a server [21][22][23]. The effort is partnership-shaped, not packaging-shaped.

## Appendix: source URLs (opened 2026-11-02)

1. https://openai.github.io/openai-agents-python/quickstart/
2. https://openai.github.io/openai-agents-python/mcp/
3. https://api.github.com/repos/openai/openai-agents-python — one request per repo in the tables
4. https://api.npmjs.org/downloads/point/last-week/@anthropic-ai/claude-agent-sdk
5. https://pypistats.org/api/packages/openai-agents/recent
6. https://raw.githubusercontent.com/anthropics/claude-agent-sdk-python/main/README.md
7. https://docs.claude.com/en/docs/claude-code/skills
8. https://docs.claude.com/en/api/agent-sdk/mcp
9. https://docs.claude.com/en/docs/claude-code/plugin-marketplaces
10. https://raw.githubusercontent.com/anthropics/claude-code/main/.claude-plugin/marketplace.json
11. https://docs.langchain.com/oss/python/langchain/mcp
12. https://docs.crewai.com/en/mcp/overview
13. https://microsoft.github.io/autogen/stable/reference/python/autogen_ext.tools.mcp.html
14. https://raw.githubusercontent.com/microsoft/autogen/main/README.md
15. https://raw.githubusercontent.com/microsoft/agent-framework/main/README.md
16. https://adk.dev/llms-full.txt
17. https://registry.npmjs.org/@ai-sdk/mcp/latest
18. https://ai-sdk.dev/docs/ai-sdk-core/mcp-tools
19. https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-langchain.mcpclient/
20. https://registry.npmjs.org/-/v1/search?text=keywords:n8n-community-node-package
21. https://docs.zapier.com/mcp/home
22. https://zapier.com/pricing
23. https://help.make.com/make-mcp-server
24. https://raw.githubusercontent.com/apify/apify-mcp-server/master/README.md
25. https://registry.modelcontextprotocol.io/v0/servers
26. https://www.make.com/blog/connect-to-anything-with-an-api
