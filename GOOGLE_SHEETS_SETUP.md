# Waitlist / demo email capture

The email box in the `Get started` section at the bottom of [index.html](index.html)
is a real form. Its config block sits in the `EMAIL CAPTURE` script near the end of
the file:

```javascript
const FORMSPREE_URL = "https://formspree.io/f/meedaaov";  // active
const SHEETS_URL    = "";                                 // paste /exec URL to enable
const CALENDLY_URL  = "https://calendly.com/chetan-kasim";
```

Both sinks are independent — leave one blank and the other still works.

| Sink | Cost | Good for | Notes |
| --- | --- | --- | --- |
| **Formspree** (active) | Free tier: 50 submissions/month | Instant email notification + dashboard, no setup | Already wired to form `meedaaov`. Paid plan needed past 50/month. |
| **Google Sheets** (Apps Script) | Free | A spreadsheet you can sort, filter and export | No notification unless you add one; setup below. |
| **Formspree + Sheets** | Free | Notification *and* a durable list | Set both constants. |

Each submission sends `email`, `source`, `page` and `referrer`.

---

## Enabling the Google Sheet

### Step 1 — Create the sheet

1. Go to [sheets.google.com](https://sheets.google.com) and create a spreadsheet
2. Name it **"GaiaLM Waitlist"**
3. In Row 1 add these headers: `Timestamp`, `Email`, `Source`, `Page`, `Referrer`

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

The browser posts to Apps Script with `mode: 'no-cors'` (Apps Script sends no CORS
headers on its response), so the page cannot see whether the row landed. Formspree
is what decides the success/error message the visitor sees — if you run Sheets
*only*, the form will always report success, so check the sheet after a test submit.

---

## Troubleshooting

**Nothing arrives from Formspree** — confirm form `meedaaov` is still active in the
Formspree dashboard and that the monthly quota isn't used up. A failed POST shows the
visitor a `hello@gaialm.ai` fallback and logs the status to the browser console.

**Nothing arrives in the sheet**
- Check **Extensions → Apps Script → Executions** for errors
- "Who has access" must be **Anyone**
- After editing the script: **Deploy → Manage deployments → Edit → New version → Deploy**
  (the `/exec` URL stays the same)
