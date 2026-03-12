# OpenClaw X Ingestion Playbook

This repo captures the current best-known workflow for ingesting X/Twitter links into a `bian`-hosted OpenClaw + Obsidian knowledge system.

It is intentionally practical:

- what works now
- what failed in real tests
- which components are worth keeping
- which components should only be fallbacks
- what the likely final solution should be

## Goal

Given an X/Twitter status URL, we want `bian` to:

1. accept the link automatically
2. resolve the real content target
3. extract durable text with the lowest possible cost
4. store the result in Obsidian BufferZone / KnowledgeLibrary
5. avoid burning Tavily credits unless strictly necessary

## Current Stack

### On `bian`

- OpenClaw
- Obsidian knowledge workflow
- `tavily-search`
- `search-layer`
- `opennews`
- `opentwitter`
- `memory-lancedb-pro`

### Current local wrappers on `bian`

- `/root/.openclaw/workspace/bin/intake-link-note`
- `/root/.openclaw/workspace/bin/intake-link-auto`
- `/root/.openclaw/workspace/bin/opennews-api`
- `/root/.openclaw/workspace/bin/opentwitter-api`
- `/root/.openclaw/workspace/bin/x-status-extract`

## What Works

### 1. Plain X status URL intake

`bian` can automatically create a standardized Obsidian note for a pasted link.

### 2. OpenTwitter user and tweet data

`opentwitter` is useful for:

- `twitter_user_info`
- `twitter_user_tweets`
- `twitter_search`

This is already better than using Tavily alone for X discovery because it can reliably identify:

- the author
- the exact status ID
- the linked `x.com/i/article/<id>` target

### 3. OpenNews for crypto/news monitoring

`opennews` is useful for:

- crypto news aggregation
- listings
- onchain alerts
- market anomaly feeds

It should be preferred over generic web search for crypto-news tasks.

### 4. Cheap pre-processing path

For many X links, the cheap path works partially:

1. `oEmbed`
2. extract `t.co`
3. resolve final target

This is low cost and worth keeping as a first-stage resolver.

## What Failed

### 1. `r.jina.ai` is not a reliable X Article reader

Real test result:

- `status -> t.co -> x.com/i/article/<id>` works
- `r.jina.ai/http://x.com/i/article/<id>` often returns:
  - `This page is not supported`
  - login wall content
  - a stale or unusable snapshot

Conclusion:

- keep it as an opportunistic tool
- do not treat it as the main extractor

### 2. Tavily as the first step is too expensive

Tavily works well for:

- discovering the correct X link
- getting a snippet

But using Tavily on every X link burns credits too fast.

Conclusion:

- Tavily should be a fallback, not the primary path

### 3. `twitter_article_by_id` is promising but unstable

`opentwitter` exposes `twitter_article_by_id`, which is exactly the right endpoint conceptually.

But in real tests it returned:

- `API返回错误: try again!`

Conclusion:

- keep it in the chain
- do not rely on it as the only article-content source yet

### 4. `search-layer` is not yet clearly better than `tavily-search`

With only Tavily configured, `search-layer` mainly behaves like:

- Tavily plus scoring / packaging

It becomes more valuable only after adding Exa and optionally Grok.

## Best Current Strategy

For X/Twitter status links:

1. detect `x.com/.../status/...`
2. resolve user + status via `opentwitter`
3. if the status points to an article, record the article ID / URL
4. if the tweet has enough visible text, store it directly
5. if not, try article retrieval
6. if article retrieval fails, use Tavily only as fallback

For crypto/news tasks:

1. prefer `opennews`
2. only fall back to generic web search if needed

## Recommended Final Architecture

### Stage 1: low-cost resolver

- `oEmbed`
- `t.co` resolution
- `opentwitter` status lookup

### Stage 2: structured data

- `opentwitter` for tweet/user/article metadata
- `opennews` for crypto/news streams

### Stage 3: expensive fallback

- Tavily

### Stage 4: final, robust solution

Use a logged-in browser session on `bian` to extract X Article DOM content directly.

This is the most likely path to a truly stable final solution because:

- it avoids repeated Tavily spend
- it avoids public reader limitations
- it operates on the same authenticated page the user can already see

## Current Recommendation

Keep this priority order:

1. `opentwitter`
2. `opennews` for crypto/news
3. cheap URL resolution
4. Tavily fallback
5. browser DOM extraction as the long-term final fix

## Next Steps

### High priority

- add browser-based X Article DOM extraction on `bian`
- make `intake-link-auto` use `opentwitter` first, then Tavily fallback
- store article URL even when正文提取失败

### Medium priority

- add Exa to `search-layer`
- add structured test cases for successful / failed X links

### Low priority

- revisit `twitter_article_by_id` when 6551 improves backend stability

