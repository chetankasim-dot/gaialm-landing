# Waitlist / demo email capture

The email box in the `Get started` section at the bottom of [index.html](index.html)
is a real form. Its config block sits in the `EMAIL CAPTURE` script near the end of
the file:

```javascript
const FORMSPREE_URL = "https://formspree.io/f/moeayybp";                          // active
const SHEETS_URL    = "https://script.google.com/macros/s/AKfycbxE.../exec";      // active
const CALENDLY_URL  = "https://calendly.com/chetan-kasim";
```

Both sinks are live and independent: the page writes to both and shows the visitor
success if **either** one accepts the row, so a Formspree quota overrun or an Apps
Script outage doesn't lose the lead. Failures are logged to the browser console.

| Sink | Cost | Good for | Notes |
| --- | --- | --- | --- |
| **Formspree** (active) | Free tier: 50 submissions/month | Instant email notification + dashboard, no setup | Already wired to form `moeayybp`. Paid plan needed past 50/month. |
| **Google Sheets** (Apps Script) | Free | A spreadsheet you can sort, filter and export | No notification unless you add one; setup below. |
| **Formspree + Sheets** | Free | Notification *and* a durable list | Set both constants. |

Each submission sends `email`, `source`, `page` and `referrer`.

---

## Enabling the Google Sheet

The sheet in use is
[GaiaLM Waitlist](https://docs.google.com/spreadsheets/d/1I57zjV1OpEGTBHMu-EoT5Z9m6SsOf1QosV_Mb2fK-vY/edit?gid=0#gid=0)
(spreadsheet ID `1I57zjV1OpEGTBHMu-EoT5Z9m6SsOf1QosV_Mb2fK-vY`).

### Step 1 — Headers

In Row 1 of that sheet, add: `Timestamp`, `Email`, `Source`, `Page`, `Referrer`

### Step 2 — Add the script

**Extensions → Apps Script**, delete the placeholder code, paste this and **Save**:

```javascript
function doPost(e) {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const data = JSON.parse(e.postData.contents);

  sheet.appendRow([
    new Date(),
    data.email || '',
    data.source || '',
    data.page || '',
    data.referrer || ''
  ]);

  return ContentService
    .createTextOutput(JSON.stringify({ success: true }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

### Step 3 — Deploy

1. **Deploy → New deployment**, gear icon → **Web app**
2. Description `Waitlist API`; Execute as **Me**; Who has access **Anyone**
3. **Deploy** → **Authorize access** → pick your account → **Allow**
4. Copy the **Web app URL** — it looks like
   `https://script.google.com/macros/s/AKfycbw.../exec`

### Step 4 — Wire it up

Paste that URL into `SHEETS_URL` in [index.html](index.html) and redeploy the site.
**Done — the current deployment is already wired in and verified.**

The post uses `Content-Type: text/plain` on purpose: that keeps it a "simple" CORS
request, because Apps Script answers no `OPTIONS` preflight. Its `/exec` URL 302s to
a `script.googleusercontent.com` echo response, and *that* response does carry
`Access-Control-Allow-Origin: *` — so the page can read the `{"success":true}` body
and genuinely knows whether the row landed.

---

## Troubleshooting

**Nothing arrives from Formspree** — confirm form `moeayybp` is still active in the
Formspree dashboard and that the monthly quota isn't used up. A failed POST shows the
visitor a `hello@gaialm.ai` fallback and logs the status to the browser console.

**Nothing arrives in the sheet**
- Check **Extensions → Apps Script → Executions** for errors
- "Who has access" must be **Anyone**
- Redeploying Apps Script can mint a new `/exec` URL — if you used
  **New deployment** rather than **Manage deployments → Edit**, update `SHEETS_URL`
- After editing the script: **Deploy → Manage deployments → Edit → New version → Deploy**
  (the `/exec` URL stays the same)
