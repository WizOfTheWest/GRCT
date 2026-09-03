# The Great Garden Route Trek — progress tracker

A single self-contained HTML page that tracks the [Garden Route Children's Trust](https://www.grct.org.uk/)
20th-anniversary trek: 1,000 km from Wellington to Port Elizabeth, lit up kilometre by
kilometre as supporters send in the distances they've walked, run, cycled or swum.

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
- **A finish celebration** that plays on every visit once the route is complete — and the
  moment a live submission tips it past 1,000 km while someone is watching.
- Light and dark themes, responsive down to a phone, and reduced-motion support.

## Adding submissions

Today the numbers are typed in. Everything lives in `CONFIG` near the top of the `<script>`:

```js
CONFIG.data.inline = [
  { name: 'Penny Fleming', km: 42, activity: 'walk', date: '2026-09-01', note: '…' },
  …
]
```

Set `CONFIG.data.sample = false` once the real list replaces the placeholder — that clears
the orange "Sample data" badge. `activity` is one of
`walk · run · cycle · swim · hike · scoot · other`. Order matters: contributions are laid
along the road in the order they appear.

## When it gets automated

Change `CONFIG.data.mode` and nothing else:

| mode | what it does |
|---|---|
| `inline` | the list in this file (the default) |
| `fetch` | polls `CONFIG.data.endpoint` every `pollMs` — JSON or CSV, including a published Google Sheet |
| `live` | a push feed; implement `CONFIG.data.connect(push)` for SSE, WebSocket, Firebase, Supabase… |

`CONFIG.data.normalise()` maps whatever shape your source returns onto
`{ id, name, km, activity, date, note }`, so column names don't have to match.

A live system can also drive the page directly:

```js
GRCTTrek.add({ name: 'Ada L', km: 12, activity: 'run', date: '2026-09-14' })
GRCTTrek.set([ …all contributions… ])
GRCTTrek.refresh()          // re-pull from the endpoint
GRCTTrek.state              // { totalKm, complete, trekkers, legs, … }
GRCTTrek.zoomTo(0, 95)      // fly the map to a kilometre range
```

## Notes

- **Tiles.** OpenStreetMap's own tiles need no key and are fine for a site this size under
  the [tile usage policy](https://operations.osmfoundation.org/policies/tiles/). They're
  desaturated in CSS so the orange route stays the loudest thing on the map. If traffic
  grows, `CONFIG.map.tiles` has commented one-line swaps for CARTO or Stadia with a free
  key — set `filterTiles: false` to let their own styling through.
- **Leg distances** are GRCT's published figures. The drawn road is shorter on two legs;
  each leg maps its own published kilometres onto its own drawn length, so the towns always
  sit at the right cumulative distance.
- **Dot granularity** is `CONFIG.map.lod`. Radii are tuned so dots never merge into a line —
  keep a tier's radius under about 0.36 × its on-screen spacing.

## Built with

[Leaflet](https://leafletjs.com/) · [OpenStreetMap](https://www.openstreetmap.org/copyright) ·
[OSRM](https://project-osrm.org/) · [motion.dev](https://motion.dev/) ·
design tokens from [Bklit UI](https://ui.bklit.com/)
