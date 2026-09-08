# Linking the tracker to a Google Sheet

Once this is done, the page maintains itself: anyone who can edit the sheet updates the
website, and nobody has to touch the HTML file again.

Roughly ten minutes, start to finish.

---

## 1 · Make the sheet

Import [`trek-submissions-template.csv`](trek-submissions-template.csv) into a new Google
Sheet — **File → Import → Upload**, and choose *Replace spreadsheet*. That gives you the
right column headers and a few example rows to overwrite.

| column | what goes in it | required |
|---|---|---|
| `name` | Who covered it. Repeat the same name across rows and the page merges them into one trekker. | yes |
| `km` | Distance in kilometres. Put `0` for a donation with no distance behind it. | yes* |
| `gbp` | Money raised — sign-up plus sponsorship. Numbers only, no `£` sign. | yes* |
| `activity` | One of `walk` `run` `cycle` `swim` `hike` `scoot` `other`. Anything else is filed as *other*. | no |
| `date` | When it was logged. Drives the week-by-week timeline. | no |
| `note` | A line shown when someone hovers that stretch of road. Commas and line breaks are fine. | no |

\* A row needs *either* a distance or an amount. Rows with neither are ignored, so a
half-finished line at the bottom does no harm.

**Two things that matter:**

- **Row order is route order.** The page lays contributions along the road in the order
  they appear, so the first row starts at Wellington. Add new entries at the bottom.
- **Don't delete or rename the header row.** Capitalisation doesn't matter (`Name` and
  `name` both work), but the words do.

Dates can be `01/09/2026` or `2026-09-01` — both are read as 1 September. The page assumes
day-first, British style. (If you ever switch the sheet to a US locale, set
`dayFirstDates: false` in the file.)

---

## 2 · Publish it

This is the step people get wrong. **"Share → Anyone with the link" is not enough** — the
page needs the *published* address, which is a different thing.

1. **File → Share → Publish to web**
2. Under *Link*, pick the specific tab (not "Entire document")
3. Change **Web page** to **Comma-separated values (.csv)**
4. Click **Publish**, and confirm
5. Copy the URL. It should look like:

       https://docs.google.com/spreadsheets/d/e/2PACX-1vQ.../pub?gid=0&single=true&output=csv

If your URL doesn't contain `/d/e/` and end in `output=csv`, it's the wrong one.

Publishing makes only that tab readable to anyone with the link. Keep anything you don't
want public on a separate, unpublished tab.

---

## 3 · Point the page at it

Open `trek-tracker.html` in any text editor and find `CONFIG` near the top. Change two
lines:

```js
mode: 'fetch',                        // was 'inline'
endpoint: 'https://docs.google.com/spreadsheets/d/e/2PACX-1vQ.../pub?gid=0&single=true&output=csv',
```

That's the whole change. Save, upload, done.

The orange **"Sample data"** badge disappears on its own — it only shows in `inline` mode.

---

## 4 · Check it worked

Open the page and look at the top right:

- **"Auto-updating"** — it's reading the sheet. 
- **"Sample data"** — still on the built-in list; `mode` didn't change.
- Everything at zero — it reached the sheet but read no rows. Check the header row.

The footer shows how many submissions it loaded, which is the quickest way to confirm the
whole sheet came through.

---

## Living with it

- **The page re-reads the sheet every 60 seconds**, so an open browser updates itself.
- **Google caches published sheets for up to about 5 minutes**, so an edit takes a few
  minutes to appear on the site. That's Google's cache, not the page — waiting is the only
  fix. It is not broken.
- **Nothing to maintain at our end.** No plugin, no server code, no API key, no password.
- **Deleting a row** removes it from the site at the next refresh. There's no undo beyond
  the sheet's own version history, so it's worth turning that on.

### Letting people submit their own totals

Attach a **Google Form** to the sheet (**Tools → Create a new form**) with the same fields.
Responses land as new rows and appear on the site within minutes, without anyone
re-typing them. Note that Forms write to a *Form responses* tab — publish that tab, and
add a `gbp` column to it if the form doesn't collect money.
