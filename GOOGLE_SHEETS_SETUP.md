# Google Sheets Waitlist Setup

## Step 1: Create the Sheet

1. Go to [sheets.google.com](https://sheets.google.com)
2. Create a new spreadsheet
3. Name it **"GaiaLM Waitlist"**
4. In Row 1, add these headers:
   - A1: `Timestamp`
   - B1: `Email`

## Step 2: Add the Script

1. In your sheet, go to **Extensions → Apps Script**
2. Delete any existing code
3. Paste this code:

```javascript
function doPost(e) {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const data = JSON.parse(e.postData.contents);

  sheet.appendRow([
    new Date(),
    data.email
  ]);

  return ContentService
    .createTextOutput(JSON.stringify({ success: true }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

4. Click **Save** (Ctrl+S / Cmd+S)

## Step 3: Deploy

1. Click **Deploy → New deployment**
2. Click the gear icon → Select **Web app**
3. Fill in:
   - Description: `Waitlist API`
   - Execute as: `Me`
   - Who has access: **Anyone**
4. Click **Deploy**
5. Click **Authorize access** → Choose your Google account → Allow
6. Copy the **Web app URL**

## Step 4: Update the Landing Page

Give the Web App URL to update `index.html`. The URL looks like:
```
https://script.google.com/macros/s/AKfycbw.../exec
```

---

## Troubleshooting

**"Authorization required" error:**
- Make sure you clicked "Authorize access" during deployment

**Submissions not appearing:**
- Check the Apps Script execution log: Extensions → Apps Script → Executions
- Make sure "Who has access" is set to "Anyone"

**Need to update the script:**
- Go to Extensions → Apps Script
- Make changes
- Deploy → Manage deployments → Edit → New version → Deploy
