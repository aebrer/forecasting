# Claude Usage "Crisis" is Context Window Misuse

*Conversation between Drew and Claude, 2026-03-25*

## Prediction

The March 2026 wave of Reddit complaints about Claude burning through Pro/Max usage limits in 2-3 prompts is primarily caused by users accumulating massive contexts in the new default 1M token window — not by Anthropic secretly throttling or running a "fractional reserve."

**Drew's confidence:** Hat-eating stakes.

## Sources

- **Thread 1 (free plan user):** [r/ClaudeAI — "I'm out of tokens with just 3-4 prompts, need advice to use efficiently please"](https://www.reddit.com/r/ClaudeAI/comments/1jwwksd/) by u/MiserableBus8139 (59 upvotes, 214 comments)
- **Thread 2 (open letter):** [r/ClaudeCode — "Open Letter to the CEO and Executive Team of Anthropic"](https://www.reddit.com/r/ClaudeCode/comments/1jx3j47/) by u/onimir3989 (238 upvotes, 161 comments)

## Evidence

### Timing
- Complaints correlate with the 5x context window expansion (200k → 1M)
- Multiple users in the threads say "this started after the new update" and "I used to code for hours without hitting limits"

### User Behavior

**Thread 1** — OP is on the free plan, vibe-coding an entire web app. Describes loading "a big instruction block" into project context and hitting 73% usage in 3 prompts. In comments, mentions instructing Claude to read 49 markdown files in full. Also says:

> "2 days ago, i literally had an entire convo than i've ever had with any girl in my entire life [...] so yes it was a really long 'single' chat not multiple broken one's"

**Thread 2** — The "open letter" frames the following as a routine task:

> "Feeding Sonnet or Opus routine tasks — such as analyzing a 100,000-token document and managing a few dozen Markdown files — maxes out limits in just three prompts."

100k tokens is half the old context window. Add "a few dozen Markdown files" on top and you're at 150-200k tokens of context before typing a single prompt. Every subsequent message round-trips all of it.

- Nobody complaining has posted session logs (which would reveal the context bloat)

### Control Group
- Users reporting zero issues consistently describe good context hygiene: `/compact`, fresh sessions, scoped tasks
- u/saintpetejackboy (Thread 2): "80+ hours a week, no issues. [...] Where are the screenshots and logs of them burning up a week data in one session? I have NOT seen any of these people show anything to back up their claims about usage."
- u/Last_Magazine2542 (Thread 2): "Have you ever used /compact or /clear, or started a fresh instance? How many mcp servers and tools are you running? There is just no way you are using Claude correctly and hitting limits that fast."
- u/OverSoft (Thread 2): "No, I have no issues with a medium workload, using Claude (Code and Cowork) about 8 hours a day on a 20x plan. After 5 days, I'm currently at about 20% of my week limit and haven't hit session limits all week."
- The people hit hardest are likely those with the *least* reason to have managed context historically (Max plan users with unlimited-feeling quotas)

### The Mechanism
Each prompt round-trips the entire context window. If a user lets context grow to 500k+ tokens, every subsequent prompt is re-sending all of it through the usage meter. 3 prompts × 500k context ≈ 1.5M tokens consumed — enough to drain a Pro quota.

### Our Own Usage
In the same session where we read both full Reddit threads, discussed them, wrote files, initialized a git repo, and pushed to GitHub — we were at 8% of a 5-hour usage window on the 5x Max plan, using Opus 4.6 with a 1M context window. Context hygiene works.

## Suspected Astroturfing

- Suspicious pattern of "claude sucks, switch to Codex/GPT" recommendations across threads
- Thread 2's "open letter" was AI-generated (OP admitted it was written by Gemini), framed as a grassroots community movement with calls to "send thousands of emails"
- u/Quiet_Progress7931 (Thread 2): "Is reddit getting astroturfed? I'm not subscribed to any AI subs, yet see tons of 'claude sucks, codex is better' posts in my home feed."
- u/PandorasBoxMaker (Thread 2): "I'm somewhat convinced that this is a gorilla social media campaign by OpenAI"
- Same playbook as NFT/AI-art "debate" — legitimate frustration exists, gets amplified and steered by actors who benefit from the narrative

## Caveat

If there *is* a genuine bug, it's most likely a caching bug — the 1M context expansion may have broken assumptions in how cached/pre-filled tokens are counted against usage. A system that previously cached 200k of context and only billed the delta might now be failing to cache entirely, re-billing the full window on each turn, or double-counting cached tokens. This would be consistent with the timing (correlates with the context window change) and the symptoms (sudden multiplier on usage without changed behavior). But even in this case, the users loading 100k+ token files are the ones who'd feel it most.

## Update: 2026-03-26 — Anthropic Confirms Peak-Hour Throttling

**Source:** [r/ClaudeCode — "Update on Session Limits"](https://www.reddit.com/r/ClaudeCode/comments/1jxunique/) by u/ClaudeOfficial (71 upvotes, 107 comments)

Anthropic officially announced they are adjusting 5-hour session limits during peak hours (weekdays 5am–11am PT / 1pm–7pm GMT). Users burn through session limits faster during peak; weekly limits remain unchanged. They estimate ~7% of users will be affected.

### What this means for the prediction

**Still holds:**
- Context bloat remains the best explanation for *why* some users drain limits in 1-3 prompts while others on the same plan are fine. The "control group" pattern persists in this thread too — users with good context hygiene report no issues.
- Nobody complaining has posted session logs or token breakdowns.

**Weakened:**
- Anthropic is confirming a real, deliberate server-side change to rate limiting — it's not *purely* user-side context mismanagement.
- Multiple commenters (and the timing) strongly suggest silent A/B testing was happening in the days before this announcement, which means some of the complaints we attributed to context bloat may have been genuine throttling.
- Max subscribers ($200/mo) reporting full weekly limits drained in 1 prompt is hard to explain with context bloat alone, even with 1M windows.
- Several users directly dispute Anthropic's "just peak hours, weekly unchanged" framing — u/RedEagle182: "The weekly limits were filled as quickly as the 5h session ones, so this is false." u/CreativetechDC: "Verified that weekly usage is also severely affected." If weekly limits are also being hit harder, the peak-hour redistribution explanation doesn't fully account for it.

**Assessment:** The prediction was probably *partially right* — context bloat is a real amplifier and explains the variance between users — but it missed that Anthropic was also actively tightening the supply side. The caching bug caveat (line 59 of the original) was closer to the mark than the main thesis. Waiting for more dust to settle before calling it.

## Update: 2026-03-30 — Caching Bugs Confirmed via Reverse Engineering

**Source:** [r/ClaudeAI — "PSA: Claude Code has two cache bugs that can silently 10-20x your API costs"](https://www.reddit.com/r/ClaudeAI/comments/1jx_placeholder/) by u/skibidi-toaleta-2137 (306 upvotes, 51 comments)

A user reverse-engineered the Claude Code standalone binary (Ghidra + MITM proxy) and found two independent caching bugs that silently inflate token usage:

### Bug 1: Sentinel replacement breaks cache when conversation discusses billing internals

The standalone Claude Code binary (the one from `claude.ai/install.sh` or `npm install -g`) has a **native-layer string replacement** baked into Anthropic's custom Bun fork. On every API request, it searches for a billing attribution sentinel (`cch=eaab0`) and replaces part of it with a hash derived from the body. The replacement targets the **first** occurrence — so if the sentinel string appears in your conversation history (from reading CC source, discussing billing, or having it in CLAUDE.md), it modifies `messages[]` instead of `system[0]`, breaking the cache prefix. Every subsequent request becomes a full cache rebuild.

**Cost:** ~$0.04–0.15 per request on a 500k context. Workaround: use `npx @anthropic-ai/claude-code` instead of the standalone binary (the npm package runs on standard Bun/Node with no replacement).

### Bug 2: `--resume` / `--continue` always breaks cache (since v2.1.69)

A `deferred_tools_delta` feature introduced in v2.1.69 injects system-reminder attachments into different positions depending on fresh vs resumed sessions. On fresh sessions, ~13KB of tool/skill metadata goes into `messages[0]`; on resume, it's appended to `messages[N]` while `messages[0]` contains only ~352B. This creates three independent cache-breaking differences (different prefix content, different billing hash, different cache breakpoint position). Every resume/continue triggers a full cache miss on the first request.

**Cost:** ~$0.15 one-time per resume on a 500k context. No external workaround — pinning to v2.1.68 avoids it but loses 60+ versions of features.

### What this means for the prediction

**The caching bug caveat was correct.** The original prediction's main thesis was "context bloat explains everything." The caveat on line 59 said: *"If there is a genuine bug, it's most likely a caching bug — the 1M context expansion may have broken assumptions in how cached/pre-filled tokens are counted against usage."* That's essentially what was found.

The full picture is now three compounding factors:

1. **Context bloat** (user-side) — users letting context grow to 500k+ tokens in the 1M window, then every prompt round-trips all of it. Still explains the variance between users who hit limits fast and users who don't.
2. **Caching bugs** (client-side) — the two bugs above mean Claude Code itself can silently 10-20x the cost of individual requests, even for users with reasonable context sizes. Users running via the standalone binary who discuss CC internals or frequently resume sessions get hit hardest.
3. **Peak-hour throttling** (server-side) — Anthropic confirmed deliberate rate limit tightening during peak hours, reducing the budget available to absorb the above.

All three are real, all three compound each other, and the combination explains why the complaints were so intense and so variable between users. A user with big context + standalone binary + frequent resumes + peak hours could easily see 20-50x the effective cost of a user with tight context + npx + fresh sessions + off-peak.

**Drew's assessment:** Not hat-eating territory — context bloat is real and was the right call as the primary amplifier — but not a clean win either. The prediction was right about the mechanism that explains *variance between users*, but the caching bugs and throttling are genuine platform-side issues that the main thesis dismissed too readily. The caveat was closer to the truth than the headline.

## How to Verify

- ~~If Anthropic clarifies, check whether explanation involves context/token accounting~~ — Clarification arrived (March 26), it's about peak-hour rate limiting. Caching bugs confirmed independently (March 30).
- ~~If complaints persist equally among users with good context hygiene, prediction wrong~~ — Caching bugs mean good context hygiene alone isn't sufficient; the harness itself introduces cost multipliers. Users with good hygiene *and* npx (not standalone) *and* no resume should be fine.
- Watch for Anthropic's response to the GitHub issues ([#40524](https://github.com/anthropics/claude-code/issues/40524), [#34629](https://github.com/anthropics/claude-code/issues/34629))
- Watch whether a fix for the resume bug meaningfully reduces complaints

## Analogy

Filling the bathtub to wash your hands, while the faucet has a leak that doubles your water bill, and the water company turned down the pressure during peak hours. *(Three problems, three different owners — user, client, server.)*
