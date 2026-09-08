# The Great Garden Route Trek — progress tracker

A single self-contained HTML page that tracks the [Garden Route Children's Trust](https://www.grct.org.uk/)
20th-anniversary trek: 1,000 km from Wellington to Port Elizabeth (or 2,000, there and back —
one switch), lit up kilometre by kilometre as supporters send in the distances they've walked,
run, cycled or swum, and the money they've raised towards a £10,000 target. Open right through
the autumn, with no hard stop at the end of September.

**[`trek-tracker.html`](trek-tracker.html)** — that's the whole thing. Open it in a browser,
or drop it on the website. No build step, no server, no API keys.

![The trek map](https://img.shields.io/badge/status-ready%20to%20deploy-1f8454)

## What it does

- **A real map.** Leaflet over OpenStreetMap tiles, with the route traced along the actual
  N2 and coastal roads (geometry from OSRM, simplified to 18 m).
- **Dots that subdivide as you zoom.** One dot is 20 km on the whole-route view and 100 m
  at street level — eight tiers, with only the dots in view drawn.
- **A shade of orange per trekker**, stable as the data grows, so you can see who covered
  which stretch. Hover to highlight; click a name, a leg or a submission to fly there.
- **A week-by-week timeline.** A slider beside the map (under it on a phone) winds the
  whole page back through the autumn — the road un-lights, and every number follows. Press
  play to watch the weeks run.
- **Money tracked alongside distance**, against a £10,000 target.
- **One switch to double the route** — out to Port Elizabeth and all the way home again.
- **A finish celebration** that plays on every visit once the route is complete — and the
  moment a live submission tips it past the goal while someone is watching.
- Light and dark themes, responsive down to a phone, and reduced-motion support.

## The dials

All at the top of `CONFIG`, near the start of the `<script>`:

```js
returnLeg: false,   // false = 1,000 km one way.  true = 2,000 km, out and back.
goalGbp:  10000,    // the fundraising target
returnDoublesTarget: false,   // does doubling the route double the money target too?
```

`returnLeg` is the only thing you change to extend the challenge. Flip it and the goal, every
percentage, the kilometres remaining, the leg table, the town milestones and the finish line
all follow. On the map the way home gets its own lane a few pixels off the road, so the two
directions never sit on top of each other.

The £10,000 target stands whichever route you run, unless you also set
`returnDoublesTarget: true`.

## Adding submissions

Today the numbers are typed in. Everything lives in `CONFIG.data`:

```js
CONFIG.data.inline = [
  { name: 'Penny Fleming', km: 42, gbp: 320, activity: 'walk', date: '2026-09-01', note: '…' },
  …
]
```

Set `CONFIG.data.sample = false` once the real list replaces the placeholder — that clears
the orange "Sample data" badge. `activity` is one of
`walk · run · cycle · swim · hike · scoot · other`. Order matters: contributions are laid
along the road in the order they appear.

`gbp` is money raised — sign-up plus sponsorship. A donation with no distance behind it is
fine: give it `km: 0` and it still counts towards the total. `date` drives the timeline, so
it's worth filling in; undated entries are always counted, at every point on the slider.

## When it gets automated

Change `CONFIG.data.mode` and nothing else:

| mode | what it does |
|---|---|
| `inline` | the list in this file (the default) |
| `fetch` | polls `CONFIG.data.endpoint` every `pollMs` — JSON or CSV, including a published Google Sheet (Google serves those with permissive CORS, so the browser can read one directly) |
| `live` | a push feed; implement `CONFIG.data.connect(push)` for SSE, WebSocket, Firebase, Supabase… |

`CONFIG.data.normalise()` maps whatever shape your source returns onto
`{ id, name, km, activity, date, note }`, so column names don't have to match.

A live system can also drive the page directly:

```js
GRCTTrek.add({ name: 'Ada L', km: 12, gbp: 25, activity: 'run', date: '2026-09-14' })
GRCTTrek.set([ …all contributions… ])
GRCTTrek.refresh()          // re-pull from the endpoint
GRCTTrek.state              // { totalKm, gbp, goalGbp, returnLeg, week, legs, … }
GRCTTrek.zoomTo(0, 95)      // fly the map to a kilometre range
GRCTTrek.setWeek(3)         // rewind the page; setWeek('now') returns to live
GRCTTrek.playWeeks()        // run the timeline
```

## The timeline

Weeks run Monday to Monday and are derived from the submissions themselves — from the
earliest date to whichever is later of the last submission and today. Nothing to configure:
the track simply grows a notch each week as the autumn goes on, and thins its tick marks out
once there are more than 26. Rewinding never triggers the finish celebration, and the numbers
stop animating while you drag so scrubbing stays responsive.

## Putting it on the GRCT website

The file is a complete page, so the simplest route is to upload it and link to it. To place
it inside an existing page instead, add `?embed=1` — that hides this page's own header,
footer and top rule so the site's own chrome isn't duplicated:

```html
<iframe src="/trek-tracker.html?embed=1" title="Great Garden Route Trek tracker"
        style="width:100%;height:2500px;border:0" loading="lazy"></iframe>
```

The iframe needs a fixed height (there is no auto-resize yet — say the word and it's a small
addition). Nothing else is needed: no plugin, no server-side code, no API keys.

## Notes

- **Tiles.** OpenStreetMap's own tiles need no key and are fine for a site this size under
  the [tile usage policy](https://operations.osmfoundation.org/policies/tiles/). They're
  desaturated in CSS so the orange route stays the loudest thing on the map. If traffic
  grows, `CONFIG.map.tiles` has commented one-line swaps for CARTO or Stadia with a free
  key — set `filterTiles: false` to let their own styling through.
- **Leg distances** are GRCT's published figures. The drawn road is shorter on two legs;
  each leg maps its own published kilometres onto its own drawn length, so the towns always
  sit at the right cumulative distance.
- **Wording.** The page says the trek stays open through the autumn, with no hard stop at the
  end of September — hero, the "Join the trek" card, and a badge by the headline.
- **Dot granularity** is `CONFIG.map.lod`. Radii are tuned so dots never merge into a line —
  keep a tier's radius under about 0.36 × its on-screen spacing.

## Built with

[Leaflet](https://leafletjs.com/) · [OpenStreetMap](https://www.openstreetmap.org/copyright) ·
[OSRM](https://project-osrm.org/) · [motion.dev](https://motion.dev/) ·
design tokens from [Bklit UI](https://ui.bklit.com/)
