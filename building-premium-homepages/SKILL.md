---
name: building-premium-homepages
description: Use when building a new premium marketing/corporate homepage or brand website from scratch — especially multilingual sites that need a distinctive (non-template, non-AI-slop) visual identity, generated 3D/illustration assets, interactive signature sections, an AI chatbot, lead capture, and deployment.
---

# Building Premium Homepages

A reference-driven workflow for shipping a distinctive, multilingual marketing site that looks bespoke (not template/AI-slop) and is actually wired (i18n, generated assets, interactions, lead capture, chatbot, deploy).

**Working reference implementation:** `~/Projects/iromism-homepage-v2` (iroumism.com) — has every pattern below in real code: Tailwind v4 `@theme` tokens (`src/app/globals.css`), next-intl scaffold + per-locale fonts (`src/i18n/`, `[locale]/layout.tsx`), the x-ray scanner (`components/motion/AxScanner.tsx`), Gemini chat SSE route (`app/api/chat/route.ts`), Supabase lead capture (`actions/inquiry.ts`), scroll frame-math (`lib/frame-math.ts`). Read those files when you need concrete syntax.

## Core principles

- **Reference-driven, never from memory.** Collect real reference sites (Pinterest, Awwwards, godly.website, actual competitor sites) by screenshotting with Playwright. Extract color tokens from real sites' live CSS — don't invent palettes. Confirm 2025+ palette/trend choices with web search, never guess hex/pricing.
- **Avoid AI-slop clichés.** No purple/indigo gradients, no abstract 3D blobs / neural-net glows, no fake-text UI. These read as "AI generated this." Differentiate with a warm-neutral or clean-light base + a single signature accent + a real product visual.
- **Local-JSON-first content.** Site must render fully from local JSON defaults; a DB (Supabase) is an optional overlay, never a hard dependency. Gate DB reads behind an env flag.
- **One material spec for every generated asset.** All `$imagegen` images share an identical material/lighting/palette prompt block so the set looks cohesive. (See asset pipeline.)
- **Screenshot-verify every visual change** with Playwright before claiming done. Build a section → screenshot → read it → adjust.

## Workflow

1. **Brainstorm** — positioning, conversion goal, site structure (one-page + subpages), language scope. Decide which sections.
2. **Reference + design system** — gather real-site screenshots; extract palette from real CSS; pick base + single accent; choose per-locale fonts; define a motion vocabulary (stagger headline, count-up stats, tilt cards, parallax, marquee). Write it to a design doc.
3. **Scaffold** — Next.js (App Router, TS) + Tailwind v4 (`@theme` tokens) + next-intl (`[locale]` routing, default locale fallback) + content layer + Vitest.
4. **Build sections** subagent-by-subagent, TDD for pure logic (merge/locale/parse/frame-math), screenshot each.
5. **Generate assets** (see below), wire in, screenshot.
6. **Wire dynamics** — chatbot, lead capture, SEO (hreflang/sitemap/robots), per-locale fonts/metadata.
7. **Deploy** Vercel (`vercel deploy --prod --yes`); verify production with curl + Playwright.

## Stack (proven)

Next.js (latest stable, App Router, TS) + Tailwind v4 (`@theme` tokens) + next-intl (multi-locale, default-locale fallback, `localeDetection` on) + Supabase (optional) + Vercel. Content = `content/defaults/<locale>.json` merged with optional DB rows (4-level fallback: db[locale] > defaults[locale] > db[ko] > defaults[ko]). Chat = plain `fetch` streaming to the LLM provider's SSE endpoint (no SDK needed).

- **next-intl gotcha:** call `setRequestLocale(locale)` at the top of every `[locale]` page/layout (after awaiting params), or routes render dynamic and break ISR/SEO.
- **Per-locale fonts:** inject `--font-sans`/`--font-display` per locale on `<html>` and load only that locale's webfont (CJK fonts are heavy — don't ship all to every visitor). Headlines use a display font, body a separate sans.

## Asset pipeline (codex $imagegen)

Call via codex CLI: `codex exec --skip-git-repo-check '$imagegen <prompt>. 저장하지 말고 생성만.'` — output lands at `~/.codex/generated_images/<uuid>/ig_*.png` (sandbox blocks writing elsewhere). Grab the newest by mtime (`find ~/.codex/generated_images -name '*.png' -mmin -10 -exec stat -f '%m %N' {} \; | sort -n | tail`), read it to eyeball, then `sharp(src).resize().webp()` into `public/objects/`.

- **Run generations sequentially** in one bash call — parallel codex calls collide and silently drop a batch.
- **Shared material spec** — a *per-brand template*: keep the structure (material, lighting, single-accent, plain bg, no-text) identical across the whole set; swap the palette hexes to the brand. Example used on one project:
  `soft matte 3D render, smooth rounded geometry, light powder-blue and cool gray palette (#DCE7F2, #F2F4F6, #FFFFFF) with a single cyan accent (#03CCE1), clean studio lighting from upper left, subtle soft shadows, plain white background, no text, minimal premium corporate style`
- **Realistic app/UI mockups:** prompt "realistic dark-mode SaaS UI screenshot, ... only tiny abstract placeholder label bars (no readable paragraphs)" — avoids garbled fake text while looking like a real product.
- **Reference-edit** to keep composition across variants: `codex exec '... 첨부 이미지 구도 유지 ...' -i <source.png>` (prompt BEFORE `-i`).
- **Video → scroll frames:** Seedance/clip → `ffmpeg fps=N,scale=1600:-2 -c:v libwebp` → canvas scroll-scrub; first frame as LCP poster; reduced-motion fallback.
- **Forbidden:** rising-sun / radiating-ray patterns (욱일기 연상) — add as negative.

## Signature interactions (reusable, high-impact)

- **X-ray scanner hero** — cursor-following circular lens reveals a "before→after" image (legacy office → AI workspace) via `clip-path: circle()`; touch = before/after slider; overlay headline on top with a white scrim, borderless so the white-bg image blends seamlessly.
- **Word↔image sync** — rotating kinetic word (`유튜브 영상→쇼츠→…`) cross-fades a matching product panel image by index. Korean subject particle (이/가) auto-picked by final-consonant only for Hangul.
- **Video-wall gallery** — multi-column vertical marquee mixing video reels + diverse art-style images; pair two 9:16 reels to fill a column width; fade masks top/bottom.
- Count-up stats, stagger-reveal headline (`@keyframes` word-rise), tilt cards, parallax — all behind `prefers-reduced-motion`.

## Lead capture + chatbot

- **Contact form → Supabase**, anon key + RLS **insert-only** (no select policy → leads private). **Reuse an existing active Supabase project and add one prefixed table** (`iro_inquiries`) rather than spinning up a new project — a new project on a Pro org adds ~$10/mo; an extra table is $0. DDL needs the dashboard SQL editor (REST can't do DDL).
- **Email notify** via Resend (`replyTo` = inquirer). Default sender `onboarding@resend.dev` only delivers to the account's own email until a domain is verified.
- **Chatbot** = Gemini 2.5 Flash-Lite (cheapest; consultation bots don't need frontier models), server route streaming Gemini SSE → plain-text deltas, system prompt loaded with company facts, responds in visitor locale, floating widget full-width on mobile (`right-3 left-3 sm:w-[380px]`).

## Common mistakes

- Inventing colors/prices from memory instead of extracting/searching real values.
- Parallel codex `$imagegen` calls (one batch silently lost).
- Forgetting `setRequestLocale(locale)` in next-intl pages/layout → routes render dynamic, breaking ISR/SEO.
- Putting service_role keys in a public marketing site (use anon + RLS for writes).
- Letting a white-bg generated image sit on a gradient section (visible seam) — match section bg to the image bg.
- Claiming a visual is done without a Playwright screenshot.
- Decorative generated images with empty `alt`; meaningful ones (logos, work thumbnails) need localized `alt` for SEO/accessibility.
