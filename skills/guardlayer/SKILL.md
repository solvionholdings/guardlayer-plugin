---
name: guardlayer
description: Security check for Next.js + Supabase apps with GuardLayer's MCP tools. Use before a deploy or push, after creating or changing tables, RLS policies, storage buckets, API routes, Server Actions or Edge Functions, when the user asks "is my app secure?", or when a Supabase permission error tempts you to loosen access. Scans code and checks the live Supabase project, then confirms fixes landed.
---

# GuardLayer security check

GuardLayer finds the holes AI-built Next.js + Supabase apps usually ship with: tables anyone can read, RLS off, policies that only check for a signed-in user, users who can change their own role or plan, service role keys in frontend code or public env vars (NEXT_PUBLIC_, VITE_), and API routes, Server Actions and Edge Functions that don't check the caller.

The GuardLayer plugin connects its MCP server for you. If its tools (`scan_files`, `get_supabase_snapshot_sql`, `check_supabase_snapshot`) are not available, the server is not connected: check `/mcp`, or add it by hand:

```
claude mcp add --transport http guardlayer https://www.guardlayer.io/api/mcp
```

## 1. Scan the code

Call `scan_files` with the project's source files, paths relative to the project root:
- Include `app/`, `src/`, `lib/`, `components/`, `pages/`, `supabase/migrations/`, `supabase/functions/`, `middleware.ts` / `proxy.ts`, `next.config.*`, and any committed `.env*` files.
- Skip `node_modules`, `.next`, `dist`, `build`, lockfiles and images.
- At most 300 files and 4 MB per call. Split bigger projects over several calls.

## 2. Check the live Supabase project

The repo isn't the database: tables made in the dashboard and policies edited by hand only show up here.

1. Call `get_supabase_snapshot_sql`. It returns a read-only query that reads catalog metadata only, never rows.
2. Run it against the project, in this order of preference:
   - the Supabase MCP server's `execute_sql`, if connected;
   - the Management API read-only endpoint with the user's access token (never print the token);
   - otherwise ask the user to run it in the Supabase SQL editor and paste the single value back.
3. If you can, also read the project's Auth settings (`GET /v1/projects/<ref>/config/auth` with the same token) and pass them as `auth_config`. They decide whether "any signed-in user" means anyone who signs up. Only the fields the tool lists are read.
4. Call `check_supabase_snapshot` with the snapshot (and `auth_config`). Keep the returned `issue_keys`.

## 3. Fix, one issue at a time

- Show the user the findings first, critical ones on top, and fix what they approve.
- Database changes go in a new migration (`supabase/migrations/<timestamp>_guardlayer_fixes.sql`) that you actually apply. A migration file that was never run fixes nothing.
- Start from the SQL fix GuardLayer gives. Change it only if it doesn't match the schema, and say why.
- Never fix an error by disabling RLS, adding `using (true)`, granting to `anon`, or moving the service role / secret key into client code or a public env var.
- After a change, check the places in the app that use it still work for the right users.

## 4. Confirm the fixes landed

Run the snapshot query again and call `check_supabase_snapshot` with `previous_issue_keys` set to the last result's `issue_keys`. It reports what closed, what got less severe, what is new and what is still open.
- "Nothing changed" after a fix usually means the migration wasn't applied, or went to a different project.
- Anything new or worse is often a side effect of the last fix: look at it before moving on.

Report to the user what was fixed, what is still open, and anything you could not check (for example Auth settings when no token was available).

Nothing sent to GuardLayer's tools is stored. Free read-only check in the browser: https://www.guardlayer.io/check-supabase
