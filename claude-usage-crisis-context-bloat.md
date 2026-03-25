# Claude Usage "Crisis" is Context Window Misuse

*Conversation between Drew and Claude, 2026-03-25*

## Prediction

The March 2026 wave of Reddit complaints about Claude burning through Pro/Max usage limits in 2-3 prompts is primarily caused by users accumulating massive contexts in the new default 1M token window — not by Anthropic secretly throttling or running a "fractional reserve."

**Drew's confidence:** Hat-eating stakes.

## Evidence

### Timing
- Complaints correlate with the 5x context window expansion (200k → 1M)
- Multiple users in the threads say "this started after the new update" and "I used to code for hours without hitting limits"

### User Behavior
- The "open letter" post frames analyzing a **100,000-token document** as a "routine task" — that's half the old context window before a single prompt is typed
- One user loaded 49 markdown files in full, wondered why 3 prompts ate their quota
- Free-plan user had "a big instruction block" in project context, never compacted
- Nobody complaining has posted session logs (which would reveal the context bloat)

### Control Group
- Users reporting zero issues consistently describe good context hygiene: `/compact`, fresh sessions, scoped tasks
- One user: "80+ hours a week, no issues"
- The people hit hardest are likely those with the *least* reason to have managed context historically (Max plan users with unlimited-feeling quotas)

### The Mechanism
Each prompt round-trips the entire context window. If a user lets context grow to 500k+ tokens, every subsequent prompt is re-sending all of it through the usage meter. 3 prompts × 500k context ≈ 1.5M tokens consumed — enough to drain a Pro quota.

## Suspected Astroturfing

- Suspicious pattern of "claude sucks, switch to Codex/GPT" recommendations in threads
- AI-generated "open letter" (author admitted using Gemini) framed as grassroots community movement
- One commenter noted: "Is reddit getting astroturfed? I'm not subscribed to any AI subs, yet see tons of 'claude sucks, codex is better' posts in my home feed"
- Same playbook as NFT/AI-art "debate" — legitimate frustration exists, gets amplified and steered by actors who benefit from the narrative

## How to Verify

- If Anthropic clarifies, check whether explanation involves context/token accounting
- If usage complaints fade as people learn context management, prediction confirmed
- If complaints persist equally among users with good context hygiene, prediction wrong

## Analogy

Filling the bathtub to wash your hands.
