# X-Agent

AI agent for X/Twitter with full read + write capabilities.

**Reads are free** by default (twitter-cli via cookie auth), **hosted reads and analytics can use Xquik MCP**, and **writes use the official API** (xmcp MCP server).

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

Optional hosted reads, extraction, monitoring, and analytics can route through
the `xquik` remote MCP server configured in `.cursor/mcp.json` when
`XQUIK_API_KEY` is set.

## What You Can Do

### Free (twitter-cli)
- Search tweets
- Read tweets + replies
- Browse home timeline
- View user profiles and posts
- Read X Articles
- View bookmarks and likes

### Optional (Xquik MCP)
- Structured tweet search and account lookups
- Follower and following exports
- Media and article retrieval
- Account or keyword monitoring after explicit approval
- Webhook delivery after explicit approval

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

### Optional: Xquik MCP (hosted reads and analytics)

Create an API key from the Xquik dashboard and export it before starting Cursor:

```bash
export XQUIK_API_KEY="your-dashboard-key"
```

The `.cursor/mcp.json` file already includes the remote `xquik` MCP server.
Use it for hosted read/search, extraction, monitoring, analytics, and webhook
workflows. Keep it read-only by default and approve monitoring or webhook
setup explicitly.

### 3. Use it

Open this project in Cursor. The `.cursor/rules/x-agent.mdc` rule tells the agent how to route:
- Reads → `twitter` CLI commands
- Hosted reads and analytics → `xquik` MCP when `XQUIK_API_KEY` is set
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
