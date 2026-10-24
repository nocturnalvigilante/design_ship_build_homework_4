# Security review: `feat/clerk` before merging to `main`

Run with `/security-review` on 2026-10-24, on branch `feat/clerk` against `origin/main` (21 files: Clerk sign-in, Supabase tables and RLS, the Divvy API routes, and the main screen).

## Result

**No high-confidence vulnerabilities found.** Nothing to fix before merging.

## What was checked

| Area | Result |
|---|---|
| Sign-in on the server | Every page, Route Handler and Server Action that shows or changes data calls `await auth.protect()` first (`page.tsx`, both `api/` routes, all four actions). Only `/sign-in` and `/sign-up` are public. |
| Row-level security | RLS on all 3 tables; every policy limits rows to `user_id = auth.jwt()->>'sub'`. Updates check both the old and new row, so a row can't be moved to another user. Signed-out visitors (`anon`) have no grants at all. |
| Departures across users | A departure can only point at a station the same user saved, on insert and on update, even through Supabase's REST API directly. |
| Calling Supabase directly | With your own Clerk token and the publishable key you still only reach your own rows; the policies enforce it, not the app code. |
| Delete by id | `deleteStation` / `deleteDeparture` take an id from the form, but RLS limits deletes to your own rows. |
| Secrets | `src/lib/supabase.ts` is `server-only` and uses only the publishable key plus the user's Clerk token. No secret key in `src/`; no env file tracked; `.mcp.json` holds only the project ref. |
| Injection | All queries use supabase-js filters (`.eq`, `.in`); no raw SQL. `rls_auto_enable()` can't be called through the API. |
| Server-side requests | The Divvy host and paths are fixed in `src/lib/gbfs.ts`; user input only filters results. |
| XSS | No `dangerouslySetInnerHTML`; nicknames and station names render as React text. |
| Route data | `/api/stations` and `/api/status` return only public Divvy data, never Supabase rows. |

## Notes (not vulnerabilities)

- Isolation also depends on two dashboard settings outside the code: Supabase's third-party auth trusting only this Clerk instance, and Clerk's Supabase integration adding the `authenticated` role. Both were set up and checked on 2026-10-24.
- When Divvy is down, `saveStation` skips checking that the station id exists. The worst case is a user saving a bad id into their own row.
- Separate from this review: `npm audit` still reports `postcss` bundled inside Next. It runs only at build time on our own CSS; the fix needs Next 16.

## Found earlier, before this review

- Critical Next.js image-optimizer bug (GHSA-2464-fhr4-w9xw) in 15.5.27: patched to 15.5.28.
- Supabase security advisor: `rls_auto_enable()` was callable by anyone through the API: revoked in a migration.
