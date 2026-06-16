# X-Agent

AI agent for X/Twitter with full read + write capabilities.

**Reads are free** (twitter-cli via cookie auth), **writes use the official API** (xmcp MCP server).

## Architecture

```
┌─────────────────────────────────────────────────┐
│                  Cursor Agent                    │
│              (rules in .cursor/)                 │
├────────────────────┬────────────────────────────-┤
│                    │                              │
│  READ / SEARCH     │  WRITE (tweet, RT, DM, etc) │
│  ▼                 │  ▼                           │
│  twitter-cli       │  xmcp MCP server             │
│  (cookie auth)     │  (OAuth1, official X API)    │
│  FREE              │  API tier costs apply        │
└────────────────────┴──────────────────────────────┘
```

## What You Can Do

### Free (twitter-cli)
- Search tweets
- Read tweets + replies
- Browse home timeline
- View user profiles and posts
- Read X Articles
- View bookmarks and likes

### Official API (xmcp)
- Post tweets
- Retweet / unretweet
- Like / unlike
- Send DMs, read DM history
- Follow / unfollow / mute / unmute
- Delete tweets, hide replies
- Manage bookmarks
- Search users, get trends
- Anything else in the [X API](https://developer.x.com/en/docs)

## Setup

### 1. twitter-cli (reads — already done)

```bash
pipx install twitter-cli
twitter status  # should show authenticated
```

### 2. xmcp (writes)

Get X Developer App credentials from https://developer.x.com/en/portal/dashboard:
- Create a new app (or use existing)
- Enable OAuth 1.0a with read+write+DM permissions
- Set callback URL to `http://127.0.0.1:8976/oauth/callback`
- Copy Consumer Key, Consumer Secret, and Bearer Token

Fill in `xmcp/.env`:

```bash
X_OAUTH_CONSUMER_KEY=your_consumer_key
X_OAUTH_CONSUMER_SECRET=your_consumer_secret
X_BEARER_TOKEN=your_bearer_token
```

Start the server:

```bash
./start-xmcp.sh
```

On first run, a browser window opens for OAuth1 consent. Approve it.
The MCP server runs at `http://127.0.0.1:8000/mcp`.

### 3. Use it

Open this project in Cursor. The `.cursor/rules/x-agent.mdc` rule tells the agent how to route:
- Reads → `twitter` CLI commands
- Writes → `x-api` MCP server tools

Just ask naturally: "search for tweets about AI agents", "post a tweet saying hello", "DM @someone", etc.

## File Structure

```
X-Agent/
├── .cursor/
│   ├── mcp.json          # MCP server config (xmcp endpoint)
│   └── rules/
│       └── x-agent.mdc   # Agent routing rules
├── xmcp/                  # Official X API MCP server (cloned)
│   ├── server.py
│   ├── .env               # Your API credentials (git-ignored)
│   └── .venv/             # Python virtualenv
├── start-xmcp.sh          # One-command server startup
├── .gitignore
└── README.md
```

## Tool Allowlist

The `.env` pre-configures a curated allowlist of ~35 tools. To add more,
see the full list of 100+ tools in `xmcp/README.md` and append to
`X_API_TOOL_ALLOWLIST` in `xmcp/.env`.

## Pairing: GetXAPI MCP server (optional alternative backend)

Users who already adopt this project sometimes ask about routing read-heavy operations (tweet search, profile lookup, follower lists) to a different X/Twitter backend during testing or for specific workflows. The [GetXAPI MCP server](https://github.com/getxapi/getxapi-mcp) (MIT licensed, open source) implements the same read tools and can be paired with this project without changing existing behavior.

Two integration patterns:

1. **Side-by-side in your AI client.** Keep this project for its primary workflow and add the GetXAPI MCP server when you need a different backend for read operations. Each tool name maps to whichever backend the user has configured.

2. **Code-level toggle.** For a worked reference of an optional alternative backend behind a single env variable, see the [pattern merged into GenAIwithMS/twitter-mcp](https://github.com/GenAIwithMS/twitter-mcp/pull/3).

Tool compatibility (subset that pairs cleanly with this project's read path):

- `search_tweets`
- `get_user_profile`
- `get_user_followers`
- `get_tweet_by_id`

Repository: https://github.com/getxapi/getxapi-mcp

This pairing is fully optional. No behavior change for existing users of this project.

