# Test Results

## Verified Components

### Tavily

Good at:

- finding correct X links
- returning useful snippets

Weak at:

- stable full-article extraction from X Articles
- cost efficiency if used on every link

## OpenNews

Verified:

- source list works
- latest news works
- news search works

Useful for:

- crypto news
- listings
- onchain
- market anomaly workflows

Not intended for:

- general X Article extraction

## OpenTwitter

Verified:

- `twitter_user_info`
- `twitter_user_tweets`
- `twitter_search`

Partially verified:

- `twitter_article_by_id`

Observed behavior:

- correct on user and status discovery
- useful for extracting exact `article` URL targets
- unstable when fetching article content directly

## Real Link Outcomes

### `wsl8297/status/2031654192817873253`

Observed:

- original status points to article
- plain public readers were inconsistent
- Tavily could extract a useful visible excerpt

Result:

- acceptable with fallback

### `onehopeA9/status/2031965027314647316`

Observed:

- `opentwitter` could locate the status and article target
- `twitter_article_by_id` returned transient backend errors
- Tavily fallback drifted to the wrong status in one test

Result:

- target resolution is solved
- article body extraction is not fully solved yet

