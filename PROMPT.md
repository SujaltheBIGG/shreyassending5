# Meditations Website Reel — Build Prompt

This file is a complete, self-contained spec for rebuilding this website **exactly**.
Hand it to a person or an AI agent: everything needed is here, including the full source
of every file in the project (Appendix A and B).

---

## 1. What this is

A **single-page static marketing site** — a dark SaaS landing page. It began life as the
"Kloudexa" Webflow HTML template export and was then customized (see §7 for every change).

Key property: **there is no build step.** No React, no Tailwind, no npm, no bundler, no
framework. It is one HTML file that loads its CSS and JS from CDNs. You edit the HTML and
refresh the browser.

**It requires an internet connection to render.** The stylesheet and JavaScript that make the
page look like anything at all are hosted on Webflow's CDN, not in this repo. If the page ever
loads as unstyled black text on white, that is the CDN failing — not a bug in the markup.

---

## 2. File tree

The entire project is two files:

```
.
├── .gitignore     # ignores .DS_Store  (Appendix B)
└── index.html     # the entire website  (Appendix A — 69,788 chars)
```

The filename `index.html` is **load-bearing**. Web servers serve a file named `index.html`
automatically at `/`. Name it anything else and visitors get a directory listing instead of
the site. (This is a real bug that occurred during development.)

---

## 3. Running it

Any static file server works. From the project root:

```bash
python3 -m http.server 8080
# then open http://localhost:8080
```

Do **not** just double-click the file to open it over `file://` — some paths behave
differently there. Serve it over HTTP.

**Deploying:** GitHub Pages (deploy from branch, root), Vercel, or Netlify all work with no
build command and an output directory of `./`.

---

## 4. External dependencies

All loaded from CDNs at runtime. Several carry Subresource Integrity hashes — **copy the
`integrity` and `crossorigin` attributes verbatim from Appendix A**, because a mismatched hash
means the browser refuses to load the file and the page silently breaks.

| Dependency | Purpose |
|---|---|
| Webflow shared CSS | Every visual style on the page (`kloudexa.webflow.shared.*.min.css`) |
| jQuery 3.5.1 | Required by the Webflow runtime |
| Webflow runtime chunks (×3) | Scroll-triggered animations (IX2), dropdowns, lightbox |
| Google Fonts (Inter, Space Grotesk) | Typography, via the WebFont loader |
| Lenis 1.1.18 | Smooth scrolling (added; see §7.7) |

Exact URLs in use:

```
https://ajax.googleapis.com/ajax/libs/webfont/1.6.26/webfont.js
https://cdn.jsdelivr.net/npm/lenis@1.1.18/dist/lenis.min.js
https://cdn.prod.website-files.com/68036da11dd5436718772525/css/kloudexa.webflow.shared.87c665f93.min.css
https://cdn.prod.website-files.com/68036da11dd5436718772525/js/webflow.0d985c84.68f15673198f19d7.js
https://cdn.prod.website-files.com/68036da11dd5436718772525/js/webflow.schunk.36b8fb49256177c8.js
https://cdn.prod.website-files.com/68036da11dd5436718772525/js/webflow.schunk.6b42e12f681f0615.js
```

---

## 5. Design tokens

Defined by the Webflow stylesheet's `:root` — reuse these rather than inventing new colors:

| Token | Value |
|---|---|
| `--black` | `black` — the page background |
| `--light-slate-gray` | `#a3a8b8` — default body text |
| `--royal-blue` | `#05f` — accent / CTA |
| `--white` | `white` |
| `--dark-slate-blue` | `#263960` |
| `--jet-gray` | `#24262c` |

The site is **dark**: `body` is black with `#a3a8b8` text in Inter. Any custom UI you add must
assume a dark background (light borders at ~12% white, etc.).

**Breakpoints.** Webflow's own CSS uses max-width `991px` / `767px` / `479px`. The custom
navbar added here uses a min-width `768px` desktop breakpoint. Both conventions coexist.

---

## 6. Page structure

`<body>` is a flat list of sections, in this order:

```
1. <header class="fc-header (custom navbar — §7.8)">
2. <section class="rt-about-v1">
3. <section class="rt-feature-v1 rt-position-relative">
4. <section class="rt-work">
5. <section class="rt-result-v1">
6. <section class="rt-service-v1 rt-position-relative">
7. <section class="rt-faq-v1">
8. <section class="rt-footer rt-position-relative">
```

…followed by the script tags (jQuery → Webflow runtime → Lenis → custom navbar JS).

Everything except the navbar is stock Webflow template markup. **It cannot be regenerated from
a description** — it is ~55 KB of generated markup with hashed class names and `data-w-id`
animation hooks. To reproduce this site exactly, take `index.html` from Appendix A verbatim.

⚠️ **Never hand-edit `data-w-id` attributes.** Webflow's IX2 runtime binds its scroll and hover
animations to them. Change or drop one and that element's animation dies silently.

---

## 7. The customizations

These are the changes applied on top of the stock template. If you are rebuilding from the raw
Webflow export rather than from Appendix A, apply all eight.

### 7.1 Rename the entry file to `index.html`
The export ships as `Kloudexa - Webflow HTML Website Template.html`. Rename it. See §2.

### 7.2 Share-preview / social metadata
Set the page title and the Open Graph + Twitter titles so link previews read correctly:

```html
<title>Meditations website reel</title>
<meta content="Meditations website reel" property="og:title"/>
<meta content="Meditations website reel" name="twitter:title"/>
```

> **Known loose end:** the `<meta name="description">` still contains the template's original
> copy ("Transform your business with Kloudexa's intelligent cloud services…"). That text also
> appears in link previews, under the title. Replace it with real copy.

### 7.3 Hide the "Made in Webflow" badge
Webflow's JS injects a floating badge into the bottom-right corner at runtime. It cannot be
deleted from the markup — it does not exist until the script runs — so suppress it with CSS:

```css
.w-webflow-badge { display: none !important; }
```

### 7.4 Remove the Radiant Templates promo widget
The export embeds a floating template-advertisement widget (a pill in the corner, a
"Get 180+ templates" tooltip, and a "Similar templates" panel). It has **three** parts, all of
which must go:
1. its `<style>` block,
2. its markup (`.floating-popup`, `#wandTooltip`, `.similar-panel`), and
3. its `<script>` plus the Lottie animation library it pulls from a CDN.

⚠️ **Keep one rule from its stylesheet:** `a { text-decoration: none; }`. It looks like it
belongs to the widget, but it is a **global** rule the whole page relies on. Delete it and
every link on the site sprouts an underline.

### 7.5 Footer: remove the "Support" column
Delete the entire column containing Style Guide / Licenses / Changelog / 404.

⚠️ **This is not just a markup deletion.** The footer is a CSS **grid** with a fixed number of
column tracks (despite its `w-layout-hflex` class name, it is `display: grid`). Desktop is 5
tracks: the left block spans 2, plus three link columns. Remove one column without removing a
track and you get a visible empty gap. Each breakpoint must drop a track — see the
`.rt-footer-top-wrapper` overrides in §7.9.

### 7.6 Footer: replace the credit line
Replace `Designed by Radiant Templates / Powered by Webflow` (and its two outbound links) with
plain text:

```html
<div class="rt-mobile-text-center">Design &amp; Developed by - Fastcreek AI</div>
```

### 7.7 Smooth scrolling (Lenis)
Adds inertia-based "hypersmooth" scrolling. Lenis drives the **real** scroll position, so
Webflow's scroll-triggered animations keep working. Loaded from a CDN, then:

```html
<script src="https://cdn.jsdelivr.net/npm/lenis@1.1.18/dist/lenis.min.js"></script>
<script>
(function () {
  var reduced = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
  if (!window.Lenis || reduced) return;

  var lenis = new Lenis({
    duration: 1.6,
    easing: function (t) { return Math.min(1, 1.001 - Math.pow(2, -10 * t)); },
    wheelMultiplier: 0.9,
    smoothWheel: true,
    syncTouch: false
  });
  window.__lenis = lenis;

  function raf(time) {
    lenis.raf(time);
    requestAnimationFrame(raf);
  }
  requestAnimationFrame(raf);

  // Route in-page anchor clicks through Lenis so jumps ease instead of snapping.
  document.querySelectorAll('a[href^="#"]').forEach(function (link) {
    link.addEventListener('click', function (e) {
      var target = document.querySelector(link.getAttribute('href'));
      if (!target) return;
      e.preventDefault();
      lenis.scrollTo(target, { offset: 0 });
    });
  });
})();
</script>
```

Notes:
- It respects `prefers-reduced-motion` and no-ops if the CDN fails.
- The instance is exposed as `window.__lenis` **on purpose** — the mobile menu needs it (§7.8).

### 7.8 Rebuild the navbar
The stock Webflow navbar (`.rt-navbar`, hover dropdowns, `w-nav` component) is **replaced**
with a custom one. Design: a sticky bar that **collapses into a floating, blurred pill on
scroll**.

Behavior:
- Scroll past **10px** → the header narrows from **1024px to 896px**, drops **16px** from the
  top, and gains a rounded border, translucent blurred background, and a shadow.
- **Mobile:** a two-bar icon that rotates into an X (45° / −45°, 300ms), opening a full-screen
  blurred overlay — links stacked at the top, buttons pinned to the bottom. Escape closes it;
  so does resizing to desktop.
- It is `position: fixed`, **not** `sticky`. The stock navbar was fixed and the hero's 210px
  top padding is calibrated to that. Sticky would take up layout space and shove the whole page
  down 64px.

⚠️ **The mobile menu must call `lenis.stop()`.** The usual `document.body.style.overflow =
'hidden'` does **not** hold the page still, because Lenis scrolls the page itself and will
happily keep scrolling behind the open overlay.

Markup:

```html
<header class="fc-header" id="fcHeader">
  <nav class="fc-nav">
    <a href="/" class="fc-brand" aria-label="Home">
    <img width="164" height="44" alt="brand" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/6835838adffe4868ef036767_logo.svg" loading="eager"/>
  </a>
  <div class="fc-nav-links">
    <a href="/home-one" class="fc-link" aria-current="page">Home</a>
    <a href="/about" class="fc-link">About</a>
    <a href="/features" class="fc-link">Features</a>
    <a href="/pricing-one" class="fc-link">Pricing</a>
    <a href="/blog-one" class="fc-link">Blog</a>
    <a href="/contact-one" class="fc-link">Contact</a>
  </div>
  <div class="fc-nav-actions">
    <div class="fc-socials">
      <a href="https://www.facebook.com" class="rt-icon-wrap w-inline-block" aria-label="Facebook">
      <img width="6" height="10" alt="facebook-icon" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/6835838adffe4868ef036768_facebbok.svg" loading="lazy"/>
    </a>
    <a href="https://dribbble.com" class="rt-icon-wrap w-inline-block" aria-label="Dribbble">
    <img width="11" height="11" alt="dribble-icon" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/6835838adffe4868ef036769_dribble.svg" loading="lazy"/>
  </a>
  <a href="https://in.linkedin.com" class="rt-icon-wrap w-inline-block" aria-label="LinkedIn">
  <img width="14" height="14" alt="linkedin-icon" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/6835838adffe4868ef03676a_linkedin.svg" loading="lazy"/>
</a>
</div>
<a data-wf--rt-button--variant="base" data-w-id="9f977b01-2ceb-c0cc-499c-caa5446b189c" href="/contact-one" class="rt-button w-inline-block">
<div class="w-layout-hflex rt-button-content rt-overflow-hidden">
<div class="rt-button-text-box rt-position-relative">
  <div class="rt-button-text">Start Free Trial</div>
</div>
<div class="rt-button-circle">
</div>
</div>
<div class="rt-button-linear">
</div>
</a>
</div>
<button type="button" class="fc-menu-toggle" id="fcMenuToggle" aria-label="Toggle menu" aria-expanded="false" aria-controls="fcMobileMenu">
<span class="fc-menu-icon">
<span>
</span>
<span>
</span>
</span>
</button>
</nav>
<div class="fc-mobile-menu" id="fcMobileMenu">
<div class="fc-mobile-inner">
<div class="fc-mobile-links">
<a href="/home-one" class="fc-link fc-link-block" aria-current="page">Home</a>
<a href="/about" class="fc-link fc-link-block">About</a>
<a href="/features" class="fc-link fc-link-block">Features</a>
<a href="/pricing-one" class="fc-link fc-link-block">Pricing</a>
<a href="/blog-one" class="fc-link fc-link-block">Blog</a>
<a href="/contact-one" class="fc-link fc-link-block">Contact</a>
</div>
<div class="fc-mobile-actions">
<div class="fc-socials">
<a href="https://www.facebook.com" class="rt-icon-wrap w-inline-block" aria-label="Facebook">
<img width="6" height="10" alt="facebook-icon" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/6835838adffe4868ef036768_facebbok.svg" loading="lazy"/>
</a>
<a href="https://dribbble.com" class="rt-icon-wrap w-inline-block" aria-label="Dribbble">
<img width="11" height="11" alt="dribble-icon" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/6835838adffe4868ef036769_dribble.svg" loading="lazy"/>
</a>
<a href="https://in.linkedin.com" class="rt-icon-wrap w-inline-block" aria-label="LinkedIn">
<img width="14" height="14" alt="linkedin-icon" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/6835838adffe4868ef03676a_linkedin.svg" loading="lazy"/>
</a>
</div>
<a data-wf--rt-button--variant="base" data-w-id="9f977b01-2ceb-c0cc-499c-caa5446b189c" href="/contact-one" class="rt-button w-inline-block">
<div class="w-layout-hflex rt-button-content rt-overflow-hidden">
<div class="rt-button-text-box rt-position-relative">
<div class="rt-button-text">Start Free Trial</div>
</div>
<div class="rt-button-circle">
</div>
</div>
<div class="rt-button-linear">
</div>
</a>
</div>
</div>
</div>
</header>
```

Behavior script:

```html
<script>
(function () {
  var reduced = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
  if (!window.Lenis || reduced) return;

  var lenis = new Lenis({
    duration: 1.6,
    easing: function (t) { return Math.min(1, 1.001 - Math.pow(2, -10 * t)); },
    wheelMultiplier: 0.9,
    smoothWheel: true,
    syncTouch: false
  });
  window.__lenis = lenis;

  function raf(time) {
    lenis.raf(time);
    requestAnimationFrame(raf);
  }
  requestAnimationFrame(raf);

  // Route in-page anchor clicks through Lenis so jumps ease instead of snapping.
  document.querySelectorAll('a[href^="#"]').forEach(function (link) {
    link.addEventListener('click', function (e) {
      var target = document.querySelector(link.getAttribute('href'));
      if (!target) return;
      e.preventDefault();
      lenis.scrollTo(target, { offset: 0 });
    });
  });
})();
</script>
```

The CTA (`.rt-button`, "Start Free Trial") is reused from the template **byte-for-byte**,
including its `data-w-id`, so its Webflow hover animation survives. Because the stock button is
hero-sized (`padding: 13px 40px` at 16px text ≈ 48px tall), it is shrunk **only inside the nav
bar** — the copy in the mobile menu stays full-size, where it is full-width.

### 7.9 All custom CSS

Every custom style, verbatim. It lives in a `<style>` block in `<head>`, placed **after** the
Webflow stylesheet link so it wins the cascade:

```css
  /* Webflow attribution badge */
  .w-webflow-badge { display: none !important; }

  /* Footer grid: the Support column was removed, so each breakpoint drops one track. */
  .rt-footer-top-wrapper { grid-template-columns: 1fr 1fr 1fr 1fr; }
  @media screen and (max-width: 991px) {
    .rt-footer-top-wrapper { grid-template-columns: 1.8fr 1fr 1fr; }
  }
  @media screen and (max-width: 767px) {
    .rt-footer-top-wrapper { grid-template-columns: 1fr 1fr; }
    #w-node-_3d0008a9-c775-29ca-2610-f2534409d4e8-4409d4e5 { grid-column: span 2 / span 2; }
  }

  /* Lenis smooth scroll */
  html.lenis, html.lenis body { height: auto; }
  .lenis.lenis-smooth { scroll-behavior: auto !important; }
  .lenis.lenis-stopped { overflow: hidden; }

  /* ------------------ Navbar ------------------ */
  .fc-header {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    z-index: 999;
    width: 100%;
    max-width: 1024px;
    margin-inline: auto;
    border: 1px solid transparent;
    border-radius: 0;
  }
  @media (min-width: 768px) {
    .fc-header {
      border-radius: 10px;
      transition: max-width .35s cubic-bezier(.22,1,.36,1), top .35s cubic-bezier(.22,1,.36,1),
                  background-color .35s ease-out, border-color .35s ease-out, box-shadow .35s ease-out;
    }
  }

  /* Scrolled: collapse into a floating, blurred pill. */
  .fc-header.is-scrolled,
  .fc-header.is-open {
    background-color: rgba(0, 0, 0, .95);
    border-color: rgba(255, 255, 255, .12);
    backdrop-filter: blur(16px);
    -webkit-backdrop-filter: blur(16px);
  }
  @supports ((backdrop-filter: blur(1px)) or (-webkit-backdrop-filter: blur(1px))) {
    .fc-header.is-scrolled { background-color: rgba(0, 0, 0, .6); }
    .fc-header.is-open { background-color: rgba(0, 0, 0, .9); }
  }
  @media (min-width: 768px) {
    .fc-header.is-scrolled {
      top: 16px;
      max-width: 896px;
      box-shadow: 0 10px 30px rgba(0, 0, 0, .45);
    }
    .fc-header.is-scrolled .fc-nav { padding-left: 8px; padding-right: 8px; }
  }

  .fc-nav {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 16px;
    width: 100%;
    height: 64px;
    padding: 0 16px;
  }
  @media (min-width: 768px) {
    .fc-nav { transition: padding .35s cubic-bezier(.22,1,.36,1); }
  }

  .fc-brand { display: flex; align-items: center; flex: none; }
  .fc-brand img { display: block; width: auto; height: 30px; }

  .fc-nav-links,
  .fc-nav-actions { display: none; }
  @media (min-width: 768px) {
    .fc-nav-links { display: flex; align-items: center; gap: 2px; }
    .fc-nav-actions { display: flex; align-items: center; gap: 14px; }
  }

  /* Ghost link */
  .fc-link {
    display: inline-flex;
    align-items: center;
    height: 34px;
    padding: 0 12px;
    border-radius: 7px;
    color: var(--light-slate-gray, #a3a8b8);
    font-size: 15px;
    font-weight: 500;
    text-transform: capitalize;
    white-space: nowrap;
    transition: background-color .2s ease, color .2s ease;
  }
  .fc-link:hover { background-color: rgba(255, 255, 255, .08); color: #fff; }
  .fc-link[aria-current="page"] { color: #fff; }
  .fc-link-block { justify-content: flex-start; height: 44px; font-size: 17px; }

  .fc-socials { display: flex; align-items: center; gap: 8px; }

  /* Compact CTA: the stock .rt-button is hero-sized (13px/40px padding at 16px text). */
  .fc-nav .rt-button-content { padding: 9px 22px; }
  .fc-nav .rt-button-text { font-size: 14px; line-height: 1.45; }

  /* Menu toggle (two bars that cross into an X) */
  .fc-menu-toggle {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 40px;
    height: 40px;
    padding: 0;
    background: transparent;
    border: 1px solid rgba(255, 255, 255, .18);
    border-radius: 8px;
    cursor: pointer;
    color: #fff;
  }
  .fc-menu-toggle:hover { background-color: rgba(255, 255, 255, .08); }
  @media (min-width: 768px) { .fc-menu-toggle { display: none; } }

  .fc-menu-icon { position: relative; display: block; width: 20px; height: 20px; }
  .fc-menu-icon span {
    position: absolute;
    left: 1px;
    right: 1px;
    height: 2px;
    border-radius: 2px;
    background-color: currentColor;
    transition: top .3s ease, transform .3s ease;
  }
  .fc-menu-icon span:nth-child(1) { top: 6px; }
  .fc-menu-icon span:nth-child(2) { top: 12px; }
  .fc-header.is-open .fc-menu-icon span:nth-child(1) { top: 9px; transform: rotate(45deg); }
  .fc-header.is-open .fc-menu-icon span:nth-child(2) { top: 9px; transform: rotate(-45deg); }

  /* Mobile overlay menu */
  .fc-mobile-menu {
    display: none;
    position: fixed;
    top: 64px;
    left: 0;
    right: 0;
    bottom: 0;
    z-index: 998;
    background-color: rgba(0, 0, 0, .92);
    backdrop-filter: blur(16px);
    -webkit-backdrop-filter: blur(16px);
    border-top: 1px solid rgba(255, 255, 255, .12);
    overflow: hidden auto;
  }
  .fc-header.is-open .fc-mobile-menu {
    display: block;
    animation: fcMenuIn .25s cubic-bezier(.22,1,.36,1);
  }
  @media (min-width: 768px) { .fc-mobile-menu { display: none !important; } }

  .fc-mobile-inner {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    gap: 24px;
    min-height: 100%;
    padding: 16px;
  }
  .fc-mobile-links { display: grid; gap: 4px; }
  .fc-mobile-actions { display: flex; flex-direction: column; align-items: stretch; gap: 16px; }
  .fc-mobile-actions .fc-socials { justify-content: center; }
  .fc-mobile-actions .rt-button { width: 100%; justify-content: center; }

  @keyframes fcMenuIn {
    from { opacity: 0; transform: scale(.97); }
    to { opacity: 1; transform: none; }
  }
```

---

## 8. Verification checklist

After building, confirm:

- [ ] `http://localhost:8080/` serves the page directly — **not** a directory listing.
- [ ] No "Made in Webflow" badge in the bottom-right corner.
- [ ] No floating promo pill / "Similar templates" panel.
- [ ] Links are **not** underlined (proves §7.4's `a { text-decoration: none; }` survived).
- [ ] Footer shows three columns with **no empty gap** — check desktop, tablet, and mobile.
- [ ] Footer credit reads "Design & Developed by - Fastcreek AI".
- [ ] Scrolling is smooth and weighted, not snappy.
- [ ] Scrolling down collapses the navbar into a floating pill; scrolling back to the top
      expands it flush again.
- [ ] The "Start Free Trial" button fits inside the nav bar without crowding it.
- [ ] On a narrow viewport, the hamburger morphs into an X, the overlay opens, and **the page
      behind it does not scroll**.
- [ ] Browser console is free of errors.

---

## 9. Known limitations

- **The nav links 404.** `/about`, `/features`, `/pricing-one`, `/blog-one`, `/contact-one`
  point at pages that do not exist — only the homepage was ever exported. Either build those
  pages, repoint the links at on-page sections, or remove them.
- **The dropdowns are gone.** The stock navbar had five hover dropdowns (Home ▸ home one/two/
  three, Pages ▸ 8 items, etc.). They were flattened to six top-level links, since every
  sub-link 404'd anyway.
- **The meta description still says "Kloudexa"** — see §7.2.
- **Offline = broken.** See §1.

---

## Appendix A — `index.html`

The complete file, verbatim. Copy this into `index.html` and you have the site.

```html
<!DOCTYPE html><!-- This site was created in Webflow. https://webflow.com --><!-- Last Published: Wed Mar 18 2026 03:46:52 GMT+0000 (Coordinated Universal Time) --><html data-wf-domain="kloudexa.webflow.io" data-wf-page="6835811ddc64590676a5b949" data-wf-site="68036da11dd5436718772525" lang="en"><head><meta charset="utf-8"/><link href="https://cdn.prod.website-files.com" rel="preconnect" crossorigin="anonymous"/><title>Meditations website reel</title><meta content="Transform your business with Kloudexa&#x27;s intelligent cloud services—designed for agility, performance, and seamless digital growth." name="description"/><meta content="Meditations website reel" property="og:title"/><meta content="Transform your business with Kloudexa&#x27;s intelligent cloud services—designed for agility, performance, and seamless digital growth." property="og:description"/><meta content="Meditations website reel" name="twitter:title"/><meta content="Transform your business with Kloudexa&#x27;s intelligent cloud services—designed for agility, performance, and seamless digital growth." name="twitter:description"/><meta property="og:type" content="website"/><meta content="summary_large_image" name="twitter:card"/><meta content="width=device-width, initial-scale=1" name="viewport"/><meta content="Webflow" name="generator"/><link href="https://cdn.prod.website-files.com/68036da11dd5436718772525/css/kloudexa.webflow.shared.87c665f93.min.css" rel="stylesheet" type="text/css" integrity="sha384-h8Zl+TxVh3KnK0F3gPRV445k7+A0e1jz8S1T92HkUQeyRqihPDz9azmW/AUAN/gA" crossorigin="anonymous"/><style>@media (min-width:992px) {html.w-mod-js:not(.w-mod-ix) [data-w-id="9e5ef46f-e0c1-0ffa-ab2c-d4eb1d81e308"] {opacity:0;}html.w-mod-js:not(.w-mod-ix) [data-w-id="9e5ef46f-e0c1-0ffa-ab2c-d4eb1d81e305"] {opacity:0;}html.w-mod-js:not(.w-mod-ix) [data-w-id="ac55e6fa-0092-9744-23e1-5d2e97fa86cd"] {opacity:0;}html.w-mod-js:not(.w-mod-ix) [data-w-id="ac55e6fa-0092-9744-23e1-5d2e97fa86d2"] {opacity:0;}html.w-mod-js:not(.w-mod-ix) [data-w-id="b7fb1ccd-310c-c292-6141-30af1e7a4301"] {opacity:0;}html.w-mod-js:not(.w-mod-ix) [data-w-id="b7fb1ccd-310c-c292-6141-30af1e7a430b"] {opacity:0;}html.w-mod-js:not(.w-mod-ix) [data-w-id="b7fb1ccd-310c-c292-6141-30af1e7a4307"] {opacity:0;}html.w-mod-js:not(.w-mod-ix) [data-w-id="ac55e6fa-0092-9744-23e1-5d2e97fa86fb"] {width:0px;}html.w-mod-js:not(.w-mod-ix) [data-w-id="ac55e6fa-0092-9744-23e1-5d2e97fa8702"] {-webkit-transform:translate3d(0, 0, 0) scale3d(0, 0, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);-moz-transform:translate3d(0, 0, 0) scale3d(0, 0, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);-ms-transform:translate3d(0, 0, 0) scale3d(0, 0, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);transform:translate3d(0, 0, 0) scale3d(0, 0, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);}html.w-mod-js:not(.w-mod-ix) [data-w-id="ac55e6fa-0092-9744-23e1-5d2e97fa8700"] {-webkit-transform:translate3d(0, 0, 0) scale3d(0, 0, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);-moz-transform:translate3d(0, 0, 0) scale3d(0, 0, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);-ms-transform:translate3d(0, 0, 0) scale3d(0, 0, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);transform:translate3d(0, 0, 0) scale3d(0, 0, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);}html.w-mod-js:not(.w-mod-ix) [data-w-id="ac55e6fa-0092-9744-23e1-5d2e97fa86fe"] {-webkit-transform:translate3d(0, 0, 0) scale3d(0, 0, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);-moz-transform:translate3d(0, 0, 0) scale3d(0, 0, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);-ms-transform:translate3d(0, 0, 0) scale3d(0, 0, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);transform:translate3d(0, 0, 0) scale3d(0, 0, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);}html.w-mod-js:not(.w-mod-ix) [data-w-id="847360cb-a1ab-f1d7-7487-e3c398fd0f41"] {width:0px;}html.w-mod-js:not(.w-mod-ix) [data-w-id="3aa4b5e6-e844-7118-0002-ebf4ee26c9a8"] {width:0px;}html.w-mod-js:not(.w-mod-ix) [data-w-id="a43b2b34-6759-8581-9bd3-234637420995"] {width:0px;}html.w-mod-js:not(.w-mod-ix) [data-w-id="c5521f26-28ae-8819-1cf8-92a3b4f3e013"] {width:0px;}html.w-mod-js:not(.w-mod-ix) [data-w-id="a7217904-4456-9784-c6d0-b700bdfd7950"] {width:0%;}html.w-mod-js:not(.w-mod-ix) [data-w-id="9e5ef46f-e0c1-0ffa-ab2c-d4eb1d81e312"] {-webkit-transform:translate3d(0, 0, 0) scale3d(1.15, 1.15, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);-moz-transform:translate3d(0, 0, 0) scale3d(1.15, 1.15, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);-ms-transform:translate3d(0, 0, 0) scale3d(1.15, 1.15, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);transform:translate3d(0, 0, 0) scale3d(1.15, 1.15, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);}html.w-mod-js:not(.w-mod-ix) [data-w-id="9e5ef46f-e0c1-0ffa-ab2c-d4eb1d81e314"] {-webkit-transform:translate3d(0, 50%, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);-moz-transform:translate3d(0, 50%, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);-ms-transform:translate3d(0, 50%, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);transform:translate3d(0, 50%, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);opacity:0;}html.w-mod-js:not(.w-mod-ix) [data-w-id="cade29ae-b64c-b5e7-06ee-b23c4ad40107"] {-webkit-transform:translate3d(0px, 0, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);-moz-transform:translate3d(0px, 0, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);-ms-transform:translate3d(0px, 0, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);transform:translate3d(0px, 0, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);}html.w-mod-js:not(.w-mod-ix) [data-w-id="181eb98f-46da-6b9f-e36c-26e50bdce346"] {-webkit-transform:translate3d(0px, 0, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);-moz-transform:translate3d(0px, 0, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);-ms-transform:translate3d(0px, 0, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);transform:translate3d(0px, 0, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);}html.w-mod-js:not(.w-mod-ix) [data-w-id="7368a5ea-1360-fcab-3f78-796294980440"] {width:0px;}}@media (max-width:991px) and (min-width:768px) {html.w-mod-js:not(.w-mod-ix) [data-w-id="9e5ef46f-e0c1-0ffa-ab2c-d4eb1d81e314"] {opacity:0;}}@media (max-width:767px) and (min-width:480px) {html.w-mod-js:not(.w-mod-ix) [data-w-id="9e5ef46f-e0c1-0ffa-ab2c-d4eb1d81e314"] {opacity:0;}}@media (max-width:479px) {html.w-mod-js:not(.w-mod-ix) [data-w-id="9e5ef46f-e0c1-0ffa-ab2c-d4eb1d81e314"] {opacity:0;}}</style><link href="https://fonts.googleapis.com" rel="preconnect"/><link href="https://fonts.gstatic.com" rel="preconnect" crossorigin="anonymous"/><script src="https://ajax.googleapis.com/ajax/libs/webfont/1.6.26/webfont.js" type="text/javascript"></script><script type="text/javascript">WebFont.load({  google: {    families: ["Inter:regular,500","Space Grotesk:regular,500"]  }});</script><script type="text/javascript">!function(o,c){var n=c.documentElement,t=" w-mod-";n.className+=t+"js",("ontouchstart"in o||o.DocumentTouch&&c instanceof DocumentTouch)&&(n.className+=t+"touch")}(window,document);</script><link href="https://cdn.prod.website-files.com/68036da11dd5436718772525/6853a0655dc3befe308bea02_32%20x%2032.svg" rel="shortcut icon" type="image/x-icon"/><link href="https://cdn.prod.website-files.com/68036da11dd5436718772525/6853a0656d7323cf257edf22_256%20x%20256.svg" rel="apple-touch-icon"/><style>
a { text-decoration: none; }
</style>
<style>
  /* Webflow attribution badge */
  .w-webflow-badge { display: none !important; }

  /* Footer grid: the Support column was removed, so each breakpoint drops one track. */
  .rt-footer-top-wrapper { grid-template-columns: 1fr 1fr 1fr 1fr; }
  @media screen and (max-width: 991px) {
    .rt-footer-top-wrapper { grid-template-columns: 1.8fr 1fr 1fr; }
  }
  @media screen and (max-width: 767px) {
    .rt-footer-top-wrapper { grid-template-columns: 1fr 1fr; }
    #w-node-_3d0008a9-c775-29ca-2610-f2534409d4e8-4409d4e5 { grid-column: span 2 / span 2; }
  }

  /* Lenis smooth scroll */
  html.lenis, html.lenis body { height: auto; }
  .lenis.lenis-smooth { scroll-behavior: auto !important; }
  .lenis.lenis-stopped { overflow: hidden; }

  /* ------------------ Navbar ------------------ */
  .fc-header {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    z-index: 999;
    width: 100%;
    max-width: 1024px;
    margin-inline: auto;
    border: 1px solid transparent;
    border-radius: 0;
  }
  @media (min-width: 768px) {
    .fc-header {
      border-radius: 10px;
      transition: max-width .35s cubic-bezier(.22,1,.36,1), top .35s cubic-bezier(.22,1,.36,1),
                  background-color .35s ease-out, border-color .35s ease-out, box-shadow .35s ease-out;
    }
  }

  /* Scrolled: collapse into a floating, blurred pill. */
  .fc-header.is-scrolled,
  .fc-header.is-open {
    background-color: rgba(0, 0, 0, .95);
    border-color: rgba(255, 255, 255, .12);
    backdrop-filter: blur(16px);
    -webkit-backdrop-filter: blur(16px);
  }
  @supports ((backdrop-filter: blur(1px)) or (-webkit-backdrop-filter: blur(1px))) {
    .fc-header.is-scrolled { background-color: rgba(0, 0, 0, .6); }
    .fc-header.is-open { background-color: rgba(0, 0, 0, .9); }
  }
  @media (min-width: 768px) {
    .fc-header.is-scrolled {
      top: 16px;
      max-width: 896px;
      box-shadow: 0 10px 30px rgba(0, 0, 0, .45);
    }
    .fc-header.is-scrolled .fc-nav { padding-left: 8px; padding-right: 8px; }
  }

  .fc-nav {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 16px;
    width: 100%;
    height: 64px;
    padding: 0 16px;
  }
  @media (min-width: 768px) {
    .fc-nav { transition: padding .35s cubic-bezier(.22,1,.36,1); }
  }

  .fc-brand { display: flex; align-items: center; flex: none; }
  .fc-brand img { display: block; width: auto; height: 30px; }

  .fc-nav-links,
  .fc-nav-actions { display: none; }
  @media (min-width: 768px) {
    .fc-nav-links { display: flex; align-items: center; gap: 2px; }
    .fc-nav-actions { display: flex; align-items: center; gap: 14px; }
  }

  /* Ghost link */
  .fc-link {
    display: inline-flex;
    align-items: center;
    height: 34px;
    padding: 0 12px;
    border-radius: 7px;
    color: var(--light-slate-gray, #a3a8b8);
    font-size: 15px;
    font-weight: 500;
    text-transform: capitalize;
    white-space: nowrap;
    transition: background-color .2s ease, color .2s ease;
  }
  .fc-link:hover { background-color: rgba(255, 255, 255, .08); color: #fff; }
  .fc-link[aria-current="page"] { color: #fff; }
  .fc-link-block { justify-content: flex-start; height: 44px; font-size: 17px; }

  .fc-socials { display: flex; align-items: center; gap: 8px; }

  /* Compact CTA: the stock .rt-button is hero-sized (13px/40px padding at 16px text). */
  .fc-nav .rt-button-content { padding: 9px 22px; }
  .fc-nav .rt-button-text { font-size: 14px; line-height: 1.45; }

  /* Menu toggle (two bars that cross into an X) */
  .fc-menu-toggle {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 40px;
    height: 40px;
    padding: 0;
    background: transparent;
    border: 1px solid rgba(255, 255, 255, .18);
    border-radius: 8px;
    cursor: pointer;
    color: #fff;
  }
  .fc-menu-toggle:hover { background-color: rgba(255, 255, 255, .08); }
  @media (min-width: 768px) { .fc-menu-toggle { display: none; } }

  .fc-menu-icon { position: relative; display: block; width: 20px; height: 20px; }
  .fc-menu-icon span {
    position: absolute;
    left: 1px;
    right: 1px;
    height: 2px;
    border-radius: 2px;
    background-color: currentColor;
    transition: top .3s ease, transform .3s ease;
  }
  .fc-menu-icon span:nth-child(1) { top: 6px; }
  .fc-menu-icon span:nth-child(2) { top: 12px; }
  .fc-header.is-open .fc-menu-icon span:nth-child(1) { top: 9px; transform: rotate(45deg); }
  .fc-header.is-open .fc-menu-icon span:nth-child(2) { top: 9px; transform: rotate(-45deg); }

  /* Mobile overlay menu */
  .fc-mobile-menu {
    display: none;
    position: fixed;
    top: 64px;
    left: 0;
    right: 0;
    bottom: 0;
    z-index: 998;
    background-color: rgba(0, 0, 0, .92);
    backdrop-filter: blur(16px);
    -webkit-backdrop-filter: blur(16px);
    border-top: 1px solid rgba(255, 255, 255, .12);
    overflow: hidden auto;
  }
  .fc-header.is-open .fc-mobile-menu {
    display: block;
    animation: fcMenuIn .25s cubic-bezier(.22,1,.36,1);
  }
  @media (min-width: 768px) { .fc-mobile-menu { display: none !important; } }

  .fc-mobile-inner {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    gap: 24px;
    min-height: 100%;
    padding: 16px;
  }
  .fc-mobile-links { display: grid; gap: 4px; }
  .fc-mobile-actions { display: flex; flex-direction: column; align-items: stretch; gap: 16px; }
  .fc-mobile-actions .fc-socials { justify-content: center; }
  .fc-mobile-actions .rt-button { width: 100%; justify-content: center; }

  @keyframes fcMenuIn {
    from { opacity: 0; transform: scale(.97); }
    to { opacity: 1; transform: none; }
  }
</style>
</head><body><header class="fc-header" id="fcHeader"><nav class="fc-nav"><a href="/" class="fc-brand" aria-label="Home"><img width="164" height="44" alt="brand" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/6835838adffe4868ef036767_logo.svg" loading="eager"/></a><div class="fc-nav-links"><a href="/home-one" class="fc-link" aria-current="page">Home</a><a href="/about" class="fc-link">About</a><a href="/features" class="fc-link">Features</a><a href="/pricing-one" class="fc-link">Pricing</a><a href="/blog-one" class="fc-link">Blog</a><a href="/contact-one" class="fc-link">Contact</a></div><div class="fc-nav-actions"><div class="fc-socials"><a href="https://www.facebook.com" class="rt-icon-wrap w-inline-block" aria-label="Facebook"><img width="6" height="10" alt="facebook-icon" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/6835838adffe4868ef036768_facebbok.svg" loading="lazy"/></a><a href="https://dribbble.com" class="rt-icon-wrap w-inline-block" aria-label="Dribbble"><img width="11" height="11" alt="dribble-icon" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/6835838adffe4868ef036769_dribble.svg" loading="lazy"/></a><a href="https://in.linkedin.com" class="rt-icon-wrap w-inline-block" aria-label="LinkedIn"><img width="14" height="14" alt="linkedin-icon" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/6835838adffe4868ef03676a_linkedin.svg" loading="lazy"/></a></div><a data-wf--rt-button--variant="base" data-w-id="9f977b01-2ceb-c0cc-499c-caa5446b189c" href="/contact-one" class="rt-button w-inline-block"><div class="w-layout-hflex rt-button-content rt-overflow-hidden"><div class="rt-button-text-box rt-position-relative"><div class="rt-button-text">Start Free Trial</div></div><div class="rt-button-circle"></div></div><div class="rt-button-linear"></div></a></div><button type="button" class="fc-menu-toggle" id="fcMenuToggle" aria-label="Toggle menu" aria-expanded="false" aria-controls="fcMobileMenu"><span class="fc-menu-icon"><span></span><span></span></span></button></nav><div class="fc-mobile-menu" id="fcMobileMenu"><div class="fc-mobile-inner"><div class="fc-mobile-links"><a href="/home-one" class="fc-link fc-link-block" aria-current="page">Home</a><a href="/about" class="fc-link fc-link-block">About</a><a href="/features" class="fc-link fc-link-block">Features</a><a href="/pricing-one" class="fc-link fc-link-block">Pricing</a><a href="/blog-one" class="fc-link fc-link-block">Blog</a><a href="/contact-one" class="fc-link fc-link-block">Contact</a></div><div class="fc-mobile-actions"><div class="fc-socials"><a href="https://www.facebook.com" class="rt-icon-wrap w-inline-block" aria-label="Facebook"><img width="6" height="10" alt="facebook-icon" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/6835838adffe4868ef036768_facebbok.svg" loading="lazy"/></a><a href="https://dribbble.com" class="rt-icon-wrap w-inline-block" aria-label="Dribbble"><img width="11" height="11" alt="dribble-icon" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/6835838adffe4868ef036769_dribble.svg" loading="lazy"/></a><a href="https://in.linkedin.com" class="rt-icon-wrap w-inline-block" aria-label="LinkedIn"><img width="14" height="14" alt="linkedin-icon" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/6835838adffe4868ef03676a_linkedin.svg" loading="lazy"/></a></div><a data-wf--rt-button--variant="base" data-w-id="9f977b01-2ceb-c0cc-499c-caa5446b189c" href="/contact-one" class="rt-button w-inline-block"><div class="w-layout-hflex rt-button-content rt-overflow-hidden"><div class="rt-button-text-box rt-position-relative"><div class="rt-button-text">Start Free Trial</div></div><div class="rt-button-circle"></div></div><div class="rt-button-linear"></div></a></div></div></div></header><section data-w-id="9e5ef46f-e0c1-0ffa-ab2c-d4eb1d81e2f5" class="rt-hero-v1"><div class="w-layout-blockcontainer rt-container w-container"><div class="w-layout-vflex rt-hero-v1-top-part rt-overflow-hidden"><div data-w-id="9e5ef46f-e0c1-0ffa-ab2c-d4eb1d81e2f7" style="opacity:0" class="w-layout-hflex rt-sub-text-bottom-gap"><div data-w-id="d126363a-8414-9aab-87c1-bda33ca993d6" class="w-layout-hflex rt-sub-text-main rt-position-relative"><div class="w-layout-hflex rt-sub-text-wrapper"><div class="rt-dot"></div><div class="rt-sub-text">Instant Access</div></div><div class="rt-button-linear color-change"></div></div></div><div data-w-id="9e5ef46f-e0c1-0ffa-ab2c-d4eb1d81e2fe" style="opacity:0" class="rt-hero-v1-heading rt-desktop-text-center rt-heading-bottom-gap"><h1 class="rt-text-gradiant rt-no-margin">Simplify Your Workflow With One <span class="text-span"><em class="rt-italic-text">Powerful</em></span> Platform</h1></div><div data-w-id="9e5ef46f-e0c1-0ffa-ab2c-d4eb1d81e305" class="rt-hero-v1-para"><p class="rt-no-margin rt-desktop-text-center">Elit nec suspendisse placerat volutpat quisque vestibulum fusce sem at. Diam sagittis vitae tristique in in diam in. Proin est posuere neque a non pretium</p></div><div data-w-id="9e5ef46f-e0c1-0ffa-ab2c-d4eb1d81e308" class="w-layout-hflex rt-hero-v1-button-wrapper"><a data-wf--rt-button--variant="color-blue" data-w-id="9f977b01-2ceb-c0cc-499c-caa5446b189c" href="/contact-two" class="rt-button w-inline-block"><div class="w-layout-hflex rt-button-content w-variant-3e2f90de-5e76-c3ff-e2ef-f9fca0fabd74 rt-overflow-hidden"><div class="rt-button-text-box rt-position-relative"><div class="rt-button-text">get in touch</div></div><div class="rt-button-circle w-variant-3e2f90de-5e76-c3ff-e2ef-f9fca0fabd74"></div></div><div class="rt-button-linear"></div></a><a data-wf--rt-button--variant="base" data-w-id="9f977b01-2ceb-c0cc-499c-caa5446b189c" href="/about" class="rt-button w-inline-block"><div class="w-layout-hflex rt-button-content rt-overflow-hidden"><div class="rt-button-text-box rt-position-relative"><div class="rt-button-text">Discover More</div></div><div class="rt-button-circle"></div></div><div class="rt-button-linear"></div></a></div></div><div data-w-id="9e5ef46f-e0c1-0ffa-ab2c-d4eb1d81e311" class="w-layout-vflex rt-hero-v1-bottom-part rt-position-relative"><div class="w-layout-vflex rt-hero-v1-circle-main"><div class="rt-hero-v1-circle-line-wrap rt-tab-display-none"><div data-w-id="a7217904-4456-9784-c6d0-b700bdfd7950" class="rt-hero-v1-circle-border-line"><img src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683e89979b63a6f1f0dc4cbb_hero%20circle%20border.svg" loading="lazy" width="800" height="304" alt="line " class="rt-hero-v1-circle-line"/></div></div><div data-w-id="9e5ef46f-e0c1-0ffa-ab2c-d4eb1d81e312" class="rt-hero-v1-circle-image"><img width="745" height="741" alt="hero circle
" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/685b9865855ad767bf0080bf_hero%20circle.svg" loading="lazy" class="rt-auto-fit rt-desktop-full-width circle-image"/></div><div class="w-layout-vflex rt-hero-v1-small-line-box rt-tab-display-none"><div data-w-id="847360cb-a1ab-f1d7-7487-e3c398fd0f41" class="w-layout-hflex rt-hero-v1-small-line one"><img src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683e957635bd18d00e96cdcc_hero%20line%201.svg" loading="lazy" width="233" height="169" alt="line " class="rt-max-width-none"/></div><div data-w-id="c5521f26-28ae-8819-1cf8-92a3b4f3e013" class="w-layout-hflex rt-hero-v1-small-line two"><img src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683e89974399d4e23db7944d_hero%20line%202.svg" loading="lazy" width="237" height="247" alt="line " class="rt-max-width-none"/></div><div data-w-id="a43b2b34-6759-8581-9bd3-234637420995" class="w-layout-hflex rt-hero-v1-small-line three"><img src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683e8997575b494575774ff8_71b7883890fde9e502d5a50e1ddac864_hero%20line%203.svg" loading="lazy" width="566" height="289" alt="line " class="rt-max-width-none"/></div><div data-w-id="3aa4b5e6-e844-7118-0002-ebf4ee26c9a8" class="w-layout-hflex rt-hero-v1-small-line for"><img src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683e9ab0367c4f3422f19f57_hero%20line%204.svg" loading="lazy" width="199" height="227" alt="line " class="rt-max-width-none"/></div></div></div><div data-w-id="9e5ef46f-e0c1-0ffa-ab2c-d4eb1d81e314" class="rt-hero-v1-image-board rt-radius-20 rt-overflow-hidden"><img width="1069" height="675" alt="hero dashboard " src="https://cdn.prod.website-files.com/68036da11dd5436718772525/685d180cdb2b73c6dc840add_hero%20dasbord.webp" loading="lazy" srcset="https://cdn.prod.website-files.com/68036da11dd5436718772525/685d180cdb2b73c6dc840add_hero%20dasbord-p-500.webp 500w, https://cdn.prod.website-files.com/68036da11dd5436718772525/685d180cdb2b73c6dc840add_hero%20dasbord-p-800.webp 800w, https://cdn.prod.website-files.com/68036da11dd5436718772525/685d180cdb2b73c6dc840add_hero%20dasbord-p-1080.webp 1080w, https://cdn.prod.website-files.com/68036da11dd5436718772525/685d180cdb2b73c6dc840add_hero%20dasbord-p-1600.webp 1600w, https://cdn.prod.website-files.com/68036da11dd5436718772525/685d180cdb2b73c6dc840add_hero%20dasbord-p-2000.webp 2000w, https://cdn.prod.website-files.com/68036da11dd5436718772525/685d180cdb2b73c6dc840add_hero%20dasbord.webp 2138w" sizes="(max-width: 767px) 100vw, (max-width: 991px) 728px, 940px" class="rt-auto-fit rt-desktop-full-width rt-radius-20"/></div></div></div></section><section data-w-id="4cb2141c-a793-b6c9-f827-0e96dcfe65ed" class="rt-clients"><div class="w-layout-blockcontainer rt-container w-container"><div class="rt-marquee-wrap rt-overflow-hidden"><div data-w-id="4cb2141c-a793-b6c9-f827-0e96dcfe65f0" style="opacity:0" class="w-layout-hflex rt-marquee-heading-wrap rt-overflow-hidden"><div>Our Trusted Clients</div></div><div class="w-layout-hflex rt-marquee-main"><div class="w-layout-hflex rt-marquee-train"><img width="136" height="35" alt="line " src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7d5cdebe0a37aa83443_logo-one.svg" loading="lazy"/><img width="163" height="38" alt="line " src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7d5cdebe0a37aa83444_logo-two.svg" loading="lazy"/><img width="122" height="37" alt="line " src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7d5cdebe0a37aa83445_logo-three.svg" loading="lazy"/><img width="168" height="37" alt="line " src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7d5cdebe0a37aa83446_logo-four.svg" loading="lazy"/><img width="130" height="30" alt="line " src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7d5cdebe0a37aa83447_logo-five.svg" loading="lazy"/></div><div class="w-layout-hflex rt-marquee-train"><img width="136" height="35" alt="line " src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7d5cdebe0a37aa83443_logo-one.svg" loading="lazy"/><img width="163" height="38" alt="line " src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7d5cdebe0a37aa83444_logo-two.svg" loading="lazy"/><img width="122" height="37" alt="line " src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7d5cdebe0a37aa83445_logo-three.svg" loading="lazy"/><img width="168" height="37" alt="line " src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7d5cdebe0a37aa83446_logo-four.svg" loading="lazy"/><img width="130" height="30" alt="line " src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7d5cdebe0a37aa83447_logo-five.svg" loading="lazy"/></div><div class="w-layout-hflex rt-marquee-train"><img width="136" height="35" alt="line " src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7d5cdebe0a37aa83443_logo-one.svg" loading="lazy"/><img width="163" height="38" alt="line " src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7d5cdebe0a37aa83444_logo-two.svg" loading="lazy"/><img width="122" height="37" alt="line " src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7d5cdebe0a37aa83445_logo-three.svg" loading="lazy"/><img width="168" height="37" alt="line " src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7d5cdebe0a37aa83446_logo-four.svg" loading="lazy"/><img width="130" height="30" alt="line " src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7d5cdebe0a37aa83447_logo-five.svg" loading="lazy"/></div></div></div></div></section><section class="rt-about-v1"><div class="rt-about-v1-container"><div class="w-layout-hflex rt-about-v1-main"><div class="w-layout-vflex rt-about-v1-left-part rt-mobile-text-center"><div data-w-id="ac55e6fa-0092-9744-23e1-5d2e97fa86c0" style="opacity:0" class="rt-sub-text-bottom-gap"><div data-w-id="d126363a-8414-9aab-87c1-bda33ca993d6" class="w-layout-hflex rt-sub-text-main rt-position-relative"><div class="w-layout-hflex rt-sub-text-wrapper"><div class="rt-dot"></div><div class="rt-sub-text">Abouit</div></div><div class="rt-button-linear color-change"></div></div></div><div data-w-id="ac55e6fa-0092-9744-23e1-5d2e97fa86c7" style="opacity:0" class="rt-about-v1-left-heading rt-heading-bottom-gap"><h2 class="rt-text-gradiant rt-no-margin">Optimize <em class="rt-italic-text">Operations</em> With Real-Time Analytics</h2></div><div data-w-id="ac55e6fa-0092-9744-23e1-5d2e97fa86cd" class="rt-about-para-wrap"><p>Elit nec suspendisse placerat volutpat quisque vestibulum fusce sem at. Diam sagittis vitae tristique in in diam in. Proin est posuere neque a non pretium<br/></p></div><div class="w-layout-vflex rt-about-point-box-main rt-overflow-hidden"><div data-w-id="ac55e6fa-0092-9744-23e1-5d2e97fa86d2" class="w-layout-hflex rt-about-icon-box"><div class="rt-about-icon-wrap"><img width="29" height="29" alt="icon" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7edba958d1509c4e19c_icon.svg" loading="lazy"/></div><div class="w-layout-vflex rt-about-icon-box-contant"><div class="rt-text-style-h5">Streamline Operations Instantly</div><p>Lorem ipsum dolor sit amet consectetur adipiscing elit sed eiusmod tempor incididunt magna aliqua.</p></div></div><div data-w-id="ac55e6fa-0092-9744-23e1-5d2e97fa86da" style="opacity:0" class="w-layout-hflex rt-about-icon-box"><div class="rt-about-icon-wrap"><img width="29" height="29" alt="icon" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7edba958d1509c4e19c_icon.svg" loading="lazy"/></div><div class="w-layout-vflex rt-about-icon-box-contant"><div class="rt-text-style-h5">Analyze Performance Efficiently</div><p>Lorem ipsum dolor sit amet consectetur adipiscing elit sed eiusmod tempor incididunt magna aliqua.</p></div></div></div><a data-wf--rt-button--variant="color-blue" data-w-id="9f977b01-2ceb-c0cc-499c-caa5446b189c" href="/contact-one" class="rt-button w-inline-block"><div class="w-layout-hflex rt-button-content w-variant-3e2f90de-5e76-c3ff-e2ef-f9fca0fabd74 rt-overflow-hidden"><div class="rt-button-text-box rt-position-relative"><div class="rt-button-text">get in touch</div></div><div class="rt-button-circle w-variant-3e2f90de-5e76-c3ff-e2ef-f9fca0fabd74"></div></div><div class="rt-button-linear"></div></a></div><div class="rt-about-right-part"><div class="rt-about-card-main"><div data-w-id="ac55e6fa-0092-9744-23e1-5d2e97fa86e7" style="opacity:0" class="w-layout-vflex rt-about-card one rt-radius-20"><div class="rt-text-color-white">Big Data Analytics</div><div class="rt-about-card-icon-wrap one"><div class="rt-about-card-icon"><img width="40" height="40" alt="line " src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7edba958d1509c4e19e_about-icon-one.svg" loading="lazy"/></div><div class="w-layout-vflex rt-about-card-text"><div class="rt-about-card-text-style">Solution</div><div>Lorem ipsum dolor sit amet</div></div></div><div class="rt-about-card-icon-wrap two"><div class="rt-about-card-icon"><img width="40" height="40" alt="icon" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/68540c6522995d9a8d337ee3_icon-1.svg" loading="lazy"/></div><div class="w-layout-vflex rt-about-card-text"><div class="rt-about-card-text-style">Data Security</div><div>Lorem ipsum dolor sit amet</div></div></div></div></div><div data-w-id="ac55e6fa-0092-9744-23e1-5d2e97fa86fa" class="rt-position-relative rt-about-line"><div data-w-id="ac55e6fa-0092-9744-23e1-5d2e97fa86fb" class="w-layout-hflex rt-about-line-image-wrapper"><img width="651" height="175" alt="line " src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7edba958d1509c4e19f_Vector%20823.svg" loading="lazy" class="rt-line-effect"/></div><div data-w-id="ac55e6fa-0092-9744-23e1-5d2e97fa86fe" class="rt-client-icon-one"><img width="63" height="63" alt="logo" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7edba958d1509c4e1b0_client-image-1.svg" loading="lazy"/></div><div data-w-id="ac55e6fa-0092-9744-23e1-5d2e97fa8700" class="rt-client-icon-two"><img width="63" height="63" alt="logo" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7edba958d1509c4e1b1_client-image-2.svg" loading="lazy"/></div><div data-w-id="ac55e6fa-0092-9744-23e1-5d2e97fa8702" class="rt-client-icon-three"><img width="63" height="63" alt="logo" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7edba958d1509c4e1b2_client-image-3.svg" loading="lazy"/></div></div><div data-w-id="ac55e6fa-0092-9744-23e1-5d2e97fa8704" style="opacity:0" class="rt-about-card-image-wrap rt-radius-20"><div class="rt-about-progress-image"><img width="285" height="128" alt="line " src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7edba958d1509c4e1b3_Group%201009004711%20(3).png" loading="lazy" class="rt-tab-image-full-width"/></div><div class="rt-about-card-heading-wrap-two"><div class="rt-about-card-heading-wrap">Ultrices dictum</div></div><div class="w-layout-hflex rt-about-card-grow"><div class="rt-card-grow-line-box"><div style="height:0px" class="rt-grow-card-bar-line one"></div></div><div class="rt-card-grow-line-box"><div style="height:0px" class="rt-grow-card-bar-line two"></div></div><div class="rt-card-grow-line-box"><div style="height:0px" class="rt-grow-card-bar-line three"></div></div><div class="rt-card-grow-line-box"><div style="height:0px" class="rt-grow-card-bar-line four"></div></div><div class="rt-card-grow-line-box"><div style="height:0px" class="rt-grow-card-bar-line five"></div></div><div class="rt-card-grow-line-box"><div style="height:0px" class="rt-grow-card-bar-line six"></div></div><div class="rt-card-grow-line-box"><div style="height:0px" class="rt-grow-card-bar-line seven"></div></div></div></div><div data-w-id="ac55e6fa-0092-9744-23e1-5d2e97fa8719" style="opacity:0" class="rt-about-card two rt-radius-20 rt-tab-display-none"><div class="rt-card-image-1"><img width="58" height="17" alt="logo" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7edba958d1509c4e1a9_Group%201009004713.png" loading="lazy" class="rt-image-move"/></div><div class="w-layout-hflex rt-about-card-image-wrapper"><img width="38" height="38" alt="logo" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7edba958d1509c4e1a1_Group%201000001978.png" loading="lazy"/><img width="38" height="38" alt="image" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7edba958d1509c4e1a3_Group%201000001973.png" loading="lazy"/><img width="38" height="38" alt="image" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7edba958d1509c4e1a5_Group%201000001974.png" loading="lazy"/><img width="38" height="38" alt="small image" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7edba958d1509c4e1a7_Group%201000001975.png" loading="lazy"/></div><div class="rt-card-image-wrap"><div class="rt-card-image-2"><img width="48" height="16" alt="logo" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7edba958d1509c4e1ab_Group%201000001980.png" loading="lazy" class="rt-image-move"/></div><div class="rt-card-image-3"><img width="35" height="15" alt="logo" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7edba958d1509c4e1ad_Group%201009004714%20(1).png" loading="lazy" class="rt-image-move"/></div></div></div></div></div></div></section><section class="rt-feature-v1 rt-position-relative"><div class="w-layout-hflex rt-top-style-part"><img width="1920" height="211" alt="svg " src="https://cdn.prod.website-files.com/68036da11dd5436718772525/6846cf6132513cd8ae4a4dae_top%20style.svg" loading="lazy" class="rt-auto-fit"/></div><div class="w-layout-blockcontainer rt-container w-container"><div class="w-layout-hflex rt-feature-heading-main"><div class="w-layout-vflex rt-heading-wrap"><div class="w-layout-hflex rt-sub-text-bottom-gap rt-center"><div data-w-id="d126363a-8414-9aab-87c1-bda33ca993d6" class="w-layout-hflex rt-sub-text-main rt-position-relative"><div class="w-layout-hflex rt-sub-text-wrapper"><div class="rt-dot"></div><div class="rt-sub-text">Features</div></div><div class="rt-button-linear color-change"></div></div></div><div data-w-id="7ff743a6-e168-3cc9-4110-98208a7f4cae" style="opacity:0" class="rt-heading-bottom-gap"><h2 class="rt-no-margin rt-text-gradiant rt-desktop-text-center">Regular <span class="rt-italic-text">Software</span> Updates For Continuous Improvement</h2></div><div data-w-id="9d8e130a-2eab-41c8-9271-019cb2322569" style="opacity:0" class="rt-feature-v1-para-wrap"><p class="rt-desktop-text-center rt-no-margin">Feugiat cursus non laoreet fringilla urna dictum pharetra nunc viverra Non duis tellus <br/></p></div></div></div><div class="rt-feature-card"><div class="w-layout-grid rt-feature-grid-one"><div data-w-id="9d8e130a-2eab-41c8-9271-019cb232256f" style="opacity:0" class="w-layout-vflex rt-feature-box rt-radius-20"><div class="w-layout-hflex rt-overlay-image-one rt-position-relative"><img width="384" height="225" alt="card image" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683ed928abbd877010c3b3ce_feature%20card.webp" loading="lazy" srcset="https://cdn.prod.website-files.com/68036da11dd5436718772525/683ed928abbd877010c3b3ce_feature%20card-p-500.webp 500w, https://cdn.prod.website-files.com/68036da11dd5436718772525/683ed928abbd877010c3b3ce_feature%20card.webp 696w" sizes="(max-width: 479px) 100vw, 384px" class="rt-tab-image-full-width"/><div class="rt-user-image-wrap-one"><img width="25" height="25" alt="image" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683d206cc19e082c7debebb9_Ellipse%2087.png" loading="lazy"/></div><div class="rt-user-name-wrap-one"><img width="49" height="15" alt="logo" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683d206cc19e082c7debebbc_user-name.svg" loading="lazy"/></div><div class="rt-user-image-wrap-two"><img width="25" height="25" alt="small img " src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683d206cc19e082c7debebbd_Ellipse%2086%20(1).svg" loading="lazy"/></div><div class="rt-user-name-wrap-two"><img width="48" height="15" alt="logo" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683d206cc19e082c7debebbe_user-name2.svg" loading="lazy"/></div></div><div class="w-layout-vflex rt-feature-card-heading-wrap rt-tab-text-center"><div class="rt-text-style-h5">Customizable User Interface</div><p class="rt-no-margin">Ornare arcu lorem tristique sed commodo diam nunc donec.<br/></p></div></div><div data-w-id="9d8e130a-2eab-41c8-9271-019cb232259d" style="opacity:0" class="w-layout-vflex rt-feature-box rt-position-relative rt-radius-20"><div data-w-id="9d8e130a-2eab-41c8-9271-019cb232259e" style="opacity:0" class="rt-feature-card-image-wrap rt-display-on"><img width="308" height="191" alt="card " src="https://cdn.prod.website-files.com/68036da11dd5436718772525/685d2758dae1a264fcd6651d_feature%20card%20.svg" loading="lazy" class="rt-auto-fit rt-radius-20 rt-tab-image-full-width border"/></div><div class="w-layout-vflex rt-feature-card-heading-wrap rt-tab-text-center"><div class="rt-text-style-h5">Streamlined Project Tracking</div><p class="rt-no-margin">Ornare arcu lorem tristique sed commodo diam nunc donec.<br/></p></div><div data-w-id="9d8e130a-2eab-41c8-9271-019cb23225a6" style="opacity:0" class="rt-absoulte-image-one rt-radius-10 rt-overflow-hidden rt-tab-display-none"><img width="158" height="83" alt="dashbord image" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683d206cc19e082c7debebc4_feature-card-image-3.webp" loading="lazy" class="rt-auto-fit"/></div><div data-w-id="9d8e130a-2eab-41c8-9271-019cb23225a8" style="opacity:0" class="rt-absoulte-image-two rt-radius-10 rt-overflow-hidden rt-tab-display-none"><img width="125" height="65" alt="image" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683d206cc19e082c7debebc2_feature-card-image-1.webp" loading="lazy" class="rt-auto-fit"/></div><div data-w-id="9d8e130a-2eab-41c8-9271-019cb23225aa" style="opacity:0" class="rt-absoulte-image-three rt-radius-10 rt-overflow-hidden rt-tab-display-none"><img width="154" height="94" alt="line bar" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683d206cc19e082c7debebc5_feature-card-image-2.webp" loading="lazy" class="rt-auto-fit"/></div></div><div data-w-id="9d8e130a-2eab-41c8-9271-019cb23225ac" style="opacity:0" class="w-layout-vflex rt-feature-box rt-position-relative rt-radius-20"><div class="w-layout-vflex rt-feature-box-top"><div class="w-layout-hflex rt-featire-card-top-heading-wrap"><div class="rt-sub-text rt-feature-card-text">Estate planning and wealth transfer<br/></div></div><img width="347" height="134" alt="card image" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683d206cc19e082c7debebc6_feature-image-5.webp" loading="lazy" class="rt-auto-fit rt-height-auto rt-tab-image-full-width"/></div><div class="w-layout-vflex rt-feature-card-heading-wrap rt-tab-text-center"><div class="rt-text-style-h5">Real-Time Collaboration Tools</div><p class="rt-no-margin">Ornare arcu lorem tristique sed commodo diam nunc donec.<br/></p></div></div></div><div class="w-layout-grid rt-feature-grid-two"><div data-w-id="9d8e130a-2eab-41c8-9271-019cb23225b9" style="opacity:0" class="w-layout-vflex rt-feature-card-box-two rt-radius-20"><div class="w-layout-hflex rt-feature-card-box-2-image"><img width="276" height="276" alt="image" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683d206cc19e082c7debebc7_feature-card-image-6.webp" loading="lazy" class="rt-tab-image-full-width"/></div><div class="w-layout-vflex rt-feature-card-heading-wrap rt-tab-text-center"><div class="rt-text-style-h5">Cloud Backup Services</div><p>Ornare arcu lorem tristique sed commodo diam nunc donec.<br/></p></div></div><div id="w-node-_9d8e130a-2eab-41c8-9271-019cb23225c1-76a5b949" data-w-id="9d8e130a-2eab-41c8-9271-019cb23225c1" style="opacity:0" class="w-layout-vflex rt-feature-card-box-three rt-position-relative rt-overflow-hidden rt-radius-20"><div class="w-layout-vflex rt-feature-card-heading-wrap rt-tab-text-center"><div class="rt-text-style-h5">Role-based Access Control</div><p>Ornare arcu lorem tristique sed commodo diam nunc donec.<br/></p></div><div data-w-id="9d8e130a-2eab-41c8-9271-019cb23225c8" class="w-layout-vflex rt-cloud-backup-box-bottom rt-radius-20"><div class="w-layout-vflex rt-image-cloud-top rt-tab-display-none"><img src="https://cdn.prod.website-files.com/68036da11dd5436718772525/685410b9bc4e3bda4759aeb1_image-1.svg" loading="lazy" width="255" height="68" alt="card" data-w-id="cade29ae-b64c-b5e7-06ee-b23c4ad40107" class="rt-auto-fit border rt-radius-10 cloud-imae-two"/><img src="https://cdn.prod.website-files.com/68036da11dd5436718772525/685410c78c08a5bac64e7b48_image-2.svg" loading="lazy" width="255" height="68" alt="card" data-w-id="181eb98f-46da-6b9f-e36c-26e50bdce346" class="rt-auto-fit border rt-radius-10 cloud-image-one"/></div><div data-w-id="7368a5ea-1360-fcab-3f78-796294980440" class="w-layout-hflex rt-image-cloud-line rt-overflow-hidden"><img src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683edea0890b9aeb59501f17_line%20.svg" loading="lazy" width="345" height="148" alt="line bar" class="rt-max-width-none rt-tab-image-full-width"/></div></div><div class="rt-image-wrap-two"><img width="466" height="217" alt="card" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/68550cc267816952c0600f78_card%202.svg" loading="lazy" class="rt-auto-fit border rt-radius-20 rt-tab-image-full-width"/></div></div></div></div></div></section><section class="rt-work"><div class="w-layout-blockcontainer rt-container w-container"><div class="w-layout-hflex rt-work-main"><div data-w-id="b7fb1ccd-310c-c292-6141-30af1e7a42f6" style="opacity:0" class="rt-work-left-part rt-radius-20"><img width="520" height="600" alt="image" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/685d188e21755e06351b7bb2_work%20card%20img%20.avif" loading="lazy" class="rt-auto-fit rt-tab-image-full-width rt-radius-20"/></div><div class="rt-work-right-part rt-mobile-text-center"><div data-w-id="b7fb1ccd-310c-c292-6141-30af1e7a42f9" style="opacity:0" class="w-layout-hflex rt-sub-text-bottom-gap"><div data-w-id="d126363a-8414-9aab-87c1-bda33ca993d6" class="w-layout-hflex rt-sub-text-main rt-position-relative"><div class="w-layout-hflex rt-sub-text-wrapper"><div class="rt-dot"></div><div class="rt-sub-text">How we work</div></div><div class="rt-button-linear color-change"></div></div></div><div class="rt-heading-bottom-gap"><h2 data-w-id="b7fb1ccd-310c-c292-6141-30af1e7a4301" class="rt-no-margin rt-text-gradiant">Personalized SaaS Solutions for Your <span class="rt-italic-text">Unique</span> Challenges</h2></div><div data-w-id="b7fb1ccd-310c-c292-6141-30af1e7a4306" style="opacity:0" class="rt-work-para-wrap"><p data-w-id="b7fb1ccd-310c-c292-6141-30af1e7a4307" class="rt-no-margin">Elit nec suspendisse placerat volutpat quisque vestibulum fusce sem at. Diam sagittis vitae tristique in in diam in. Proin est posuere neque a non pretium<br/></p></div><div class="w-layout-vflex rt-wrok-card-main"><div data-w-id="b7fb1ccd-310c-c292-6141-30af1e7a430b" class="w-layout-vflex rt-work-card rt-radius-20"><div class="w-layout-hflex rt-work-card-icon rt-radius-10"><img width="41" height="49" alt="icon" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683d20fbeed2e3df71acba46_work-icon-one.svg" loading="lazy"/></div><div class="w-layout-vflex rt-work-card-text-wrap"><div class="rt-text-style-h5">Streamlined Project Execution</div><p>Neque eget sed tempus ut in nunc magna cursus venenatis. Viverra arcu ullamcorper sagittis vel lorem. Ullamcorper purus faucibus nam.</p></div><div class="w-layout-hflex rt-work-card-button-wrap"><div class="rt-work-card-button"><div>Strategic Planning<br/></div></div><div class="rt-work-card-button"><div>Iterate<br/></div></div></div></div><div class="w-layout-vflex rt-work-card rt-radius-20"><div class="w-layout-hflex rt-work-card-icon rt-radius-10"><img width="41" height="49" alt="icon" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683d20fbeed2e3df71acba47_work-icon-two.svg" loading="lazy"/></div><div class="w-layout-vflex rt-work-card-text-wrap"><div class="rt-text-style-h5">Transparent Workflow System</div><p>Neque eget sed tempus ut in nunc magna cursus venenatis. Viverra arcu ullamcorper sagittis vel lorem. Ullamcorper purus faucibus nam.</p></div><div class="w-layout-hflex rt-work-card-button-wrap"><div class="rt-work-card-button"><div>Rapid Iteration<br/></div></div><div class="rt-work-card-button"><div>Data Insights<br/></div></div></div></div></div></div></div></div></section><section data-w-id="a684b9e5-3498-02ee-1ba9-5f034cb02ce0" class="rt-integration"><div class="w-layout-blockcontainer rt-large-container w-container"><div class="w-layout-hflex rt-integration-heading"><div class="w-layout-vflex rt-integration-heading-wrap rt-overflow-hidden"><div data-w-id="a684b9e5-3498-02ee-1ba9-5f034cb02ce4" style="opacity:0" class="w-layout-hflex rt-sub-text-bottom-gap"><div data-w-id="d126363a-8414-9aab-87c1-bda33ca993d6" class="w-layout-hflex rt-sub-text-main rt-position-relative"><div class="w-layout-hflex rt-sub-text-wrapper"><div class="rt-dot"></div><div class="rt-sub-text">integration</div></div><div class="rt-button-linear color-change"></div></div></div><div data-w-id="9105c339-e87a-9690-cab4-5ac0bcd1e998" style="opacity:0" class="rt-heading-bottom-gap"><h2 class="rt-no-margin rt-text-gradiant rt-desktop-text-center">Instant <span class="rt-italic-text">Connectivity</span> Across Multiple Software Solutions</h2></div><p data-w-id="a684b9e5-3498-02ee-1ba9-5f034cb02cf1" style="opacity:0" class="rt-desktop-text-center">Eu tristique mattis amet elementum ut gravida lacinia tortor. Massa ut amet sem et morbi vitae ante. Quisque neque id metus </p></div></div><div class="rt-integration-mobile-part"><div class="w-layout-vflex rt-integration-sticky-part"><div class="w-layout-hflex rt-integrtion-main"><div class="rt-integration-mobile"><div class="rt-mobile-frame rt-position-relative"><img width="374" height="759" alt="phone bordar" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/685506e4f58bf6c8ba6f7768_phone%20fram.webp" loading="lazy" class="rt-auto-fit rt-desktop-full-width"/></div><div class="rt-mobile-screen _1"><img width="357" height="734" alt="phone screen" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/68550853515d0a5227296b13_Screen%201.webp" loading="lazy" srcset="https://cdn.prod.website-files.com/68036da11dd5436718772525/68550853515d0a5227296b13_Screen%201-p-500.webp 500w, https://cdn.prod.website-files.com/68036da11dd5436718772525/68550853515d0a5227296b13_Screen%201.webp 714w" sizes="(max-width: 479px) 100vw, 357px" class="rt-auto-fit rt-desktop-full-width"/></div><div class="rt-mobile-screen _2"><img width="341" height="734" alt="phone screen" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/68550853deb19c98a10549fd_screen%202.webp" loading="lazy" srcset="https://cdn.prod.website-files.com/68036da11dd5436718772525/68550853deb19c98a10549fd_screen%202-p-500.webp 500w, https://cdn.prod.website-files.com/68036da11dd5436718772525/68550853deb19c98a10549fd_screen%202.webp 714w" sizes="(max-width: 479px) 100vw, 341px" class="rt-auto-fit rt-desktop-full-width"/></div></div><div class="rt-integration-image-main"><img width="1518" height="725" alt="phone side logo " src="https://cdn.prod.website-files.com/68036da11dd5436718772525/68550d47c2f8fbee5c0e15d2_ph%20bg%20line.svg" loading="lazy" class="rt-auto-fit rt-desktop-full-width"/></div></div><div class="rt-integration-mobile-overlay"></div><div class="rt-mobile-background-circle"><div class="rt-mobile-bg-circle-1"><img src="https://cdn.prod.website-files.com/68036da11dd5436718772525/6846c2b78c5cfb0a2f123113_mobile%20bg%20circle%201.svg" loading="lazy" width="783" height="783" alt="mobile circle" class="rt-desktop-full-width"/></div><div class="rt-mobile-bg-circle-2"><img src="https://cdn.prod.website-files.com/68036da11dd5436718772525/6846c2b770c5ea6d8781178b_mobile%20bg%20circle%202.svg" loading="lazy" width="1059" height="1059" alt="mobile circle" class="rt-desktop-full-width"/></div></div></div></div></div></section><section class="rt-result-v1"><div class="w-layout-blockcontainer rt-container w-container"><div class="w-layout-hflex rt-result-v1-main"><div class="rt-result-v1-left-part rt-mobile-text-center rt-overflow-hidden"><div data-w-id="ad0c6e37-8b2c-cfbf-610e-4e08dfd8a4c9" style="opacity:0" class="w-layout-hflex rt-sub-text-bottom-gap"><div data-w-id="d126363a-8414-9aab-87c1-bda33ca993d6" class="w-layout-hflex rt-sub-text-main rt-position-relative"><div class="w-layout-hflex rt-sub-text-wrapper"><div class="rt-dot"></div><div class="rt-sub-text">Result</div></div><div class="rt-button-linear color-change"></div></div></div><div class="rt-heading-bottom-gap"><h2 data-w-id="ad0c6e37-8b2c-cfbf-610e-4e08dfd8a4d1" style="opacity:0" class="rt-no-margin rt-text-gradiant">Effortless setup with no technical <span class="rt-italic-text">expertise</span> required</h2></div><div data-w-id="ad0c6e37-8b2c-cfbf-610e-4e08dfd8a4d3" style="opacity:0" class="rt-result-para-wrap"><p class="rt-no-margin">Elit nec suspendisse placerat volutpat quisque vestibulum fusce sem at. Diam sagittis vitae tristique in in diam in. Proin est posuere neque a non pretium<br/></p></div><div data-w-id="ad0c6e37-8b2c-cfbf-610e-4e08dfd8a4d7" style="opacity:0" class="w-layout-vflex rt-result-icon-wrap"><div class="w-layout-hflex rt-result-icon-box"><img width="32" height="29" alt="icon" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7edba958d1509c4e19c_icon.svg" loading="lazy"/><div class="rt-text-style-h6">Boost Your Efficiency<br/></div></div><div class="w-layout-hflex rt-result-icon-box"><img width="32" height="29" alt="icon" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683bc7edba958d1509c4e19c_icon.svg" loading="lazy"/><div class="rt-text-style-h6">Maximize Team Productivity<br/></div></div></div><div data-w-id="ad0c6e37-8b2c-cfbf-610e-4e08dfd8a4e2" style="opacity:0" class="w-layout-hflex rt-result-button-wrap"><a data-wf--rt-button--variant="color-blue" data-w-id="9f977b01-2ceb-c0cc-499c-caa5446b189c" href="/contact-one" class="rt-button w-inline-block"><div class="w-layout-hflex rt-button-content w-variant-3e2f90de-5e76-c3ff-e2ef-f9fca0fabd74 rt-overflow-hidden"><div class="rt-button-text-box rt-position-relative"><div class="rt-button-text">get in touch</div></div><div class="rt-button-circle w-variant-3e2f90de-5e76-c3ff-e2ef-f9fca0fabd74"></div></div><div class="rt-button-linear"></div></a><div class="w-layout-hflex rt-customer-wrap"><img width="41" height="42" alt="small image" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683d2167eed2e3df71acefb1_customer-Image.png" loading="lazy"/><div class="rt-customer-text">Join 15,725+ other loving customers<br/></div></div></div></div><div class="w-layout-vflex rt-result-v1-right-part rt-position-relative"><div data-w-id="ad0c6e37-8b2c-cfbf-610e-4e08dfd8a4ec" style="opacity:0" class="rt-result-card rt-radius-20"><div class="w-layout-hflex rt-square-box-wrap"><img width="171" height="188" alt="box icon" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683d2167eed2e3df71acefb0_Group%201009004719%20(1).svg" loading="lazy" data-w-id="ad0c6e37-8b2c-cfbf-610e-4e08dfd8a4ee" style="opacity:0"/></div><div class="w-layout-vflex rt-result-box-heading-wrap rt-mobile-text-center"><div class="rt-text-style-h1">2.780,00</div><p>Massa porta tellus porta elit sit tellus porttitor pellentesque massa. In amet quis ornare nisl sit. Enim odio ut bibendum lobortis sem in morbi hendrerit in. Aliquam mattis sociis malesuada</p></div></div><div data-w-id="ad0c6e37-8b2c-cfbf-610e-4e08dfd8a4f4" style="opacity:0" class="rt-result-card-two rt-tab-display-none"><img width="530" height="331" alt="card" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/68550cc2f175378c6b5c60d7_card%201.svg" loading="lazy" class="rt-auto-fit rt-desktop-full-width"/></div></div></div></div></section><section class="rt-service-v1 rt-position-relative"><div class="w-layout-hflex rt-top-style-part"><img width="1920" height="211" alt="svg " src="https://cdn.prod.website-files.com/68036da11dd5436718772525/6846cf6132513cd8ae4a4dae_top%20style.svg" loading="lazy" class="rt-auto-fit"/></div><div class="w-layout-blockcontainer rt-container w-container"><div class="rt-service-v1-main"><div class="w-layout-hflex rt-service-v1-heading-wrap rt-position-relative"><div class="w-layout-vflex rt-service-v1-heading-main"><div data-w-id="c6c6ff6e-3e60-180c-4596-60c9094d1795" style="opacity:0" class="w-layout-hflex rt-sub-text-bottom-gap rt-center"><div data-w-id="d126363a-8414-9aab-87c1-bda33ca993d6" class="w-layout-hflex rt-sub-text-main rt-position-relative"><div class="w-layout-hflex rt-sub-text-wrapper"><div class="rt-dot"></div><div class="rt-sub-text">Our Service</div></div><div class="rt-button-linear color-change"></div></div></div><div data-w-id="6145ad91-f7f4-2f0d-d31c-2e2a7a88f6f2" style="opacity:0" class="rt-heading-bottom-gap"><h2 class="rt-no-margin rt-text-gradiant rt-desktop-text-center">Regular <span class="rt-italic-text">Software</span> Updates For Continuous Improvement</h2></div><div data-w-id="66de0e75-5e54-86cf-6fba-b10794d7a0c9" style="opacity:0" class="rt-service-v1-heading-para"><p class="rt-desktop-text-center">Feugiat cursus non laoreet fringilla urna dictum pharetra nunc viverra Non duis tellus<br/></p></div></div></div></div><div class="w-layout-grid rt-service-v1-card-main"><div class="rt-service-v1-card"><div data-w-id="c6c6ff6e-3e60-180c-4596-60c9094d17a3" style="opacity:0" class="rt-service-card-image-wrap one rt-overflow-hidden"><img src="https://cdn.prod.website-files.com/68036da11dd5436718772525/6846da76214ed46f96ada4ae_card%20img%202.webp" loading="lazy" width="169.5" height="181" alt="dashbord image" class="rt-auto-fit rt-height-auto rt-landscape-image-full-width"/><div style="-webkit-transform:translate3d(50%, -50%, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(-36deg) skew(0, 0);-moz-transform:translate3d(50%, -50%, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(-36deg) skew(0, 0);-ms-transform:translate3d(50%, -50%, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(-36deg) skew(0, 0);transform:translate3d(50%, -50%, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(-36deg) skew(0, 0);opacity:0" class="rt-service-card-rotate"><img src="https://cdn.prod.website-files.com/68036da11dd5436718772525/6846da767396f7b9bed183de_card%20img%201.webp" loading="lazy" width="327" height="253" alt="dashbord image" srcset="https://cdn.prod.website-files.com/68036da11dd5436718772525/6846da767396f7b9bed183de_card%20img%201-p-500.webp 500w, https://cdn.prod.website-files.com/68036da11dd5436718772525/6846da767396f7b9bed183de_card%20img%201.webp 654w" sizes="(max-width: 479px) 100vw, 327px" class="rt-auto-fit"/></div></div><div class="w-layout-vflex rt-service-card-contant-wrap"><div class="rt-text-style-h5">Optimize User Experience</div><p class="rt-desktop-text-center">Lorem ipsum dolor sit amet consectetur adipiscing elit Mauris suscipit lobortis</p></div></div><div class="rt-service-v1-card"><div data-w-id="c6c6ff6e-3e60-180c-4596-60c9094d17ab" style="opacity:0" class="rt-service-card-image-wrap two rt-overflow-hidden"><div style="-webkit-transform:translate3d(-70%, 70%, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(38deg) skew(0, 0);-moz-transform:translate3d(-70%, 70%, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(38deg) skew(0, 0);-ms-transform:translate3d(-70%, 70%, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(38deg) skew(0, 0);transform:translate3d(-70%, 70%, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(38deg) skew(0, 0);opacity:1" class="rt-service-card-rotate-3"><img class="rt-auto-fit" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/6847cecfbcdcd55e2014d9ee_card%20img%205.webp" width="275.5" height="213" alt="card image" style="-webkit-transform:translate3d(0, 80%, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);-moz-transform:translate3d(0, 80%, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);-ms-transform:translate3d(0, 80%, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);transform:translate3d(0, 80%, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(0) skew(0, 0);opacity:0" sizes="(max-width: 479px) 100vw, 275.5px" loading="lazy" srcset="https://cdn.prod.website-files.com/68036da11dd5436718772525/6847cecfbcdcd55e2014d9ee_card%20img%205-p-500.webp 500w, https://cdn.prod.website-files.com/68036da11dd5436718772525/6847cecfbcdcd55e2014d9ee_card%20img%205.webp 551w"/></div><img src="https://cdn.prod.website-files.com/68036da11dd5436718772525/6847c4b325cc6570dd8fe5ad_card%20img%206.svg" loading="lazy" width="245" height="294" alt="progess bar card" class="rt-auto-fit rt-landscape-image-full-width"/></div><div class="w-layout-vflex rt-service-card-contant-wrap"><div class="rt-text-style-h5">Maximize Revenue Potential</div><p class="rt-desktop-text-center">Facilisis fringilla risus dapibus ultricies quis Semper sapien fermentum a augue</p></div></div><div class="rt-service-v1-card"><div data-w-id="c6c6ff6e-3e60-180c-4596-60c9094d17b3" style="opacity:0" class="rt-service-card-image-wrap one rt-overflow-hidden"><img src="https://cdn.prod.website-files.com/68036da11dd5436718772525/6847ac82aa69e7bc3bb64172_card%20img%203.svg" loading="lazy" width="269" height="189" alt="card image" class="rt-service-card-img rt-position-relative rt-landscape-image-full-width"/><div style="-webkit-transform:translate3d(50%, -50%, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(-24deg) skew(0, 0);-moz-transform:translate3d(50%, -50%, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(-24deg) skew(0, 0);-ms-transform:translate3d(50%, -50%, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(-24deg) skew(0, 0);transform:translate3d(50%, -50%, 0) scale3d(1, 1, 1) rotateX(0) rotateY(0) rotateZ(-24deg) skew(0, 0);opacity:0" class="rt-service-card-rotate-2"><img src="https://cdn.prod.website-files.com/68036da11dd5436718772525/6847ac82cbb2fc4939761611_card%20img%204.svg" loading="lazy" width="333" height="257" alt="progess bar card" class="rt-auto-fit"/></div></div><div class="w-layout-vflex rt-service-card-contant-wrap"><div class="rt-text-style-h5">Reliable Online Assistance</div><p class="rt-desktop-text-center">Proin purus eu bibendum nunc cras aliquet sodales dictum augue Facilisis</p></div></div></div></div></section><section class="rt-faq-v1"><div class="w-layout-blockcontainer rt-container w-container"><div class="w-layout-vflex rt-faq-main rt-overflow-hidden"><div data-w-id="b00a62f5-f1d4-1e54-2b98-60ffcffff1fa" style="opacity:0" class="w-layout-hflex rt-sub-text-bottom-gap rt-center"><div data-w-id="d126363a-8414-9aab-87c1-bda33ca993d6" class="w-layout-hflex rt-sub-text-main rt-position-relative"><div class="w-layout-hflex rt-sub-text-wrapper"><div class="rt-dot"></div><div class="rt-sub-text">Frequeatly Asked Questions</div></div><div class="rt-button-linear color-change"></div></div></div><div data-w-id="b00a62f5-f1d4-1e54-2b98-60ffcffff201" style="opacity:0" class="rt-faq-heading-wrap"><h2 class="rt-no-margin rt-text-gradiant rt-desktop-text-center">You Have <span class="rt-italic-text">Questions</span> We have Answer</h2></div><div class="rt-faq-wrap rt-overflow-hidden"><div data-w-id="dca369cc-2d68-d050-ea35-eac96200cef7" class="rt-fq-dropdown"><div class="w-layout-hflex rt-faq-toggle"><div class="rt-faq-pricing-one-question-top"><div class="rt-text-style-h5">How do I download, install, and activate the software successfully?</div></div><div class="rt-faq-icon"><div class="rt-minus"></div><div class="rt-plus"></div></div></div><div class="rt-overflow-hidden"><div class="rt-question-details"><p class="rt-no-margin">Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nam faucibus ante non auctor volutpat Sed vitae lorem non sapien feugiat interdum ac nec eros. Nullam est nibh, imperdiet quis nisi at, congue eleifend orci.</p></div></div></div><div data-w-id="dca369cc-2d68-d050-ea35-eac96200cf03" class="rt-fq-dropdown"><div class="w-layout-hflex rt-faq-toggle"><div class="rt-faq-pricing-one-question-top"><div class="rt-text-style-h5">What are the minimum system requirements for running this software?</div></div><div class="rt-faq-icon"><div class="rt-minus"></div><div class="rt-plus"></div></div></div><div class="rt-overflow-hidden"><div class="rt-question-details"><p class="rt-no-margin">Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nam faucibus ante non auctor volutpat Sed vitae lorem non sapien feugiat interdum ac nec eros. Nullam est nibh, imperdiet quis nisi at, congue eleifend orci.</p></div></div></div><div data-w-id="dca369cc-2d68-d050-ea35-eac96200cf0f" class="rt-fq-dropdown"><div class="w-layout-hflex rt-faq-toggle"><div class="rt-faq-pricing-one-question-top"><div class="rt-text-style-h5">What steps should I follow to update the software to the latest version?</div></div><div class="rt-faq-icon"><div class="rt-minus"></div><div class="rt-plus"></div></div></div><div class="rt-overflow-hidden"><div class="rt-question-details"><p class="rt-no-margin">Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nam faucibus ante non auctor volutpat Sed vitae lorem non sapien feugiat interdum ac nec eros. Nullam est nibh, imperdiet quis nisi at, congue eleifend orci.</p></div></div></div><div data-w-id="dca369cc-2d68-d050-ea35-eac96200cf1b" class="rt-fq-dropdown"><div class="w-layout-hflex rt-faq-toggle"><div class="rt-faq-pricing-one-question-top"><div class="rt-text-style-h5">How do I troubleshoot installation or compatibility issues with my system?</div></div><div class="rt-faq-icon"><div class="rt-minus"></div><div class="rt-plus"></div></div></div><div class="rt-overflow-hidden"><div class="rt-question-details"><p class="rt-no-margin">Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nam faucibus ante non auctor volutpat Sed vitae lorem non sapien feugiat interdum ac nec eros. Nullam est nibh, imperdiet quis nisi at, congue eleifend orci.</p></div></div></div><div data-w-id="dca369cc-2d68-d050-ea35-eac96200cf27" class="rt-fq-dropdown"><div class="w-layout-hflex rt-faq-toggle"><div class="rt-faq-pricing-one-question-top"><div class="rt-text-style-h5">What subscription plans are available, and how can I purchase one?</div></div><div class="rt-faq-icon"><div class="rt-minus"></div><div class="rt-plus"></div></div></div><div class="rt-overflow-hidden"><div class="rt-question-details"><p class="rt-no-margin">Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nam faucibus ante non auctor volutpat Sed vitae lorem non sapien feugiat interdum ac nec eros. Nullam est nibh, imperdiet quis nisi at, congue eleifend orci.</p></div></div></div></div></div></div></section><section class="rt-footer rt-position-relative"><div class="rt-large-container rt-position-relative"><div class="w-layout-hflex rt-footer-top-wrapper"><div id="w-node-_3d0008a9-c775-29ca-2610-f2534409d4e8-4409d4e5" class="rt-footer-left-wrap"><div class="w-layout-vflex rt-footer-left-text"><div class="w-layout-vflex rt-footer-heading-wrap"><div class="rt-text-style-h3 rt-text-gradiant footer-heading">Stay <em class="rt-italic-text">Connected </em>With Us Across All Platforms</div><p>Consequat pellentesque eros tempus quam amet fames odio in erat. Aliquet rhoncus nullam aenean mattis. Adipiscing<br/></p></div><a data-wf--rt-button--variant="color-blue" data-w-id="9f977b01-2ceb-c0cc-499c-caa5446b189c" href="/contact-one" class="rt-button w-inline-block"><div class="w-layout-hflex rt-button-content w-variant-3e2f90de-5e76-c3ff-e2ef-f9fca0fabd74 rt-overflow-hidden"><div class="rt-button-text-box rt-position-relative"><div class="rt-button-text">get in touch</div></div><div class="rt-button-circle w-variant-3e2f90de-5e76-c3ff-e2ef-f9fca0fabd74"></div></div><div class="rt-button-linear"></div></a></div></div><div class="w-layout-vflex rt-footer-col add-border"><div class="w-layout-vflex rt-footer-link-wrap"><div class="rt-footer-tem-heading"><div class="rt-text-style-h5">Quick Links<br/></div></div><div class="w-layout-vflex rt-footer-link-item"><a href="/home-one" aria-current="page" class="rt-footer-link w--current">Home</a><a href="/about" class="rt-footer-link">About</a><a href="/features" class="rt-footer-link">features</a><a href="/pricing-one" class="rt-footer-link">Pricing</a></div></div></div><div class="w-layout-vflex rt-footer-col"><div class="w-layout-vflex rt-footer-link-wrap"><div class="rt-footer-tem-heading"><div class="rt-text-style-h5">Contact Info<br/></div></div><div class="w-layout-vflex rt-footer-link-item"><a href="tel:8884567890" class="rt-footer-link">(888) 456 7890</a><div>410 Sandtown, California 94001,USA</div><a href="mailto:info@example.com" class="rt-footer-link">info@example.com</a></div></div></div></div></div><div class="rt-footer-bottom-wrap"><div class="w-layout-blockcontainer rt-large-container w-container"><div class="w-layout-hflex rt-footer-end-part"><div class="w-layout-hflex rt-social-link-main"><div class="w-layout-hflex rt-social-link-wrap"><a href="https://www.instagram.com/" class="rt-footer-link">Instagram</a><img width="15" height="13" alt="" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683592b418e5a68e034a7ca7_Arrow.svg" loading="lazy"/></div><div class="w-layout-hflex rt-social-link-wrap"><a href="https://www.facebook.com" class="rt-footer-link">Facebook</a><img width="13" height="13" alt="" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683592b418e5a68e034a7ca7_Arrow.svg" loading="lazy"/></div><div class="w-layout-hflex rt-social-link-wrap"><a href="https://dribbble.com/" class="rt-footer-link">dribbble</a><img width="13" height="13" alt="" src="https://cdn.prod.website-files.com/68036da11dd5436718772525/683592b418e5a68e034a7ca7_Arrow.svg" loading="lazy"/></div></div><div class="rt-footer-designe-wrap"><div class="rt-mobile-text-center">Design &amp; Developed by - Fastcreek AI</div></div></div></div></div></section><script src="https://d3e54v103j8qbb.cloudfront.net/js/jquery-3.5.1.min.dc5e7f18c8.js?site=68036da11dd5436718772525" type="text/javascript" integrity="sha256-9/aliU8dGd2tb6OSsuzixeV4y/faTqgFtohetphbbj0=" crossorigin="anonymous"></script><script src="https://cdn.prod.website-files.com/68036da11dd5436718772525/js/webflow.schunk.36b8fb49256177c8.js" type="text/javascript" integrity="sha384-4abIlA5/v7XaW1HMXKBgnUuhnjBYJ/Z9C1OSg4OhmVw9O3QeHJ/qJqFBERCDPv7G" crossorigin="anonymous"></script><script src="https://cdn.prod.website-files.com/68036da11dd5436718772525/js/webflow.schunk.6b42e12f681f0615.js" type="text/javascript" integrity="sha384-ogPPRt24gXJwZf/jogHw9aIOsHqUJSfc4Go4nMjJzbYhc/Yq71zKfQV6zbzJZZnh" crossorigin="anonymous"></script><script src="https://cdn.prod.website-files.com/68036da11dd5436718772525/js/webflow.0d985c84.68f15673198f19d7.js" type="text/javascript" integrity="sha384-16SN6FYqKRmVocJbpKW2B7SJR4dQdhuh0I4Jzafu5OBfVSWgASGTcQiQGPz+HCuJ" crossorigin="anonymous"></script>
<script src="https://cdn.jsdelivr.net/npm/lenis@1.1.18/dist/lenis.min.js"></script>
<script>
(function () {
  var reduced = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
  if (!window.Lenis || reduced) return;

  var lenis = new Lenis({
    duration: 1.6,
    easing: function (t) { return Math.min(1, 1.001 - Math.pow(2, -10 * t)); },
    wheelMultiplier: 0.9,
    smoothWheel: true,
    syncTouch: false
  });
  window.__lenis = lenis;

  function raf(time) {
    lenis.raf(time);
    requestAnimationFrame(raf);
  }
  requestAnimationFrame(raf);

  // Route in-page anchor clicks through Lenis so jumps ease instead of snapping.
  document.querySelectorAll('a[href^="#"]').forEach(function (link) {
    link.addEventListener('click', function (e) {
      var target = document.querySelector(link.getAttribute('href'));
      if (!target) return;
      e.preventDefault();
      lenis.scrollTo(target, { offset: 0 });
    });
  });
})();
</script>
<script>
(function () {
  var header = document.getElementById('fcHeader');
  var toggle = document.getElementById('fcMenuToggle');
  if (!header || !toggle) return;

  function onScroll() {
    header.classList.toggle('is-scrolled', window.scrollY > 10);
  }
  onScroll();
  window.addEventListener('scroll', onScroll, { passive: true });

  function setOpen(open) {
    header.classList.toggle('is-open', open);
    toggle.setAttribute('aria-expanded', String(open));
    // Lenis drives the scroll, so overflow:hidden alone won't hold the page still.
    document.body.style.overflow = open ? 'hidden' : '';
    if (window.__lenis) { open ? window.__lenis.stop() : window.__lenis.start(); }
  }

  toggle.addEventListener('click', function () {
    setOpen(!header.classList.contains('is-open'));
  });

  document.querySelectorAll('#fcMobileMenu a').forEach(function (a) {
    a.addEventListener('click', function () { setOpen(false); });
  });

  document.addEventListener('keydown', function (e) {
    if (e.key === 'Escape') setOpen(false);
  });

  window.addEventListener('resize', function () {
    if (window.innerWidth >= 768) setOpen(false);
  });
})();
</script>
</body></html>
```

---

## Appendix B — `.gitignore`

```
.DS_Store
```
