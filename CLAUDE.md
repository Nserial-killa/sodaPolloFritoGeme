# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

This repo is a single-page marketing/ordering website for "Pollo Frito GEME", a Costa Rican fried-chicken soda (fast-food spot). The entire site is one self-contained file: [pollo-frito-geme.html](pollo-frito-geme.html). There is no build system, package manager, or test suite — it's meant to be opened directly in a browser or dropped onto static hosting as-is.

[ideaPollosFritoGeme.txt](ideaPollosFritoGeme.txt) is the original client brief (in Spanish) describing the desired UX for ordering: product selection → cart → slide-out order drawer/form → send order as a prefilled WhatsApp message. Treat it as the source of product requirements when a feature's intent is unclear.

## Working with this codebase

- No build/lint/test commands exist. To preview changes, just open `pollo-frito-geme.html` in a browser (or serve the directory with any static file server).
- The file mixes structure, style, and behavior in one document:
  - `<head>`: loads Tailwind CSS from a CDN (`cdn.tailwindcss.com`) and Lucide icons from jsDelivr, plus Google Fonts (Alfa Slab One for headlines, Work Sans for body text). A `<style>` block defines CSS custom properties (`--ink`, `--panel`, `--wine`, `--red`, `--gold`, `--cream`, etc.) for the brand's dark/red/gold theme, plus a few custom component classes (`.card`, `.field`, `.btn-red`, `.btn-primary`, `.btn-outline`, `.headline`, `.drawer-*`, `.hero-rays`).
  - `<body>`: static markup for the header/nav, `#inicio` (hero), a quick-categories strip, `#menu` (products, filled in by JS), `#informacion` (location/hours/phone/maps placeholder), `#opiniones` (review form + list), and a full footer with social links.
  - Two `<template>` elements (`#product-template`, `#cart-row-template`) are cloned by JS to render product cards and cart line items — edit these templates (not JS string concatenation) when changing product card or cart row markup.
  - A single `<script>` block near the end of the file contains all application logic, in roughly this order: product data → cart state → drawer open/close → product rendering → cart rendering → order form validation → WhatsApp message builder → menu search/filter → localStorage-backed reviews → app init calls at the bottom.
  - Logo and hero images are inlined directly as `data:image/jpeg;base64,...` URIs inside `<img src="...">` — these are very long single lines. When editing near them, use targeted greps/offsets rather than reading the whole file, since a couple of lines are 20–80KB long and can blow past read limits.

## Architecture / data flow

- **Product catalog**: a plain array `PRODUCTS` (id, name, desc, category, price, and optional `comboPrice`/`comboLabel` for "add fries" style upsells). `CATEGORY_LABEL` maps category keys to display names. To add/edit menu items, edit this array — the DOM is generated entirely from it via `renderProducts()`.
- **Cart**: a `Map` keyed by product id (or `id + "-combo"` for the combo variant), holding `{key, name, variant, unit, quantity}`. All cart mutations flow through `addCart`/`changeQuantity`/`updateCart`, which re-render `#checkout-items` from the `cart-row-template` and update all total/badge displays (header badge, mobile bar, drawer total).
- **Order drawer**: a slide-out panel (`#order-drawer`) opened by any `.open-drawer` trigger (cart button, "Armar mi pedido", footer link, etc.) via `openDrawer()`/`closeDrawer()`. It assigns a random order code (`ensureOrder()`), and contains the checkout form (customer name/phone, order type, conditional address field for "Express", payment method with conditional "Otro" text field, instructions textarea).
- **Checkout validation**: `validateOrder()` does manual required-field checks and toggles `.invalid`/error text per field; there is no external validation library.
- **WhatsApp handoff**: `buildWhatsappMessage()` composes a plain-text order summary and `initOrderForm()`'s submit handler opens `https://wa.me/<WHATSAPP_NUMBER>?text=<encoded message>` in a new tab — this is the entire "backend": there is no server, database, or order persistence beyond the browser session. `WHATSAPP_NUMBER` is a top-level constant to update if the business number changes.
- **Reviews**: `#opiniones` section stores star rating + comment in `localStorage` under `geme_reviews_v1` (`loadReviewsLocal`/`saveReviewsLocal`) — reviews are per-device only, not shared across visitors. This is explicitly called out in the UI copy as a demo/placeholder pending a real backend.
- **Menu search/filter**: `initMenuFilters()` filters rendered `.product-card` elements by category button and/or free-text search against a precomputed `data-searchtext` attribute (no re-render, just show/hide via `.hidden`).

## Content still marked as placeholder

Several spots in the markup are explicitly left as fill-in-later placeholders (in Spanish, e.g. "Completá esta sección con la dirección exacta del local", "Ajustá este horario al real del local", social links pointing to `href="#"`, and an inline SVG illustration standing in for a real storefront photo). When asked to "finish" or "polish" the site, these are the known gaps to check first.
