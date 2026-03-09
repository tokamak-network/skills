---
name: twitterapi-cli
description: How to use the twitterapi-cli command-line tool to interact with Twitter (X). Use this skill whenever the user asks to search tweets, look up Twitter user profiles, post tweets, check trends, explore communities or spaces, look up lists, or do anything related to Twitter/X data. Also use this skill when the user mentions twitterapi-cli by name, asks about Twitter API queries, wants to find out what someone tweeted, wants to check follower relationships, or needs to retrieve tweet replies, quotes, or threads.
---

# twitterapi-cli

A CLI tool that wraps the [TwitterAPI.io](https://twitterapi.io) REST API. It lets you search tweets, look up users, post tweets, explore communities, spaces, lists, and trends — all from the terminal.

**Binary location:** `twitterapi-cli`
If the binary is not on `$PATH`, check the project directory at `./target/release/twitterapi-cli` or `./target/debug/twitterapi-cli`.

## Credential Security — Important for AI Agents

Credentials should already be stored in the system keychain via `setup`. **Never pass API keys, login cookies, or proxy URLs as CLI flags or environment variables.** When an AI agent passes secrets as `--api-key <KEY>` or sets `export TWITTER_API_KEY=...`, those values appear in shell history, process listings, agent context windows, and generated outputs — creating a serious leak risk.

If a command fails with "API Key is not configured", ask the user to run `setup` themselves in a separate terminal:

```
twitterapi-cli setup api-key <KEY>
twitterapi-cli setup login <COOKIES>
twitterapi-cli setup proxy <URL>
```

The agent must not run `setup` directly because doing so would expose the secret value in the agent's context.

## Always Use `--json`

When executing commands as an AI agent, always include the `--json` flag. This outputs the raw API JSON response instead of human-readable formatted text, making it possible to parse and reason about the data programmatically.

```bash
twitterapi-cli --json tweet search "from:elonmusk"
```

Without `--json`, results are formatted with emojis and summaries meant for humans. Commands that already return JSON by default (`me`, `space`, `trend`, `tweet article`, `tweet retweeters`, `user about`, `user batch`, `user check-follow`) are unaffected by the flag.

## Commands Reference

### tweet — Search and retrieve tweets

```bash
# Advanced search (supports full Twitter search syntax)
twitterapi-cli --json tweet search "<query>" [--query-type Latest|Top] [--cursor <cursor>]
# Examples:
#   "from:elonmusk bitcoin since:2024-01-01"
#   "web3 lang:en"
#   "#ethereum filter:media"

# Get tweet(s) by ID (comma-separated for multiple)
twitterapi-cli --json tweet get "<tweet_ids>"

# Get article attached to a tweet
twitterapi-cli --json tweet article "<tweet_id>"

# Get quote tweets of a tweet
twitterapi-cli --json tweet quotes "<tweet_id>" [--since-time <unix>] [--until-time <unix>] [--include-replies] [--cursor <cursor>]

# Get replies to a tweet
twitterapi-cli --json tweet replies "<tweet_id>" [--since-time <unix>] [--until-time <unix>] [--cursor <cursor>]

# Get replies with sort options
twitterapi-cli --json tweet replies-v2 "<tweet_id>" [--query-type Relevance|Latest|Likes] [--cursor <cursor>]

# Get users who retweeted a tweet
twitterapi-cli --json tweet retweeters "<tweet_id>" [--cursor <cursor>]

# Get full thread context of a tweet
twitterapi-cli --json tweet thread "<tweet_id>" [--cursor <cursor>]
```

### user — Look up user profiles and relationships

```bash
# Get user profile
twitterapi-cli --json user info <username>

# Get detailed "About" section
twitterapi-cli --json user about <username>

# Batch lookup by numeric user IDs
twitterapi-cli --json user batch "<user_ids>"   # comma-separated

# Followers / Followings
twitterapi-cli --json user followers <username> [--cursor <cursor>] [--page-size 20..200]
twitterapi-cli --json user followings <username> [--cursor <cursor>] [--page-size 20..200]

# Blue Verified followers (requires numeric user ID, not username)
twitterapi-cli --json user verified-followers "<user_id>" [--cursor <cursor>]

# User's recent tweets
twitterapi-cli --json user tweets <username> [--user-id <id>] [--include-replies] [--cursor <cursor>]

# Mentions of a user
twitterapi-cli --json user mentions <username> [--since-time <unix>] [--until-time <unix>] [--cursor <cursor>]

# Search users by keyword
twitterapi-cli --json user search "<query>" [--cursor <cursor>]

# Check if source follows target
twitterapi-cli --json user check-follow <source_username> <target_username>
```

### post — Post a new tweet

Requires API key, login cookies, and proxy to all be configured.

```bash
twitterapi-cli --json post "<tweet_text>"
```

### community — Explore Twitter communities

```bash
twitterapi-cli --json community info "<community_id>"
twitterapi-cli --json community members "<community_id>" [--cursor <cursor>]
twitterapi-cli --json community moderators "<community_id>" [--cursor <cursor>]
twitterapi-cli --json community tweets "<community_id>" [--cursor <cursor>]
twitterapi-cli --json community search "<query>" [--query-type Latest|Top] [--cursor <cursor>]
```

### list — Query Twitter lists

```bash
twitterapi-cli --json list followers "<list_id>" [--cursor <cursor>]
twitterapi-cli --json list members "<list_id>" [--cursor <cursor>]
```

### space — Get Space details

```bash
twitterapi-cli --json space "<space_id>"
```

### trend — Get trending topics

```bash
twitterapi-cli --json trend [--count <number>]   # default: 30
```

### me — Check API account info

```bash
twitterapi-cli --json me
```

## Pagination

Many endpoints return paginated results. When `has_next_page` is `true` in the response, pass the `next_cursor` value to `--cursor` to fetch the next page:

```bash
# First page
twitterapi-cli --json tweet search "web3"

# Next page (use next_cursor from previous response)
twitterapi-cli --json tweet search "web3" --cursor "<next_cursor_value>"
```

## Twitter Search Syntax

The `tweet search` command supports Twitter's advanced search operators:

| Operator | Example | Description |
|----------|---------|-------------|
| `from:` | `from:elonmusk` | Tweets by a specific user |
| `to:` | `to:elonmusk` | Replies to a specific user |
| `since:` | `since:2024-01-01` | Tweets after a date |
| `until:` | `until:2024-12-31` | Tweets before a date |
| `filter:media` | `bitcoin filter:media` | Only tweets with media |
| `filter:links` | `web3 filter:links` | Only tweets with links |
| `lang:` | `AI lang:en` | Tweets in a specific language |
| `min_retweets:` | `bitcoin min_retweets:100` | Minimum retweet count |
| `min_faves:` | `ethereum min_faves:500` | Minimum like count |
| `#hashtag` | `#DeFi` | Tweets with a hashtag |
| `"exact phrase"` | `"proof of stake"` | Exact phrase match |
| `-keyword` | `bitcoin -scam` | Exclude a keyword |

Operators can be combined: `from:vaborsh bitcoin since:2024-06-01 lang:en min_faves:10`

## Error Handling

| Error | Meaning | Action |
|-------|---------|--------|
| API Key is not configured | No API key found in keychain, env, or flag | Ask the user to run `setup api-key` |
| Login Cookies is not configured | Missing login cookies (needed for `post`) | Ask the user to run `setup login` |
| Proxy is not configured | Missing proxy (needed for `post`) | Ask the user to run `setup proxy` |
| API error (401) | Invalid or expired API key | Ask the user to verify their key |
| API error (429) | Rate limit exceeded | Wait and retry later |
