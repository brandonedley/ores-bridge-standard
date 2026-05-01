# Verification Guide

Pair this with [README.md](README.md). The README covers install. This guide covers what to check after.

---

## Pre-launch checklist

Before you flip this on for real traffic:

- [ ] **GTM tag is published, not just saved.** GTM → Versions → newest version shows the bridge tag.
- [ ] **GTM tag is in the widget container** (`book.mylimobiz.com`), not your site's container.
- [ ] **Receiver script is on every page that embeds the widget.** Search your site for `data-la-bridge`.
- [ ] **GA4 Measurement ID matches** the property where you want conversions to land.
- [ ] **Iframe `src` points at your company slug** (`book.mylimobiz.com/v4/YOUR-COMPANY-ID`).
- [ ] **Iframe has `width:100%`** and `min-height` (not fixed `height`) and `scrolling="no"`.

---

## Smoke test (5 minutes)

In a fresh incognito window with DevTools open:

1. **Receiver loaded?**
   ```js
   window.dataLayer.find(e => e.event === 'la_bridge_ready')
   ```
   Returns an object with `la_bridge_mode` set to `gtag` or `dataLayer`.

2. **Sender posting?** Click the widget's Service Type dropdown. In console:
   ```js
   addEventListener('message', e => console.log(e.origin, e.data?.type))
   ```
   Then click another field. You should see `LA_DATALAYER_EVENT` (and `LA_IFRAME_HEIGHT`) from `https://book.mylimobiz.com`.

3. **Iframe resizing?** Note its current pixel height in the Elements panel. Fill in pickup/dropoff. The height should grow as the widget renders results.

4. **Events reaching GA4?** GA4 → Admin → DebugView. Complete a test booking. Watch for `purchase` (or `generate_lead` for a quote).

5. **Google Ads conversion firing?** Run the test booking with a `?gclid=test123` on the parent URL. In Google Ads → Conversions, the test conversion should appear within ~3 hours.

---

## What "broken" looks like

| Symptom | Most likely cause |
|---|---|
| `la_bridge_ready` missing from `dataLayer` | Receiver script never ran. Check the `<script>` actually loaded (Network tab) and has `data-la-bridge`. |
| `la_bridge_ready` present, no events flow | GTM sender tag isn't published, or it's in the wrong container. Verify in GTM → Versions. |
| Events show in GA4 but not in Google Ads | gtag mode bypassing GTM-based conversions. Switch to `data-mode="dataLayer"`. |
| Iframe stays small | Outdated `parent-receiver.js`. The latest version picks the visible iframe on Wix. |
| Events fire twice | Two bridge installations on the same page, or two iframes both running the sender. |
| `widget-loader.js` console errors | Limo Anywhere's own code. Harmless. |

---

## ORES settings reminder

Two fields in Limo Anywhere → ORES & Mobile Settings affect the bridge:

- **Tag Manager Code** — enter only the part **after** `GTM-`. `GTM-WFMR9QB4` → `WFMR9QB4`. Entering the full ID breaks the integration.
- **Google Tag ID** — works whether you fill it in or leave it blank. ORES emits GA4-style events when set, legacy `transactionId` when not. The bridge handles both.

---

## Hand-off note

If a marketing team installs this without engineering:

> The bridge fires `purchase` and `generate_lead` events to GA4 from the parent domain. Conversions tagged in Google Ads against either event will track gclid attribution correctly. If conversions are configured in GTM as Custom Event triggers, set `data-mode="dataLayer"` on the receiver — otherwise GA4 receives events directly via gtag and GTM never sees them.
