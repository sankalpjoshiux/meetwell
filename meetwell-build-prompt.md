# Build prompt: "Meetwell" — a family/meeting timezone finder

Copy everything below into your coding agent (Claude Code, Cursor, etc.) as the task brief.

---

## What to build

A web app called **Meetwell** that helps people find good times to talk when they're spread across timezones — built for family calls, but works equally for social or professional meetings. The core idea: show each person's day as a horizontal strip of color (day/dawn-dusk/night), stack everyone's strips together, and visually highlight the hours where everyone is free at once.

## Core features

1. **Add people** — a name field plus a city/timezone field. The city field must support both typing-to-search and browsing a dropdown list (use a native `<input list>` + `<datalist>` pairing, or a custom combobox). Store each person's IANA timezone (e.g. `Asia/Kolkata`), not just a UTC offset — offsets change with DST, IANA zones don't.

2. **24-hour strip per person** — for each person, render 24 colored blocks representing their day. Color-code by time of day:
   - Day (9am–6pm): warm gold/sand
   - Dawn/dusk (6–9am, 6–9pm): dusty terracotta/rose
   - Night (9pm–6am): muted indigo/slate
   Keep these desaturated/subtle, not neon — they're data, not decoration.

3. **Benchmark timezone** — a selector (defaults to the visitor's detected local timezone via `Intl.DateTimeFormat().resolvedOptions().timeZone`) that all 24 hour-columns are aligned against. Changing it re-aligns every strip. Include an option to add "You" as a row using the detected timezone.

4. **Day scrubber** — arrows (or similar) to step the whole grid forward/backward a day at a time, with a "jump to today" shortcut. Label each day clearly (e.g. "Today · Sep 15", "Tomorrow · Sep 16").

5. **Free-time range control** — let the user define "free" hours (e.g. 8am–9pm) applied to everyone's own local time. Highlight the columns where every person falls inside their free range with a visible outline.

6. **Drag-to-pick window** — let the user click-and-drag directly across the strips to select an arbitrary time window, and show a live summary card below listing each person's corresponding local time for that exact window. This should feel direct — no separate slider UI, the grid itself is the input.

7. **Recommended windows** — automatically rank and suggest the best 1–3 time windows using a simple comfort score (daytime scores highest, dawn/dusk is acceptable, night is heavily penalized — never recommend waking someone at 3am). Show this above the manual free-time-range results, since it doesn't depend on the user configuring anything.

8. **Settings panel** — a small gear icon opening a popover with:
   - Light/dark theme toggle (build this as a real token-based theme system — a `{ light: {...}, dark: {...} }` object with every color derived from it, not hardcoded hex values scattered through the component. This avoids partial-theme bugs.)
   - 12-hour / 24-hour clock format toggle, applied everywhere a time is displayed

9. **Footer credit** — small centered line: "Made with ❤️ by [name]" linking to a personal site, plus a Buy Me a Coffee button widget.

## Design direction

Visual style: **minimalist, modern glassmorphism** — Apple's restraint (clean sans-serif type, generous whitespace, one soft light source, subtle hairline borders) combined with web3-style translucency (frosted blur cards, soft glow accents, rounded pill buttons). Use Inter or a similar clean sans-serif throughout — no serif fonts, no decorative typefaces. Keep block/data colors consistent across light and dark mode (they're meaning, not theme); only the chrome (cards, text, borders, buttons) should flip with the theme.

## Technical requirements — read carefully, these prevent real bugs

- **Timezone math must use the `Intl` API correctly**, not manual UTC offset arithmetic (that breaks on DST). To find what a given UTC instant is in a timezone's local wall-clock time, use `Intl.DateTimeFormat` with the `timeZone` option. To convert a local wall-clock time in a given zone back to a UTC instant, read that zone's actual offset at your best-guess instant via `Intl.DateTimeFormat(..., { timeZoneName: "longOffset" })` and adjust.

- **This must be deployable as a static site with zero build-server dependency** (e.g. drag-and-drop onto Netlify/Vercel's static upload, or GitHub Pages). Two acceptable approaches:
  - **Preferred**: build it as a real app with a bundler (Vite is simplest) and ship the compiled `dist/` output. This avoids every pitfall below.
  - **Acceptable fallback**: a single self-contained `.html` file using React/ReactDOM via CDN `<script>` tags, with JSX **precompiled to plain `React.createElement()` calls ahead of time** — do not rely on in-browser Babel transformation (`<script type="text/babel">`) for production use, and do not use Babel's "automatic" JSX runtime (which injects `import` statements) since plain `<script>` tags can't resolve module imports. If using Babel to compile, explicitly set `{ runtime: "classic" }` in the React preset config.

- If shipping multiple files (`index.html`, `script.js`, `styles.css`, etc.) instead of one bundled file, make this obvious and low-risk: name things exactly `index.html` at the root (required for static hosts to serve `/` correctly), and warn that all files must be uploaded together in a single deploy action, not one at a time — uploading separately can silently overwrite the previous deploy and leave files missing.

- **Test the actual compiled output before calling it done** — don't just check that the source code looks right. Run the final built/compiled file in a headless browser or JS environment and confirm it renders real DOM content without throwing, not just that it parses as valid syntax.

## Acceptance checklist

- [ ] Adding a person with a searched-or-selected city correctly shows their strip
- [ ] Changing benchmark timezone re-aligns all strips without breaking DST-affected dates
- [ ] Scrubbing days forward/back updates the grid correctly, including across a DST boundary
- [ ] Dragging across any strip produces a live, correct per-person time summary
- [ ] Recommended windows never suggest a time that falls in someone's night
- [ ] Light/dark toggle changes every visible color with no leftover mismatched tokens
- [ ] 12h/24h toggle updates every displayed time, including inside grid cells
- [ ] The final deployed build has zero console errors on load (check devtools, not just visual appearance)
