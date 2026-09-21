# Personal Reddit Digest

A **read-only** n8n workflow that checks the subreddits I follow, keeps the
posts that match my own interests, and sends me a short digest. That's all it does.

Workflow automation is a personal interest of mine as well as my line of work. I
read these threads to learn, and occasionally to answer a question I know something
about — by hand, as myself.

This repository exists so the behaviour of the app is auditable. It accompanies a
Reddit Data API access request.

---

## It never writes to Reddit

There is no node in this workflow that submits, comments, votes, messages, edits or
moderates anything on Reddit. The only scope used is `read`. No `submit`,
`privatemessages`, `vote`, `edit` or `modposting` scope is requested.

If I ever reply to one of these threads, I do it by hand in a browser, under my own
account — the same as any other Redditor.

---

## What it does

Runs on a schedule. Each run:

1. Authenticates with OAuth2 and reads `/r/{subreddit}/new` for each subreddit in my
   list — a plain public listing, one request per subreddit.
2. Scores each post against a keyword list I maintain (things like "how did you solve",
   "human in the loop", "state between runs", "show your workflow").
3. Drops anything stickied, older than 24 hours, or below my score threshold.
4. Builds a plain-text digest — title, link, which keywords matched, comment count —
   and sends it to my notification channel.
5. I read it. If something looks interesting I open the thread and read or reply
   manually.

The whole configuration — subreddits, keywords, threshold, max age — sits in one
editable block at the top of the **Config — My Interests** node.

---

## API etiquette

| | |
|---|---|
| User-Agent | `n8n:personal-reddit-digest:v1.0 (by /u/YOUR_USERNAME)` |
| Cadence | every 6 hours |
| Volume | one request per subreddit per run — a handful of requests, far below the permitted rate |
| Throttling | requests are batched one at a time with a 2s gap, 15s timeout |
| Auth | OAuth2, read scope only |

## Data handling

Only public listings are read. The digest holds post IDs, permalinks, titles, short
excerpts, subreddit names and timestamps — enough to not show me the same post twice.

It does not build profiles of users, scrape user histories, share or sell data, or use
any of it for model training. Items age out of the digest within 24 hours.

---

## Setup

1. Import `workflows/personal-reddit-digest.json` into n8n.
2. On the **Fetch New Posts** node, select your Reddit OAuth2 credential (the export
   ships with no credentials attached — see below).
3. Replace `YOUR_USERNAME` in the User-Agent header with your Reddit username. Reddit
   asks that the User-Agent identify the app and its author.
4. Edit the block at the top of **Config — My Interests** with your own subreddits and
   keywords.
5. Replace the **Send digest to me** placeholder with whatever you actually use — email, Telegram, a push service.

## Repository contents

```
workflows/personal-reddit-digest.json   the workflow, no credentials attached
scripts/sanitize_workflow.py            strips credentials before committing
```

An n8n export carries credential IDs, webhook IDs and any pinned run data. Run every
export through the sanitizer before committing it:

```bash
python3 scripts/sanitize_workflow.py raw-export.json workflows/personal-reddit-digest.json
```

## Licence

MIT — see `LICENSE`.
