# Nikah Invitation — Eleyas & Reshma

An interactive, mobile-first invitation: a personalised entry, a walk through
seven lantern-lit chapters, then the invitation itself (date reveal, countdown,
venue, RSVP, blessings).

Built as a single self-contained `index.html` — no build step, no dependencies
to install. Open it in a browser and it runs.

---

## 1. Add your files

Put these in the `assets/` folder, named exactly:

| File | What it is | Required? |
|---|---|---|
| `assets/hero.jpg` | Opening full-screen photo (portrait) | optional |
| `assets/couple.jpg` | Photo behind the names / welcome-back screen | optional |
| `assets/music.mp3` | Background audio, plays softly with a mute button | optional |
| `assets/upi-qr.png` | The gift QR shown in the Blessings section | optional |

If any file is missing, the page falls back to its animated artwork and keeps
working. Keep photos under ~1.5 MB each so the page loads fast on mobile data.

## 2. Edit your details

Everything you'd want to change lives in one block near the bottom of
`index.html` — search for `const CONFIG`:

```js
const CONFIG={
  upiId:        'www.sha1425-4@okicici',
  upiName:      'Eleyas & Reshma',
  qrImage:      'assets/upi-qr.png',   // delete this line to generate a QR from upiId instead
  whatsappPhone:'',                    // e.g. 919876543210 (no +). Blank = share sheet
  mapsQuery:    'Hazrath Teepu Aoullya Dargah Masjid Madrasa E Riyazul Jannah',
  mapsLink:     '',                    // paste an exact Google Maps link to override
  eventISO:     '2026-09-24T11:30:00+05:30',
  eventEndISO:  '2026-09-24T14:00:00+05:30',
};
```

The Blessings section shows Eleyas's own GPay QR (`assets/upi-qr.png`). Delete the
`qrImage` line and the page generates a QR from `upiId` instead; blank the `upiId`
and the whole gift block hides itself.

Other things you may want to edit, all plain text in `index.html`:

- **Order of the day** — search for `Order of the day`. The three times
  (Nikah 11:30 AM, Du'ā, Walima 12:30 PM) are a sensible guess — correct them.
- **Chapter text** — search for `const CHAPTERS`; the seven chapters, their
  Arabic line and English line are all there.
- **Names / venue lines** — search for `Reshma` or `Riyazul`.

## 3. Collect RSVPs in a Google Sheet (optional)

Without this, RSVPs still work — they are stored on the guest's own device and
you can read them via the tiny `· admin ·` link at the bottom of the RSVP
section. To collect them centrally:

1. Open <https://sheets.new> and rename the tab to **RSVPs**
2. **Extensions → Apps Script**, paste this, Save:

```js
function doPost(e) {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sheet = ss.getSheetByName('RSVPs') || ss.getActiveSheet();
  if (sheet.getLastRow() === 0) {
    sheet.appendRow(['Timestamp','Type','Name','Phone','Guests','Attending','Message']);
  }
  var d = JSON.parse(e.postData.contents);
  sheet.appendRow([d.ts, d.type || 'rsvp', d.name, d.phone || '', d.guests || '',
                   d.attending === undefined ? '' : (d.attending ? 'Yes' : 'No'), d.msg || '']);
  return ContentService.createTextOutput(JSON.stringify({status:'ok'}))
    .setMimeType(ContentService.MimeType.JSON);
}
```

3. **Deploy → New deployment → Web app**; *Execute as:* **Me**,
   *Who has access:* **Anyone**. Copy the Web App URL.
4. In `index.html`, paste it into `GOOGLE_SHEET_SCRIPT_URL` (just above `CONFIG`).

Every guest who enters their name at the gate is logged too, so you can see who
has opened the invite.

## 4. Put it on your domain

### Option A — GitHub Pages (free, same as the previous invite)

```bash
# inside this folder
git add -A
git commit -m "Nikah invitation"
git remote add origin https://github.com/<your-username>/<repo-name>.git
git branch -M main
git push -u origin main
```

Then on GitHub: **Settings → Pages → Source: Deploy from branch → main / (root)**.

For your own domain, create a file named `CNAME` in this folder containing just
your domain, e.g.:

```
eleyasreshma.com
```

commit and push it, then at your domain registrar add:

- an **A record** for `@` → `185.199.108.153`, `185.199.109.153`,
  `185.199.110.153`, `185.199.111.153`
- a **CNAME record** for `www` → `<your-username>.github.io`

Back in **Settings → Pages**, enter the domain under *Custom domain* and tick
*Enforce HTTPS* once the certificate is issued (can take up to an hour).

### Option B — Netlify / Cloudflare Pages (fastest, no git needed)

Drag this whole folder onto <https://app.netlify.com/drop>, then
**Domain settings → Add custom domain** and follow the DNS instructions it gives
you. Re-drag the folder any time you change something.

### Option C — any web host

Upload `index.html` and the `assets/` folder to your hosting root. That's all
there is; there is nothing to build or configure.

---

## Notes

- Works offline-ish: the only external requests are Google Fonts and one small
  QR-code library from a CDN. If they're blocked, the page still renders with
  fallback fonts and a plain-text UPI ID.
- A guest's name is remembered on their device, so returning guests get a
  "welcome back" screen and skip straight into the walk.
- Tested at phone (390px) and desktop widths.
