# Assignment 4 grading checklist: Full Stack App

Every item is taken from the brief. An item is ticked only once it has been **verified**, not just written. Due **Tuesday 27 October 2026, 5:30 PM**.

## A. Assessment ("A complete assignment has")

| # | Requirement | Evidence | Status |
|---|---|---|---|
| A1 | A live Vercel URL where a classmate can make an account and use the tool | 2026-10-26: a classmate signed up on their phone and saved "Quad" (University Ave & 57th St); Supabase shows it in a second account (`docs/evidence/classmate.jpg`, `docs/evidence/classmate-check.txt`) | ☑ |
| A2 | Sign-in with Clerk, and nothing visible when signed out | signed-out visit redirects to /sign-in (local + live); sign-out on phone lands on sign-in with no data (`docs/evidence/phone-signed-out.jpg`) | ☑ |
| A3 | Data in Supabase, with row-level security on every table | 2026-10-26: all 3 tables RLS on, 4 owner-only policies each, `anon` has no access, security advisor 0 lints (`docs/evidence/rls-check.txt`) | ☑ |
| A4 | Data from an external API, with loading, empty and unavailable states | `src/lib/gbfs.ts` + designed states; "nothing found" and "API down" checked by `e2e/states.spec.ts` (`docs/evidence/test-states.txt`) | ☑ |
| A5 | What you save is still there after signing out and back in | "lab" saved on laptop, seen on phone at dockcheck-phi.vercel.app (`docs/evidence/phone-persistence.jpg`) | ☑ |
| A6 | Keys kept out of the code and the repo, with a deny rule for `.env` files | deny rule committed first (`664ad2f`); `.env*` git-ignored, no env file ever tracked; user wrote `.env.local`. ⚠ Clerk secret was pasted in chat (not repo): rotate it | ☑ |
| A7 | One Playwright test, watched failing before it passed | `e2e/isolation.spec.ts`; failed with RLS broken (`docs/evidence/test-failing-rls-broken.txt`: "user B can see user A's station"), passed after the fix (`docs/evidence/test-passing-rls-fixed.txt`) | ☑ |
| A8 | `/security-review` run on a branch | 2026-10-24 on `feat/clerk` before merging: 0 findings, report in `docs/security-review.md` | ☑ |
| A9 | A GitHub repo with commits that show progress (not one-shot) | 21+ small commits, 3 feature branches, each merged after `/security-review` | ☑ |
| A10 | Ready to explain the work in class (section F) | | ☐ |

## B. Steps, each with its "Ask your agent" prompt

| # | Step | Ask-your-agent done? | Status |
|---|---|---|---|
| B1 | **Tool and API:** extend A3 or start something new; pick one external API (keyless list in the brief, or your own) and make it distinct from classmates' apps | — | ☑ new tool DockCheck, Divvy GBFS; chosen over a gym-crowd idea once UChicago turned out to publish no occupancy feed |
| B1a | The agent calls the API and shows one **real response**: which fields come back, and what happens when a search finds nothing | ☑ "Explain this response: what each field means, and which ones my tool needs." | ☑ 2026-10-24: live call, 2,076 stations; Ellis Ave & 58th St showed 0 bikes, 18 docks; a search for "zzz" and an unknown id both come back empty, not as errors |
| B2 | **Data plan with the agent:** what gets saved and whose it is; 2–3 tables; **every table holding someone's data has a `user_id` column**; data model written in `AGENTS.md` and read before building | ☑ "Draw a diagram of my tables…" (docs/learning.md §2 (ER diagram)) | ☑ AGENTS.md data model committed 2026-10-21, before any code |
| B3 | **Keys:** (deny rule ☑ committed before any key exists) `.claude/settings.json` deny rule `["Read(./.env)", "Read(./.env.*)"]` added **before** the first key; **you** create `.env.local` (the agent can't) | ☑ "Explain why this deny rule protects my keys… publishable vs secret keys…" (docs/learning.md §3) | ☑ deny rule `664ad2f` before the first key; `.env.local` created by the user |
| B4 | **Clerk:** run `npx -y clerk@latest init`; a signed-out visitor can't reach anyone's data, **checked by trying it signed out** | ☑ "Show me how the Clerk CLI works: what clerk init added, file by file." (middleware, ClerkProvider in layout, /sign-in, /sign-up) | ☑ 2026-10-24: signed-out browser got `GET / 307` → `/sign-in`; after signing in, `GET / 200` (dev server log) |
| B5 | **Supabase:** create the project; connect its **MCP server scoped to that one project** (write access recommended); Clerk connected via Supabase third-party auth (Authentication → Sign In / Providers → Clerk); tables created **as migrations**, with **RLS on every table** | ☑ "Explain each RLS policy in plain words, then show me a query one would block." (docs/learning.md §5 (each policy + blocked queries)) | ☑ 2026-10-24: project `dockcheck` (us-east-1); MCP pinned by `project_ref` in `.mcp.json`; Clerk third-party auth enabled; 2 migrations in `supabase/migrations/`; all 3 tables RLS on with 4 policies each, `anon` has no access; security advisor 0 issues |
| B6 | **Where the API is called:** browser or server, with the reason (keys, rate limits, caching, or none); designed states for **loading**, **nothing found** and **API down** | ☑ "Draw a diagram of one request, from the click to the screen…" (docs/learning.md §6 (sequence diagram)) | ☑ server-side, cached 60 s (size + caching); loading / nothing found / API down / stale all designed, two checked by `e2e/states.spec.ts` |
| B7 | **Playwright test:** two users sign in, each saves something, and neither can see the other's; **break RLS on purpose → watch it fail → fix → watch it pass** (keep the failing output as evidence) | ☑ "Explain why the test failed…": *I changed the read policy to `using (true)`, so Supabase handed every signed-in user everyone's saved stations, and user B's screen showed user A's station: 1 where the test expected 0.* | ☑ 2026-10-24: broken in SQL Editor (`e2e/break-rls.sql`) → failed → fixed (`e2e/fix-rls.sql`) → passed; outputs in `docs/evidence/` |
| B8 | **Review and deploy:** work on a **branch**; run `/security-review` before merging; **read the agent's changes yourself**; deploy on Vercel with env vars set; **a classmate makes an account and uses it** | ☐ "Quiz me on how my app works, one question at a time…" | ☐ |

## C. Requirements

| # | Requirement | Status |
|---|---|---|
| C1 | **Accounts:** sign up, sign in and sign out with Clerk; a signed-out visitor sees no one's data | ☑ 2026-10-24 sign-up (Google), sign-in on laptop and phone, sign-out on phone |
| C2 | **Data:** in Supabase, RLS on every table, each person sees only their own rows | ☑ RLS on all 3 tables; two-user Playwright test (A7) |
| C3 | **API:** data from one external API, with designed loading, nothing-found and API-down states | ☑ see A4 |
| C4 | **Persistence:** saved data is still there after signing out and back in, **on another device** | ☑ 2026-10-24 laptop → phone, `docs/evidence/phone-persistence.jpg` |
| C5 | **Keys:** in environment variables, never in the code or repo, with a deny rule for `.env` files | ☑ see A6 |
| C6 | **Test:** one Playwright test that two users can't see each other's data, watched failing first | ☑ 2026-10-24, see A7 |
| C7 | **Review:** `/security-review` on a branch, **and its findings handled** | ☑ two reviews (`feat/clerk`, `feat/playwright-test`), 0 findings each; earlier advisor + npm audit findings fixed (`docs/security-review.md`) |
| C8 | **Deployed:** live on Vercel, where a classmate can make an account and use it | ☑ classmate signed up and saved a station, 2026-10-26 (`docs/evidence/classmate-check.txt`) |
| C9 | **UI/UX:** the design process, principles and flows from earlier weeks still applied | ☑ design process and principles written up in README "Design process" |

## D. Tech stack

| # | Requirement | Status |
|---|---|---|
| D1 | Next.js and Tailwind CSS | ☑ |
| D2 | Clerk for sign-in, **development keys only** (production keys need a domain you own) | ☑ `pk_test_` / `sk_test_` development instance |
| D3 | Supabase for the database | ☑ |
| D4 | One external API | ☑ Divvy GBFS |
| D5 | Code in your own GitHub repo | ☑ github.com/nocturnalvigilante/design_ship_build_homework_4 |
| D6 | Deployed on Vercel's free Hobby plan | ☑ https://dockcheck-phi.vercel.app (2026-10-24); signed-out `/`, `/api/stations`, `/api/status` all redirect with no data |
| D7 | `AGENTS.md` is the context file; any `CLAUDE.md` merged into it and deleted; `/context` shows `AGENTS.md` loaded | ☑ no CLAUDE.md exists; `/context` lists `~/projects/dbs-a4/AGENTS.md` under memory files (`docs/evidence/context-agents-md.png`) |

## E. Free tiers and usage limits

| # | Item | Status |
|---|---|---|
| E1 | Stay on the free tier for Vercel, Supabase and Clerk | ☑ Vercel Hobby, Supabase Free, Clerk Hobby (dev instance) |
| E2 | Set usage limits and spending alerts wherever offered | ☐ check each Usage page; see docs/learning.md §8 |
| E3 | No payment method unless required; if one is required, set a spending cap first | ☑ no payment method added anywhere |
| E4 | Supabase pauses free projects after a week of no use, so open the app regularly until grades are back | ☐ ongoing: open the app at least weekly until grades are back |
| E5 | Ask your agent: "Walk me through the free tier limits for each service… and how to set usage limits and spending alerts." | ☑ docs/learning.md §8 |

## F. Be ready to explain

| # | Question | Status |
|---|---|---|
| F1 | **Tracing requests:** someone saves an item; what happens in the browser, on the server, in Clerk, in Supabase and at the API | ☐ answer drafted in docs/learning.md §6; practise with "Quiz me" |
| F2 | **Access control:** where the app knows who someone is, and where it decides what they may see | ☐ answer drafted in docs/learning.md §6 |
| F3 | **API calls:** where the app calls the API, and why there | ☐ answer drafted in docs/learning.md §6 |
| F4 | **Testing:** what the failing test looked like, and what you changed | ☐ answer drafted in docs/learning.md §7 |
| F5 | **Agentic workflow:** what you stopped the agent from doing, and what you caught reading its changes | ☐ answer drafted in docs/learning.md §9 |

## G. Submission (Google Classroom)

| # | Item | Status |
|---|---|---|
| G1 | Vercel URL: https://dockcheck-phi.vercel.app | ☐ |
| G2 | GitHub URL (README + commit history) | ☑ README rewritten; 21+ commits |

## Easy-to-miss items (each one costs marks if skipped)

- The deny rule must exist **before** the first key is pasted, and **you** write `.env.local`, not the agent.
- The test has to **fail first** with RLS broken on purpose. Save that failing run as evidence.
- `/security-review` must run **on a branch, before merging**, and its findings must be handled.
- A real **classmate** must make an account and use the deployed app.
- Persistence must be checked **on another device**, not just by signing out and back in.
- It must be **Playwright** this time. In A2 you used DevTools-protocol scripts instead, which won't count here.
