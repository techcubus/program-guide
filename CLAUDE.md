# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

**TV Simulator '99** — a static web app simulating a 1990s hotel television. It plays a YouTube video/playlist alongside a Prevue Channel-style scrolling program guide. Guide data in `data/guide.xml` was decoded from a real `Curday.dat` Prevue Guide binary (May 3, 1999).

No backend. No bundler. Scripts are loaded in dependency order directly in `index.html`.

## CSS compilation

SCSS is the source of truth; `css/site.css` is compiled output. After editing any `.scss` file, recompile:

```sh
npx sass scss/site.scss css/site.css
# or watch mode:
npx sass --watch scss/site.scss css/site.css
```

There are no tests or linting configured.

## Serving the app

The YouTube IFrame API requires an HTTP origin (not `file://`). Use any static file server from the repo root, e.g.:

```sh
python3 -m http.server 8080
```

## Architecture

### Script load order (index.html)

Libraries → helpers → channel implementations → `tv.js` → inline `new TV(); tv.startUp()`.

All classes are globals (no modules). The load order in `index.html` is the dependency order and must be maintained.

### Startup sequence

1. `TV.startUp()` waits for `$(document).ready`, then calls `YouTubeApi.loadYouTubeAPI()`.
2. When the YouTube IFrame API loads, `onYouTubeIframeAPIReady()` fires the custom `youtubeReady` DOM event.
3. On `youtubeReady`, `TV` fetches `data/guide.xml` via AJAX and calls `showChannel()` on the channel marked `watchable` and `default` in the XML.

### Channel lifecycle

`TV.showChannel(number)`:
1. Calls `teardown()` on the current channel (clears all timeouts/intervals, stops YT video, empties the DOM container).
2. Loads `channels/NNN/layout.html` into `.current-channel` via jQuery `.load()`.
3. Instantiates `new Channel{number}(container, guideData)`.
4. Calls `channel.show()` — sets up the YouTube player, DOM references, and `setInterval`/`setTimeout` timers.
5. After the fade-in animation completes, calls `channel.ready()`.

`Channel` base class (`js/helpers/channel.js`) manages teardown. Subclasses must store all timers in `this.timeouts` and `this.intervals` so `teardown()` can clear them automatically.

### Timeslot system (Channel12)

Timeslot 0 = 2:30 AM; each slot = 30 minutes. The guide advances 5 minutes early (fires at :25/:55) like the real Prevue hardware.

Formula: `Math.floor((hours * 60 + minutes + 5) / 30) - 5`

`getFirstListing(listings, timeslot)` walks backward from the target slot to find the most recent listing that covers it (handles shows longer than 30 min).

### guide.xml structure

```xml
<guide>
  <videos>
    <video id="YT_VIDEO_ID" />
    <playlist id="YT_PLAYLIST_ID" shuffle="true" />   <!-- optional; takes precedence -->
  </videos>
  <ads>
    <ad duration="30"><!-- HTML content --></ad>
  </ads>
  <channel number="12" name="ABC" watchable="" default="">
    <notice><!-- optional banner row --></notice>
    <listing timeslot="35" type="1" rating="TV-PG">Show Title</listing>
  </channel>
</guide>
```

`watchable` and `default` are presence-only attributes. `noticeonly` on a `<channel>` suppresses the channel row.

### Adding a new channel

1. Create `channels/NNN/layout.html` (HTML fragment — no `<html>`/`<body>`) and `channels/NNN/script.js` defining `class ChannelNNN extends Channel`.
2. Register in `TV.classes` (`js/core/tv.js`): `this.classes = { Channel12, ChannelNNN }`.
3. Add `<script src="channels/NNN/script.js">` in `index.html` before `tv.js`.
4. Add a `<channel number="NNN" watchable="" ...>` entry in `data/guide.xml`.
