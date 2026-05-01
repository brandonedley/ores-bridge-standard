# ORES Bridge

Track Limo Anywhere booking conversions and Google Ads gclid attribution from your own domain. Auto-resizes the iframe so the widget never scrolls inside itself.

Two pieces:
1. **GTM tag** inside the widget (`book.mylimobiz.com`) — sends events out via `postMessage`.
2. **Receiver script + iframe** on your page — receives events, fires GA4, resizes the iframe.

---

## Install

You need ten minutes and two values:

- **Your company ID** — the slug after `/v4/` in your booking URL (e.g., `limoforla`).
- **Your GA4 Measurement ID** — looks like `G-XXXXXXXXXX`. Skip if GTM manages GA4 on your site.

### Step 1 — Paste the GTM tag

In the GTM container that loads inside the widget (set in Limo Anywhere → ORES & Mobile Settings → Tag Manager Code; enter the part **after** `GTM-`):

1. **Tags → New → Custom HTML.**
2. Open [`gtm-sender-tag.html`](https://raw.githubusercontent.com/brandonedley/ores-bridge-standard/main/gtm-sender-tag.html), copy the entire file, paste into the HTML field.
3. **Triggering → DOM Ready → All Pages.**
4. **Advanced Settings → Tag firing priority** = `1`.
5. **Save → Submit → Publish.**

### Step 2 — Paste the script and iframe on your page

Replace `YOUR-COMPANY-ID` and `G-XXXXXXXXXX`. Paste both blocks anywhere a booking widget should appear:

```html
<!-- ORES Bridge — receiver -->
<script data-la-bridge data-measurement-id="G-XXXXXXXXXX">
!function(){"use strict";var e=document.currentScript||document.querySelector("script[data-la-bridge]");if(e){var t={measurementId:e.dataset.measurementId||null,mode:e.dataset.mode||"auto",eventPrefix:e.dataset.eventPrefix||"la_",allowedOrigins:(e.dataset.allowedOrigins||"https://book.mylimobiz.com").split(",").map(function(e){return e.trim()}),debug:"true"===e.dataset.debug,iframeSelector:e.dataset.iframeSelector||'iframe[src*="book.mylimobiz.com"]',minHeight:parseInt(e.dataset.minHeight,10)||400,maxHeight:parseInt(e.dataset.maxHeight,10)||2e3,enableHeightResize:"false"!==e.dataset.enableHeightResize};d("Config loaded:",t);var a=t.mode,i=t.measurementId,n={purchase:{name:"purchase",params:["transaction_id","value","currency","items","coupon","shipping","tax","booking_type","is_quote"]},generate_lead:{name:"generate_lead",params:["value","currency","lead_source","transaction_id","booking_type","is_quote"]}},r=null,o={};g(),window.addEventListener("message",c,!1),d("Receiver initialized. Listening for events from:",t.allowedOrigins.join(", ")),window.dataLayer=window.dataLayer||[],window.dataLayer.push({event:t.eventPrefix+"bridge_ready",la_bridge_mode:a,la_bridge_measurement_id:i||"auto"})}else console.error("[LA-Bridge] Could not find script element. Add data-la-bridge attribute.");function d(){if(t.debug){var e=["[LA-Bridge]"].concat(Array.prototype.slice.call(arguments));console.log.apply(console,e)}}function g(){"gtag"===t.mode||"dataLayer"===t.mode?a=t.mode:t.measurementId?a="gtag":"function"==typeof window.gtag?(a="gtag",i=function(){if(window.dataLayer&&Array.isArray(window.dataLayer))for(var e=0;e<window.dataLayer.length;e++){var t=window.dataLayer[e];if(t&&"config"===t[0]&&"string"==typeof t[1]&&t[1].startsWith("G-"))return t[1]}if(window.google_tag_data&&window.google_tag_data.tidr)for(var a=Object.keys(window.google_tag_data.tidr),i=0;i<a.length;i++)if(a[i].startsWith("G-"))return a[i];return null}()):a="dataLayer","gtag"===a&&"function"!=typeof window.gtag&&(window.dataLayer=window.dataLayer||[],window.gtag=function(){window.dataLayer.push(arguments)},d("Bootstrapped window.gtag (GTM-managed GA4 detected)")),d("Resolved mode:",a,"Measurement ID:",i||"(auto)")}function s(e,a){window.dataLayer=window.dataLayer||[];var i={event:t.eventPrefix+e};return Object.keys(a).forEach(function(e){"event"!==e&&(i[t.eventPrefix+e]=a[e])}),d("Pushing to dataLayer:",i),window.dataLayer.push(i),!0}function l(e,t){return d("Handling event:",e,t),t&&t._debug_only?(delete t._debug_only,s(e,t)):"gtag"===a?function(e,t){if("function"!=typeof window.gtag)return d("gtag not available, falling back to dataLayer"),s(e,t),!1;var a={},r=n[e];r?r.params.forEach(function(e){void 0!==t[e]&&(a[e]=t[e])}):delete(a=Object.assign({},t)).event,i&&(a.send_to=i),d("Firing gtag:",e,a);try{return window.gtag("event",e,a),!0}catch(a){return d("gtag error:",a),s(e,t),!1}}(e,t):s(e,t)}function u(e){if(t.enableHeightResize){var a=function(){var m=document.querySelectorAll(t.iframeSelector);if(!m.length)return null;for(var k=0;k<m.length;k++){var x=m[k];if(x&&x.offsetParent){var rc=x.getBoundingClientRect();if(rc.width>0&&rc.height>0)return x}}return m[0]}();if(a){var i=Math.max(t.minHeight,Math.min(t.maxHeight,e));a.style.height=i+"px",d("Updated iframe height:",i,"(raw:",e+")")}else d("No iframe found with selector:",t.iframeSelector)}else d("Height resize disabled, ignoring height:",e)}function c(e){if(a=e.origin,-1!==t.allowedOrigins.indexOf("*")||-1!==t.allowedOrigins.indexOf(a)){var a,i=e.data;if(i&&i.type)if("LA_IFRAME_HEIGHT"!==i.type){if("LA_DATALAYER_EVENT"===i.type){var n=i.payload;if(n&&n.eventName){if(!n.eventName.startsWith("gtm.")){var r=n.timestamp+"_"+n.eventName;if(o[r])d("Duplicate event skipped:",n.eventName);else{o[r]=!0;var g=Object.keys(o);g.length>100&&delete o[g[0]],d("Received from",e.origin+":",n.eventName),l(n.eventName,n.data||{})}}}else d("Invalid payload:",i)}}else"number"==typeof i.height&&i.height>0&&u(i.height)}}}();
</script>

<!-- ORES Bridge — booking widget -->
<iframe
  src="https://book.mylimobiz.com/v4/YOUR-COMPANY-ID"
  style="width:100%; min-height:600px; border:0; display:block;"
  scrolling="no"
  title="Book a ride"></iframe>
```

The iframe needs `width:100%` (default is ~300px), `min-height` not fixed `height` (the receiver auto-grows the height), and `scrolling="no"` (no scrollbar inside the widget once auto-resize works).

**Wix users:** Wix renders Custom HTML embeds twice. The receiver picks the visible iframe automatically. Stretch the Custom HTML block in the Wix editor — `width:100%` only fills the container Wix gives you.

### Step 3 — Verify

Open the page. In DevTools console:

```js
window.dataLayer.find(e => e.event === 'la_bridge_ready')
```

Returns an object → receiver is running. Click the widget's Service Type dropdown, then watch GA4 → Admin → DebugView. `purchase` and `generate_lead` events arrive after a test booking.

---

## Hosting the script externally (alternative to inline)

Skip this if the inline paste worked.

Self-host `parent-receiver.min.js` on your CDN, then replace the inline block above with:

```html
<script data-la-bridge data-measurement-id="G-XXXXXXXXXX"
        src="https://yourcdn.com/parent-receiver.min.js"></script>
```

Use this when you control your hosting and want one place to ship updates.

---

## Events fired

| Event | When it fires | Why |
|---|---|---|
| `purchase` | Booking completed with a price | Standard GA4 ecommerce |
| `generate_lead` | Quote submitted (no price shown) | Keeps ecommerce reports clean |

The bridge picks `generate_lead` over `purchase` whenever the widget runs in quote mode — when ORES displays "Request Quote" instead of a price. It checks the `#BookType` hidden input first, then button text. The result is cached per session.

---

## Configuration

Override defaults with `data-*` attributes on the `<script>` tag:

| Attribute | Default | Use when |
|---|---|---|
| `data-measurement-id` | auto-detect | You want to pin a specific GA4 stream |
| `data-mode` | `auto` | You're using GTM (set to `dataLayer`) — see below |
| `data-debug` | `false` | Diagnosing — logs to `[LA-Bridge]` console prefix |
| `data-allowed-origins` | `https://book.mylimobiz.com` | Self-hosted ORES on a different host |
| `data-iframe-selector` | `iframe[src*="book.mylimobiz.com"]` | Custom iframe wrapper |
| `data-min-height` / `data-max-height` | `400` / `2000` | Constrain auto-resize |
| `data-enable-height-resize` | `true` | Set `"false"` to keep your fixed height |

### gtag mode vs. dataLayer mode

The receiver picks one of two paths:

| Setup | Mode | Where events go |
|---|---|---|
| `data-measurement-id="G-..."` set | **gtag** | `gtag('event', ...)` direct to GA4 |
| No measurement ID, page already runs gtag | **gtag** | Same |
| Otherwise | **dataLayer** | `window.dataLayer.push({event:'la_purchase', ...})` for GTM |

If your Google Ads conversions live in GTM as Custom Event triggers (`la_purchase`, `la_generate_lead`), force dataLayer mode:

```html
<script data-la-bridge data-mode="dataLayer" ...></script>
```

gtag mode bypasses GTM entirely. This is the most common cause of "tracking isn't working."

---

## Troubleshooting

**`la_bridge_ready` missing from `dataLayer`** — the receiver script never ran. Check that `data-la-bridge` is on the `<script>` tag and the script content actually loaded.

**Receiver loaded, no events fire** — the GTM sender tag isn't published, or it's in the wrong container. Run this in the parent page console, then click in the widget:
```js
addEventListener('message', e => console.log(e.origin, e.data))
```
You should see `LA_DATALAYER_EVENT` arrive from `https://book.mylimobiz.com`. If not, GTM sender isn't running.

**Events fire to GA4 but Google Ads conversions never count** — you're in gtag mode but configured the conversion in GTM. Switch to `data-mode="dataLayer"` (above).

**Iframe stays at the wrong height** — Wix renders the embed twice; older versions of the receiver picked the hidden iframe. Update to the latest `parent-receiver.min.js`.

**Console errors from `widget-loader.js`: `event.data.indexOf is not a function`** — Limo Anywhere's own message handler choking on object-typed postMessages from third-party scripts. Harmless. Not from this bridge.

---

## Files

| File | Where it goes |
|---|---|
| `gtm-sender-tag.html` | Paste into the widget's GTM Custom HTML tag |
| `min/gtm-sender.min.html` | Smaller version of the above |
| `parent-receiver.js` | Readable source for the receiver |
| `min/parent-receiver.min.js` | Minified receiver — what's inlined above |

---

## How it works

The widget's GTM container loads the sender tag, which intercepts every `dataLayer.push()` inside the iframe and forwards it via `window.parent.postMessage()`. The receiver on your page listens for those messages, validates origin, and either calls `gtag('event', ...)` or pushes a prefixed event to `dataLayer`. A separate `LA_IFRAME_HEIGHT` message stream resizes the iframe to fit its content.

Because the receiver fires on **your** domain, GA4 attaches your gclid cookie and Google Ads attribution stays intact.

---

## Upgrade

**ORES Bridge — Advanced** adds full-funnel events (`form_start`, `view_item_list`, `select_item`, `add_contact_info`, `begin_checkout`, `add_payment_info`) plus Enhanced Conversions (SHA-256 hashed contact data for Google Ads). Contact your implementation team.

---

## License

Copyright © 2026 Brandon Edley. Free for use with Limo Anywhere ORES integrations. See [LICENSE](LICENSE).
