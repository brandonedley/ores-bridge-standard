# ORES Bridge -- Standard

Purchase conversion tracking and GCLID attribution for the Limo Anywhere ORES booking widget, with automatic iframe height resizing.

## Overview

When you embed the Limo Anywhere widget (`book.mylimobiz.com`) in an iframe, the iframe isolates analytics events from your page. This bridge captures those events and forwards them to your parent page via `postMessage`, enabling you to:

- Track purchase conversions in YOUR GA4 property
- Preserve GCLID attribution from Google Ads
- Fire conversion events from your domain
- Automatically resize the iframe to eliminate scrollbars

## ORES Settings Compatibility

**No changes needed to your Limo Anywhere settings.** The bridge works regardless of your ORES configuration.

### ORES & Mobile Settings Fields

In Limo Anywhere's backend under **ORES & Mobile Settings**, there are two relevant fields:

#### 1. Tag Manager Code

This field controls which GTM container loads in the widget. **The sender tag must be deployed to this container.**

**Important:** Enter ONLY the code **after** `GTM-`, not the full container ID.

| If your container is... | Enter this value |
|------------------------|------------------|
| `GTM-XXXXXXX` | `XXXXXXX` |
| `GTM-ABC123` | `ABC123` |

**Common mistake:** Entering `GTM-XXXXXXX` (with the prefix) breaks the integration. Enter only `XXXXXXX`.

#### 2. Google Tag ID

This field controls whether ORES fires GA4 events directly. It affects what purchase events ORES emits:

| ORES Google Tag ID Setting | What ORES Emits | Bridge Behavior |
|---------------------------|-----------------|-----------------|
| **Configured** (e.g., `G-ABC123`) | GA4-style `gtag("event", "purchase", {...})` | Captured and forwarded |
| **Empty / Not configured** | Legacy `dataLayer.push({transactionId, transactionTotal})` | Captured and forwarded |

### What This Means For You

- **No GTAG in ORES?** No problem. The bridge captures the legacy dataLayer event.
- **Already have GTAG in ORES?** The bridge captures the GA4 event. No double-tracking -- events are deduplicated.
- **Want to add GTAG to ORES later?** Go ahead. The bridge adapts automatically.

The bridge normalizes both formats into a consistent `purchase` event with `transaction_id`, `value`, and `currency`.

## Quick Start

### 1. Add the Parent Receiver to Your Page

Add this script to any page that embeds the Limo Anywhere widget:

**Option A: External script (recommended)**
```html
<!-- ORES Bridge - Parent Receiver -->
<script
  data-la-bridge
  data-measurement-id="G-XXXXXXXXXX"
  src="https://yourcdn.com/parent-receiver.js"
></script>
```

**Option B: Inline script (paste directly)**

Use this when you can't upload files to your server (e.g., page builders, CMS restrictions).

**Steps:**
1. Open [`parent-receiver.js`](https://github.com/brandonedley/ores-bridge-standard/blob/main/parent-receiver.js) (or [`min/parent-receiver.min.js`](https://github.com/brandonedley/ores-bridge-standard/blob/main/min/parent-receiver.min.js) for smaller size)
2. Click "Raw" to view the plain code
3. Copy the entire file contents
4. Paste into your page like this:

```html
<!-- ORES Bridge - Parent Receiver -->
<script data-la-bridge data-measurement-id="G-XXXXXXXXXX">
(function() {
  'use strict';
  // ... rest of the pasted code goes here ...
})();
</script>
```

**Important:** The `data-la-bridge` and `data-measurement-id` attributes go on the opening `<script>` tag itself, not inside the JavaScript.

### 2. Deploy the Sender Tag to GTM

1. Open Google Tag Manager
2. Go to the GTM container for your **widget domain** (book.mylimobiz.com)
3. Create a new **Custom HTML** tag
4. Paste the contents of `gtm-sender-tag.html`
5. Set trigger: **DOM Ready - All Pages**
6. Set Tag Firing Priority: **1** (fires before other tags)
7. Save and publish

## Configuration Options

### Parent Receiver Options

| Attribute | Required | Default | Description |
|-----------|----------|---------|-------------|
| `data-la-bridge` | Yes | - | Marker to identify the script |
| `data-measurement-id` | No | Auto-detect | GA4 Measurement ID (e.g., `G-XXXXXXXXXX`) |
| `data-mode` | No | `auto` | `auto`, `gtag`, or `dataLayer` (see below) |
| `data-event-prefix` | No | `la_` | Prefix for dataLayer events |
| `data-allowed-origins` | No | `https://book.mylimobiz.com` | Comma-separated allowed origins |
| `data-debug` | No | `false` | Enable console logging |

### Height Resize Options

| Attribute | Required | Default | Description |
|-----------|----------|---------|-------------|
| `data-iframe-selector` | No | `iframe[src*="book.mylimobiz.com"]` | CSS selector for the iframe element |
| `data-min-height` | No | `400` | Minimum iframe height in pixels |
| `data-max-height` | No | `2000` | Maximum iframe height in pixels |
| `data-enable-height-resize` | No | `true` | Set to `"false"` to disable auto height |

### Examples

**Minimal (auto-detect GA4):**
```html
<script data-la-bridge src="parent-receiver.js"></script>
```

**With explicit GA4 ID:**
```html
<script
  data-la-bridge
  data-measurement-id="G-ABC123XYZ"
  src="parent-receiver.js"
></script>
```

**Full configuration:**
```html
<script
  data-la-bridge
  data-measurement-id="G-ABC123XYZ"
  data-mode="gtag"
  data-allowed-origins="https://book.mylimobiz.com,https://book.limoanywhere.com"
  data-max-height="3000"
  data-debug="true"
  src="parent-receiver.js"
></script>
```

**Inline (pasted code):**
```html
<script
  data-la-bridge
  data-measurement-id="G-ABC123XYZ"
  data-debug="true"
>
(function() {
  'use strict';
  var _LA_BRIDGE_ID = 'LADB-2026-EDLEY-7X9K2';
  // ... paste rest of parent-receiver.js or parent-receiver.min.js here ...
})();
</script>
```

> **Tip:** When pasting inline, all `data-*` attributes go on the `<script>` tag. The JavaScript inside reads them automatically.

## How Mode Detection Works

The `data-mode` attribute controls how the bridge fires events on your parent page:

| Mode | Behavior |
|------|----------|
| `gtag` | Fires `gtag("event", ...)` directly to GA4 |
| `dataLayer` | Pushes `{event: "la_purchase", ...}` to `window.dataLayer` for GTM to handle |
| `auto` | Detects which to use (default) |

**Auto mode logic:**

1. If you set `data-measurement-id="G-XXXXXX"` --> uses **gtag** mode
2. Else if `window.gtag` exists on your page --> uses **gtag** mode and auto-detects the measurement ID from your existing GA4 setup
3. Else --> uses **dataLayer** mode (lets your GTM handle it)

**When to use each:**

| Your setup | Recommended mode |
|------------|------------------|
| gtag.js loaded directly on page (no GTM) | `auto` or `gtag` |
| GTM manages all your GA4 tags | `dataLayer` |
| Both gtag.js and GTM | `gtag` with explicit measurement ID |

**Example: Force dataLayer mode for GTM**
```html
<script
  data-la-bridge
  data-mode="dataLayer"
  src="parent-receiver.js"
></script>
```
Then create GTM triggers for events like `la_purchase`, `la_generate_lead`.

## Events Tracked

| Event | Trigger | GA4 Ecommerce |
|-------|---------|---------------|
| `purchase` | Booking completed | Yes |
| `generate_lead` | Quote submitted (no price shown) | No (custom) |

## Quote Detection

The bridge automatically detects whether the user is completing a reservation (with pricing) or submitting a quote request (no pricing available).

When no price is available -- either because the operator has disabled pricing display or because no rates match the selected route -- the widget presents a quote/lead flow instead of a booking flow. In this case, the bridge fires `generate_lead` instead of `purchase`, keeping your ecommerce reports clean.

### Detection Signals

The sender tag checks three signals to determine quote vs. reservation mode:

| Signal | What It Checks | Priority |
|--------|---------------|----------|
| `#BookType` hidden input | `value` equals `"Quote"` | Primary |
| Button `onclick` attribute | Contains `"Quote"` in `#step2RateSection` | Secondary |
| Button text | Contains the word "Quote" in `#step2RateSection` | Tertiary |

If none of these signals indicate a quote, the bridge defaults to reservation mode and fires `purchase` as normal.

### How It Works

1. User completes the booking form in the widget
2. ORES fires a purchase/transaction event to the dataLayer
3. The sender tag intercepts the event and checks the quote signals
4. **Reservation mode:** Forwards the `purchase` event with `transaction_id`, `value`, and `currency`
5. **Quote mode:** Fires `generate_lead` instead, with `lead_source: "quote_request"` and `value: 0`

The detection result is cached for the session -- once determined, it does not change mid-flow.

## Event Data Examples

### purchase
```json
{
  "event": "purchase",
  "transaction_id": "14887",
  "currency": "USD",
  "value": 217.44
}
```

### generate_lead
```json
{
  "event": "generate_lead",
  "lead_source": "quote_request",
  "currency": "USD",
  "value": 0,
  "transaction_id": "14887",
  "booking_type": "quote",
  "is_quote": true
}
```

## Iframe Auto-Height Resize

The bridge automatically resizes the ORES iframe to match the widget's content height, eliminating scrollbars and double-scroll issues.

This works out of the box with no configuration required. The sender tag uses `ResizeObserver` to detect content height changes inside the widget and sends height updates to the parent page via `postMessage`. The receiver applies the new height to the iframe element, clamped to the configured min/max bounds.

**Key behaviors:**
- Height updates are throttled via `requestAnimationFrame` to avoid layout thrash
- A 5px change threshold prevents unnecessary updates
- Falls back to `setInterval` polling on browsers without `ResizeObserver` support

**To customize or disable**, use the height resize `data-*` attributes on the receiver (see [Height Resize Options](#height-resize-options)).

## Files

| File | Purpose | Where to install |
|------|---------|------------------|
| `parent-receiver.js` | Receives events on parent page | Your website |
| `gtm-sender-tag.html` | Sends events from widget | GTM Custom HTML |
| `min/gtm-sender.min.html` | Minified sender | GTM Custom HTML (alternative) |
| `min/parent-receiver.min.js` | Minified receiver | Your website (alternative) |

## Troubleshooting

### Events not appearing

1. **Check GTM is published**: Ensure the sender tag is published, not just saved
2. **Wait for cache**: GTM can take 30-60 seconds to propagate
3. **Enable debug mode**: Set `data-debug="true"` on the receiver
4. **Check console**: Look for `[LA-Bridge]` messages

### Events not firing (but debug works)

If `la_bridge_init` appears in debug mode but no `purchase` or `generate_lead` events fire:
- GTM container has the latest version published
- Widget page is fully loaded before interactions
- Complete the full booking or quote flow to trigger the event

**Note:** `la_bridge_init` fires only with `data-debug="true"` -- dataLayer only, bypassing GA4.

### GCLID not tracking

Ensure:
- Parent page has gtag.js loaded with your GA4 ID
- User arrived via a Google Ads click (check for `gclid` in URL)
- Receiver is using `gtag` mode (auto-detects if gtag exists)

### generate_lead firing instead of purchase

This is expected when the widget is in quote mode. Check:
- Whether the operator has pricing enabled for the selected route
- The `#BookType` hidden input value in the widget DOM
- Enable debug mode to see `[LA-Bridge Sender] Booking type from #BookType: quote` in the console

## How It Works

```
+------------------------------------------------------------------+
| Parent Page (yourdomain.com)                                      |
|                                                                   |
|  +-----------------------------------------------------------+   |
|  | iframe (book.mylimobiz.com)                                |   |
|  |                                                            |   |
|  |  GTM Sender Tag                                            |   |
|  |    +-- Intercepts dataLayer.push()                         |   |
|  |    +-- Detects quote vs reservation mode                   |   |
|  |    +-- Sends height updates via postMessage --------+      |   |
|  |    +-- Sends events via postMessage ------------+   |      |   |
|  |                                                 |   |      |   |
|  +-------------------------------------------------|---|------+   |
|                                                    |   |          |
|  Parent Receiver                                   |   |          |
|    +-- Receives postMessage <----------------------+   |          |
|    +-- Validates origin                                |          |
|    +-- Fires gtag('event', ...) or dataLayer.push()    |          |
|    +-- Resizes iframe height <-------------------------+          |
|                                                                   |
|  GA4 (yourdomain.com)                                             |
|    +-- Receives purchase / generate_lead events                   |
|    +-- Preserves GCLID attribution                                |
|                                                                   |
+------------------------------------------------------------------+
```

## Upgrade to Advanced

> Need full funnel tracking? **ORES Bridge -- Advanced** adds form_start, view_item_list, select_item, begin_checkout, add_payment_info events plus Enhanced Conversions (SHA-256 hashed PII for better Google Ads attribution). Contact your implementation team to upgrade.

## License

Copyright (c) 2026 Brandon Edley. All Rights Reserved.

Free for use with Limo Anywhere ORES integrations only. See [LICENSE](LICENSE) for details.

## Support

For issues or questions, contact your implementation team.
