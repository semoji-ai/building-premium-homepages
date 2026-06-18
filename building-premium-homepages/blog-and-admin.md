# Blog + Self-Serve Admin (reference)

How to add a DB-backed multilingual **blog**, a protected **/admin** (content · inquiries · site copy · company/SNS · analytics), and an **AI translation** workflow to a homepage built with the parent skill. Reusable across instances — each deployment points at its **own** Supabase project.

This is a capability reference, not a preset. Adapt table/route names to the project. The reference implementation is `~/Projects/iromism-homepage-v2`.

## When to use
Client wants to publish/manage content themselves (blog/insights), see inquiries and visitor stats, and edit copy without a developer or redeploy. Also: government-support-program credibility (active blog + SNS + Organization data is trackable evidence).

## Architecture (portable)
- **All Supabase access via env** (`NEXT_PUBLIC_SUPABASE_URL` / `ANON_KEY`) — never hardcode a project. Ship schema as re-runnable **migration SQL** + a `docs/admin-setup.md` so any user runs it on their own project. New project on a Pro org adds ~$10/mo; a free project or added tables are $0.
- **Blog = DB-first with file fallback.** `getAllPosts(locale)` / `getPost(slug, locale)` query a `posts` table (published only, public read via RLS); on DB error / not-configured / empty, fall back to local markdown files. This keeps the site live during migration and before the DB exists. Migrate seed posts via dollar-quoted (`$IRO$…$IRO$`) INSERT SQL pasted in the dashboard SQL editor (REST can't do DDL; anon can't bypass RLS to insert — SQL editor runs as postgres).
- **Instant updates:** admin writes call `revalidatePath('/[locale]/blog', …)` so public pages refresh without redeploy. Cover images → Supabase Storage public bucket (URL stored), not `/public` (which needs a rebuild).

## Auth (Supabase Auth + allowlist)
- `@supabase/ssr` for cookie sessions (`createServerClient` with `cookies()` adapter; `setAll` wrapped in try/catch — no-op in Server Components).
- **Allowlist table** (`admins(user_id)`) gated by a `security definer` `is_admin()` SQL function. Only listed accounts reach `/admin`, even though the project's Auth may hold other (app) users. Writes run under the **authenticated user's session + RLS** — never ship a service_role key to a public site.
- Create admin accounts via dashboard **Add user + "Auto Confirm"** (sets password directly, no email). Invite emails break if Auth **Site URL** is still `localhost:3000` — set it to the production domain (Authentication → URL Configuration).
- Route shape: `/admin/login` (unguarded) + `/admin/(protected)/…` whose group layout calls `requireAdmin()` (redirect if not admin).

## Admin sections (what's worth building)
| Section | Backing | Notes |
|---|---|---|
| Blog CRUD | `posts` table | list / editor (markdown textarea + `marked` live preview) / draft↔publish / cover upload / delete |
| Inquiries | existing leads table + `status` col | RLS: anon insert, admin select/update |
| Site copy | `site_content(key,locale,value)` | overlays local JSON when `ENABLE_DB_CONTENT=1`; whitelist editable keys incl. `meta.title/description` (move hardcoded `generateMetadata` defaults into the content tree so they're editable) |
| Company / SNS | `site_content` `org.*` (store under `ko`, applies to all locales via ko-base merge) | feeds **Organization JSON-LD** (`sameAs`) + footer business info & SNS links |
| Analytics | `pageviews` table + `/api/track` beacon (client `usePathname` effect) | in-admin dashboard (today/7d/30d, top pages, referrers, inquiry conversion); pair with `@vercel/analytics`. Filter bots by UA in the route |

## Multilingual blog
- One row per `(slug, locale)`. **Per-post fallback** in listing: group by slug, prefer requested locale else `ko` (so partially-translated sets still show every post). `getPost` picks locale row else ko.
- **Two translation paths:**
  1. **Agent-authored (preferred when building with an LLM agent):** the agent translates all locales + self-reviews against the source, you upload as **drafts** → human review → publish. Highest quality; the live app calls no API. Dispatch one review agent per language in parallel.
  2. **In-app auto-translate (fallback for solo admin authoring):** a button calls an LLM (Gemini Flash-Lite or Claude) server-side. Use **two passes — translate, then an AI "editor" pass that proofreads against the source** — and save as draft. Provide a separate "AI proofread" action for existing/edited drafts.
- **Guard against language regression:** the review/proofread pass can "correct" a translation back toward the source language. After review, if the output is predominantly source-script (e.g. Hangul ratio > 0.15 for a non-ko target), discard the review and keep the prior translation. Also state in the prompt: "output MUST be entirely in <lang>; the source is reference only."

## Content workflow discipline (do NOT skip)
- **Never auto-publish content you invented.** Plan first. The pipeline: plan topics → research real comparable blogs (format/structure/depth, by reading them, not from memory) → draft (in a non-published `*-drafts/` dir or `status:draft`) → **multi-lens evaluation** (reader / prospective-customer / SEO, e.g. parallel critique agents) → refine → human approval → publish.
- Effective B2B post shape (from research): question-hook intro, numbered/question H2s with target keywords, a real table/diagram (no text-only walls), concrete examples + numbers, FAQ block, honest external citations, inline CTA at the end (deliverable + time + "not a sales call"). Korean B2B depth ~2,000–3,000 words; thought-leadership ~1,200–1,500.
- Keep a single source of truth: once posts live in the DB, delete the markdown copies.

## Common pitfalls (all hit in practice)
- **`/admin` renders unstyled:** a `[locale]/layout` that owns `<html>` + `globals.css` doesn't cover non-locale routes. Add a separate `app/admin/layout.tsx` with its own `<html><body>` + `globals.css` (+ font, + `robots: noindex`). Root `app/layout.tsx` returning `children` only is the trap.
- **Locale middleware hijacks /admin:** exclude it from the next-intl matcher (`/((?!api|admin|_next|...).*)`), or `/admin` redirects to `/ko/admin`.
- **Tailwind `prose` does nothing:** v4 needs `@plugin "@tailwindcss/typography";` in CSS — without it, markdown tables/headings render unstyled.
- **Long server actions feel dead:** wrap the submit button in a client component using `useFormStatus()` to show a spinner/"working…". Raise `export const maxDuration` on the page when an action calls several LLM passes.
- **Image-gen "transparent background" is fake:** the model paints a checkerboard into pixels (no alpha). For a logo/favicon, luminance-key the solid bg to transparent with sharp, or use a real background-removal tool. Keep `apple-icon` on a solid bg (iOS fills transparent with black). Generate favicon.ico via `png-to-ico`.
- Treat `getAllPosts`/`getPost` as async once DB-backed; update sitemap and every caller.
