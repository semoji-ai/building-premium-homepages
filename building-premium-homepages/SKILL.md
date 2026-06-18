---
name: building-premium-homepages
description: Use when building a new premium marketing/corporate homepage or brand website from scratch — especially when you must discover the client's needs, gather design references, choose a visual identity, and ship a distinctive (non-template, non-AI-slop) multilingual site with generated assets, interactive sections, chatbot, lead capture, and deployment.
---

# Building Premium Homepages

A **collaborative process**: interview the client to understand their needs, gather real references (sites, color themes, feature ideas), present options and let the client choose, then build the site from the chosen direction. The output looks bespoke because the *decisions* come from the client + real references — never from your defaults.

**This skill is the process, not a preset.** Any specific palette/section/font mentioned below is an *example from one project*, never the answer. Re-run discovery for every new site.

**Working reference implementation:** `~/Projects/iromism-homepage-v2` (iroumism.com) — real code for every technique here (Tailwind v4 `@theme` tokens, next-intl + per-locale fonts, x-ray scanner `components/motion/AxScanner.tsx`, Gemini chat SSE `app/api/chat/route.ts`, Supabase lead capture `actions/inquiry.ts`, scroll frame-math `lib/frame-math.ts`). Open these when you need concrete syntax — copy the *structure*, not the brand choices.

## Non-negotiable principles

- **AI-slop is banned.** No purple/indigo gradients, no generic abstract 3D blobs / neural-net glows, no fake-garbled-text UI mockups, no Inter-on-purple template look. These scream "an AI made this." Differentiate via real references + a chosen identity + real product visuals. Treat any drift toward these as a defect.
- **Reference-driven, never from memory.** Every visual decision traces to a real reference or an extracted value. Never invent palettes or quote prices/trends from memory — screenshot real sites, extract real CSS tokens, web-search current trends/pricing.
- **The client selects; you collect and present.** Your job is to gather strong options and make them choosable, not to decide for them. Use the brainstorming visual companion or `AskUserQuestion` to present and capture selections.
- **Local-JSON-first content.** Site renders fully from local JSON; a DB (Supabase) is an optional overlay behind an env flag, never required.
- **Screenshot-verify every visual** with Playwright before claiming done.

## The Process

### 1. Discovery — Socratic interview (one question at a time)
Start with `superpowers:brainstorming`. Ask **one question per turn**, prefer multiple-choice, dig into:
- What does the company do? What's the single most important visitor action (conversion goal)?
- Who is the audience? B2B decision-makers, consumers, global?
- Brand personality / tone in 3 words? What feeling should the first screen give?
- Language scope (which locales; which is default/fallback)?
- **What makes you different** from competitors — the thing the site must convey?
- Existing assets: logo, brand colors, content, channels, product screenshots?
- Sites you admire (yours or others) — and *why*?
- Must-have sections / features / functionality?
Write answers into a design doc as you go.

### 2. Collect references (and let the client pick)
Use Playwright to screenshot real sites: Awwwards, godly.website, competitors, and Pinterest searches. **Present a gallery** (visual companion or screenshots) of 6–10 real examples; ask the client which they're drawn to and why. Explicitly include an "avoid" board (e.g., the AI-slop cliché look) so the client can reject it. Capture selections.

### 3. Collect color themes (and let the client pick)
Pull palettes from THREE real sources: (a) extract live CSS custom-properties/hex from sites the client liked (`browser_evaluate`), (b) Pinterest mood searches for the chosen vibe, (c) web-search the current year's color/trend authorities (Pantone, WGSN) — never guess hex. Present 3–4 concrete palette options (with base/surface/text/accent/dark + source attribution + pros/cons); client selects one. Derive Tailwind `@theme` tokens from it.

### 4. Collect functional / interaction patterns (and let the client pick)
From the references, list candidate sections and signature interactions (hero type, scroll behavior, product showcase, social proof, motion). Present them; client selects what fits. Decide the section backbone and 1–2 *signature* interactions that carry the brand.

### 5. Synthesize → design doc → get approval
Combine the selected references + palette + fonts + sections + interactions into a written design doc. Get explicit approval before building. (This is the brainstorming skill's terminal state → `superpowers:writing-plans`.)

### 6. Build from the selections
Scaffold → build sections subagent-by-subagent (TDD for pure logic) → generate assets → wire interactions/i18n/SEO/chatbot/lead-capture → deploy. Screenshot each step. Tech that worked: Next.js (App Router, TS) + Tailwind v4 + next-intl (multi-locale, default fallback, `localeDetection` on; call `setRequestLocale(locale)` in every `[locale]` page/layout or ISR/SEO breaks) + Supabase (optional) + Vercel (`vercel deploy --prod --yes`). Per-locale fonts injected on `<html>`, loading only that locale's webfont (CJK is heavy).

## Toolkit — HOW to implement, once chosen

This is a capability library for the *build* phase, not a menu of defaults. The actual sections/interactions/aesthetic come from the client's references (Steps 2–4) — never pick from this list because it's here. A law firm and an art studio share these mechanics but will choose entirely different sections, palettes, and visuals. Adapt every example (palette, motion, copy) to the chosen identity.

- **Asset generation (codex `$imagegen`):** `codex exec --skip-git-repo-check '$imagegen <prompt>. 저장하지 말고 생성만.'` → output at `~/.codex/generated_images/<uuid>/ig_*.png` (grab newest by mtime) → `sharp().resize().webp()` into `public/objects/`. **Run sequentially** (parallel codex calls collide, silently drop a batch). **One shared material-spec block in every prompt** so the set is cohesive — keep structure identical, swap palette to the chosen brand. For UI mockups prompt "only tiny abstract placeholder labels (no readable paragraphs)" to avoid garbled text. Reference-edit with `-i <src.png>` (prompt before `-i`) to keep composition. Forbidden: rising-sun/radiating-ray patterns.
- **Video → scroll frames:** clip → `ffmpeg fps=N,scale=1600:-2 -c:v libwebp` → canvas scroll-scrub; first frame = LCP poster; reduced-motion fallback.
- **Signature interactions** (pick what fits the brand): cursor-lens x-ray reveal (before→after via `clip-path: circle()`, slider on touch); kinetic word↔image sync; multi-column video-wall marquee; count-up stats; stagger-reveal headline; tilt cards; parallax — all behind `prefers-reduced-motion`.
- **Lead capture:** form → Supabase, anon key + RLS **insert-only** (no select policy = private). **Reuse an existing active Supabase project + add one prefixed table** rather than a new project (a new project on a Pro org adds ~$10/mo; a table is $0). DDL via dashboard SQL editor. Email notify via Resend (`replyTo` = inquirer; default sender only mails the account's own address until a domain is verified).
- **AI chatbot:** cheapest small model is plenty for a consultation bot (e.g., Gemini Flash-Lite); server route streams the provider's SSE → plain-text deltas; system prompt loaded with company facts; respond in visitor locale; widget full-width on mobile (`right-3 left-3 sm:w-[380px]`).
- **SEO/launch:** hreflang/sitemap/robots; OG image (real branded card, not a placeholder); set SITE_URL to the domain that actually resolves (check apex vs www — a broken www OG URL shows a gray preview); verify search-console/Naver via DNS TXT or meta tag.
- **Blog + self-serve admin + AI translation:** when the client needs to manage content themselves (posts, inquiries, editable copy, company/SNS info, visitor analytics) or wants a multilingual blog — see `blog-and-admin.md`. Covers a DB-first-with-markdown-fallback blog, a Supabase-Auth `/admin` gated by an allowlist, per-instance portable schema (migrations + setup doc), agent-authored-vs-in-app two-pass translation (with a source-language-regression guard), and the **never-auto-publish** content-planning discipline (plan → research real blogs → draft → reader/customer/SEO evaluation → approve → publish).

## Common mistakes

- Skipping discovery and imposing your own palette/sections (defeats "bespoke").
- Inventing colors/prices from memory instead of extracting/searching real values.
- Letting AI-slop clichés (purple gradient, abstract 3D, fake-text UI) creep in.
- Parallel codex `$imagegen` calls (one batch silently lost).
- Forgetting `setRequestLocale(locale)` → dynamic render, broken ISR/SEO.
- Service_role keys in a public site (use anon + RLS for writes).
- White-bg generated image on a gradient section (visible seam) — match bg.
- Claiming a visual is done without a Playwright screenshot.
- Decorative images with non-empty alt, or meaningful images (logos, work thumbs) with empty alt.
