# GuardLayer for Claude Code

Security check for Next.js + Supabase apps, run by your AI agent. GuardLayer scans the code, checks the live Supabase project read-only, and after a fix confirms what actually closed.

It catches the holes AI-built apps usually ship with: tables anyone can read, RLS off, policies that only check for a signed-in user (public in practice when anyone can sign up), users who can change their own role or plan, service role keys in frontend code or `NEXT_PUBLIC_` / `VITE_` variables, and API routes, Server Actions and Edge Functions that don't check the caller.

## Install (Claude Code)

```
claude plugin marketplace add solvionholdings/guardlayer-plugin
claude plugin install guardlayer@guardlayer
```

The plugin adds the GuardLayer MCP server and a skill that tells Claude when to use it: before a deploy, after changing tables or policies, or when it's about to loosen access to get past a permission error. Then ask: "Check this app for security issues with GuardLayer."

## Other MCP clients

The server is a remote MCP server over Streamable HTTP, no install needed:

```
https://www.guardlayer.io/api/mcp
```

Cursor (`.cursor/mcp.json`):

```json
{ "mcpServers": { "guardlayer": { "url": "https://www.guardlayer.io/api/mcp" } } }
```

## Tools

| Tool | What it does |
| --- | --- |
| `scan_files` | Static scan of the files your agent sends. Each finding comes with the fix. |
| `get_supabase_snapshot_sql` | A read-only query for the project's catalog: table and column names, RLS flags, policies, grants, triggers, functions, bucket settings. Never rows. |
| `check_supabase_snapshot` | Checks the snapshot together with your Auth settings. With `previous_issue_keys` it reports what closed, what's new and what's still open. |

## Privacy

Code and snapshots sent to the server are checked in memory and discarded. Your Supabase access token stays on your machine; Auth settings are reduced to on/off flags and numbers before they are read. See the [privacy policy](https://www.guardlayer.io/privacy).

More: [guardlayer.io/mcp](https://www.guardlayer.io/mcp) · Browser version: [guardlayer.io/check-supabase](https://www.guardlayer.io/check-supabase)
