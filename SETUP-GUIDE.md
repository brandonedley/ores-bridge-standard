# ORES Bridge -- Standard: Setup Guide

Step-by-step setup instructions for purchase conversion tracking and GCLID attribution.

---

## Prerequisites

- [ ] Access to Google Tag Manager for the **widget domain** (book.mylimobiz.com)
- [ ] Access to edit your website where the widget is embedded
- [ ] GA4 property set up on your domain

---

## Part 1: Deploy the GTM Sender Tag (Widget Side)

### Step 1: Open Google Tag Manager

1. Go to [tagmanager.google.com](https://tagmanager.google.com)
2. Select the GTM container for `book.mylimobiz.com`
   - Container ID format: `GTM-XXXXXXX`

### Step 2: Create New Tag

1. Click **Tags** in the left sidebar
2. Click **New** button
3. Name the tag: `ORES Bridge - Widget Sender`

### Step 3: Configure Tag

1. Click **Tag Configuration**
2. Select **Custom HTML**
3. Copy the **entire contents** of `gtm-sender-tag.html`
4. Paste into the HTML field

### Step 4: Configure Trigger

1. Click **Triggering**
2. Click the **+** icon to add a new trigger
3. Click **+** again to create new trigger
4. Name it: `DOM Ready - All Pages`
5. Select trigger type: **DOM Ready**
6. Set "This trigger fires on": **All Pages**
7. Save the trigger

### Step 5: Set Tag Priority

1. In the tag editor, click **Advanced Settings**
2. Set **Tag firing priority**: `1`
   - This ensures the bridge loads before other tags

### Step 6: Save and Publish

1. Click **Save**
2. Click **Submit** (top right)
3. Add version name: `ORES Bridge Sender v1.0`
4. Click **Publish**

---

## Part 2: Add the Parent Receiver (Your Website)

### Option A: Script Tag (Recommended)

Add this to your page `<head>` or before the closing `</body>`:

```html
<!-- ORES Bridge - Parent Receiver -->
<script
  data-la-bridge
  data-measurement-id="G-XXXXXXXXXX"
  src="https://yourcdn.com/parent-receiver.js"
></script>
```

Replace:
- `G-XXXXXXXXXX` with your GA4 Measurement ID
- Update the `src` to where you host the file

### Option B: Inline Script

Copy the contents of `parent-receiver.js` and add directly to your page:

```html
<!-- ORES Bridge - Parent Receiver -->
<script data-la-bridge data-measurement-id="G-XXXXXXXXXX">
  // Paste contents of parent-receiver.js here
</script>
```

---

## Part 3: Verify Installation

### Test 1: Check Bridge Initialization

1. Open your page with the embedded widget
2. Open browser DevTools (F12)
3. Go to Console tab
4. Add `data-debug="true"` to the receiver script tag temporarily
5. Look for: `[LA-Bridge] Receiver initialized`

### Test 2: Test Purchase Event

1. Complete a full booking through the widget (use a test reservation if possible)
2. Check console for: `purchase` event with `transaction_id`, `value`, and `currency`

### Test 3: Test Quote Detection

If your ORES configuration shows quotes (no pricing):

1. Submit a quote request through the widget
2. Check console for: `generate_lead` event with `lead_source: "quote_request"`
3. Confirm that `purchase` did NOT fire (only `generate_lead` should appear)

### Test 4: Verify in GA4 DebugView

1. Go to [analytics.google.com](https://analytics.google.com)
2. Select your property
3. Go to **Admin** --> **DebugView**
4. Complete a booking flow
5. Verify events appear:
   - `purchase` (for reservations with pricing)
   - `generate_lead` (for quote submissions without pricing)

### Test 5: Verify Iframe Height Resize

1. Load the page with the embedded widget
2. Navigate through the booking flow (different steps have different heights)
3. Confirm the iframe resizes to fit the content without scrollbars

---

## Configuration Reference

### Parent Receiver Data Attributes

| Attribute | Required | Default | Description |
|-----------|----------|---------|-------------|
| `data-la-bridge` | Yes | - | Required marker |
| `data-measurement-id` | | Auto | GA4 ID (e.g., `G-ABC123`) |
| `data-mode` | | `auto` | `auto`, `gtag`, or `dataLayer` |
| `data-event-prefix` | | `la_` | Prefix for dataLayer events |
| `data-allowed-origins` | | `https://book.mylimobiz.com` | Allowed iframe origins |
| `data-debug` | | `false` | Enable console logging |

### Height Resize Attributes

| Attribute | Required | Default | Description |
|-----------|----------|---------|-------------|
| `data-iframe-selector` | | `iframe[src*="book.mylimobiz.com"]` | CSS selector for the iframe |
| `data-min-height` | | `400` | Minimum height in pixels |
| `data-max-height` | | `2000` | Maximum height in pixels |
| `data-enable-height-resize` | | `true` | Set to `"false"` to disable |

### Example Configurations

**Minimal:**
```html
<script data-la-bridge src="parent-receiver.js"></script>
```

**With Debug:**
```html
<script
  data-la-bridge
  data-measurement-id="G-ABC123XYZ"
  data-debug="true"
  src="parent-receiver.js"
></script>
```

**Multiple Origins:**
```html
<script
  data-la-bridge
  data-measurement-id="G-ABC123XYZ"
  data-allowed-origins="https://book.mylimobiz.com,https://book.limoanywhere.com"
  src="parent-receiver.js"
></script>
```

---

## Troubleshooting

### No events appearing

**Check GTM is published:**
- Go to GTM --> Versions
- Verify latest version is published
- Wait 1-2 minutes for CDN propagation

**Check iframe is loading:**
- Right-click the widget --> Inspect
- Verify iframe src is `book.mylimobiz.com`

**Enable debug mode:**
```html
<script data-la-bridge data-debug="true" ...>
```

### No events appearing (debug mode)

With `data-debug="true"` set, you should see `la_bridge_init` in the console and dataLayer. If not:
1. Verify sender tag is published in GTM
2. Clear browser cache (Ctrl+Shift+R)
3. Wait 60 seconds for GTM CDN

**Note:** `la_bridge_init` fires only in debug mode -- dataLayer only, bypassing GA4.

### Events fire but not in GA4

**Check:**
1. Measurement ID is correct
2. gtag.js is loaded on the page
3. No ad blockers interfering

### GCLID not being tracked

**Ensure:**
1. User arrived via Google Ads click
2. `_gcl_aw` cookie exists on your domain
3. Receiver is using `gtag` mode (not `dataLayer`)

---

## Events Reference

| Event | Trigger | GA4 Report |
|-------|---------|------------|
| `purchase` | Booking completed (with pricing) | Ecommerce |
| `generate_lead` | Quote submitted (no pricing) | Events |

---

## Support

For technical issues, check:
1. Browser console for errors
2. Network tab for blocked requests
3. GTM Preview mode for tag firing

Need help? Contact your implementation team.
