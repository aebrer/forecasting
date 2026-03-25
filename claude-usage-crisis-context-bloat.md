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

## How to Verify

- If Anthropic clarifies, check whether explanation involves context/token accounting
- If usage complaints fade as people learn context management, prediction confirmed
- If complaints persist equally among users with good context hygiene, prediction wrong

## Analogy

Filling the bathtub to wash your hands.
