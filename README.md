# 170 Eldridge — Event Hub

A GitHub Pages pipeline dashboard that wires together the inquiry form, quote builder, and contract generator.

---

## Repo structure

```
index.html       ← Hub dashboard (this file)
inquiry.html     ← Client-facing intake form (no changes needed)
quote.html       ← Quote builder with pre-fill from hub
contract.html    ← Contract generator with pre-fill from hub
README.md
```

---

## One-time setup: Google Cloud

You need a Google Cloud project with:
- **Google Sheets API** enabled
- An **OAuth 2.0 Client ID** with your GitHub Pages URL as an authorized origin

### Step-by-step

1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Create a new project (or use an existing one)
3. **APIs & Services → Enable APIs** → search for "Google Sheets API" → Enable
4. **APIs & Services → Credentials → Create Credentials → OAuth 2.0 Client ID**
   - Application type: **Web application**
   - Authorized JavaScript origins: add your GitHub Pages URL, e.g. `https://yourusername.github.io`
   - Also add `http://localhost` if testing locally
5. Copy the **Client ID** (looks like `123456789.apps.googleusercontent.com`)
6. Paste it into the `CONFIG.CLIENT_ID` value at the top of `index.html`

---

## Google Sheet column setup

The hub expects your Sheet to have these columns **in this order**, written by the Apps Script that receives the inquiry form:

| Col | Field |
|-----|-------|
| A | timestamp |
| B | refId |
| C | firstName |
| D | lastName |
| E | email |
| F | phone |
| G | company |
| H | referral |
| I | eventType |
| J | eventDate |
| K | doorsOpen |
| L | doorsClose |
| M | guestCount |
| N | doorStaff |
| O | rooftop |
| P | servicesNeeded |
| Q | outsideVendors |
| R | guestMethod |
| S | publicEvent |
| T | alcohol |
| U | ageReq |
| V | insurance |
| W | budget |
| X | payMethod |
| Y | notes |
| **Z** | **status** ← hub writes this |
| **AA** | **quoteTotal** ← hub writes this |
| **AB** | **quoteData** ← hub writes this |
| **AC** | **contractGenerated** ← hub writes this |

If your columns are in a different order, update the `COL` object in `index.html`.

---

## How the pipeline works

1. **Client submits inquiry form** → row appended to Sheet with status empty
2. **You open the hub** → sign in with Google → inquiry appears in "New" column
3. **Click an inquiry card** → detail panel opens
4. **Click "Build Quote →"** → quote builder opens pre-filled with:
   - Day of week (derived from event date)
   - Occupancy tier (from guest count)
   - Rental hours (from doors open/close times)
   - Bar stations (from alcohol field + guest count)
   - Security headcount (from guest count)
5. **Adjust and finalize quote** → click **"Save to Hub & return"** → quote total saved to Sheet, status → `quoted`
6. **Click "Build Contract →"** → contract generator opens pre-filled with:
   - All client contact info
   - All event dates and times (with automatic next-day detection for late events)
   - Guest count, rooftop/door staff toggles
   - Rental fee and deposit pulled from saved quote
7. **Generate the .docx** → click **"Mark as Contracted & return to Hub"** → status → `contracted`

---

## Oversight checkpoints

You always have full control before anything is saved or sent:
- **Before building a quote**: you see the full inquiry detail in the hub
- **On the quote builder**: all pre-filled values are editable; the blue banner reminds you to review
- **On the contract builder**: same — all fields are pre-filled but editable; the generate button only creates a local file

Nothing is auto-sent to the client at any step.

---

## Deploying to GitHub Pages

1. Push all files to a GitHub repo
2. **Settings → Pages → Source**: select `main` branch, `/ (root)`
3. Your hub will be live at `https://yourusername.github.io/repo-name/`
4. Add that URL to your Google Cloud OAuth authorized origins
