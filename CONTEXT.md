# TV Simulator '99 - Modification Context

## Branch: fix/cross-browser-compat

## What this project is
A web app simulating a 1990s hotel TV with a Prevue Channel-style scrolling program guide. Built in 2017 by Zach Hall using jQuery 3.2.1, Moment.js, and a custom marquee polyfill. The TV guide data in `data/guide.xml` was decoded from a Prevue Guide binary file (`Curday.dat`) and contains real programming from May 3, 1999.

## What was wrong
The app only worked in Chromium browsers due to:
- Missing `px` units on CSS values throughout `scss/core/tv.scss` and `scss/channels/012.scss` (Chromium silently treats bare numbers as pixels; Firefox/Safari ignore them)
- `-webkit-calc()` used with no standard `calc()` fallback for centering the TV container
- Missing `<!DOCTYPE html>` declaration, putting Firefox into Quirks Mode
- Missing `ready()` method on the `Channel` base class, causing a JS error

## Changes made

### Cross-browser CSS fixes
- **`scss/core/tv.scss`**: Added `px` units to 7 bare numeric values, replaced `-webkit-calc()` with `calc()`
- **`scss/channels/012.scss`**: Added `px` units to 13 bare numeric values
- **`css/site.css`**: Recompiled from SCSS using `sass` (installed as dev dependency)

### HTML fixes
- **`index.html`**: Added `<!DOCTYPE html>`, removed dead Google Analytics (UA) snippet

### JS fixes
- **`js/helpers/channel.js`**: Added empty `ready()` method to base `Channel` class

### YouTube playlist support
- **`data/guide.xml`**: Added `<playlist id="PLBA04CAC15E08E188" shuffle="true" />` element alongside existing `<video>` element
- **`channels/012/script.js`**: Updated player initialization to read playlist config from XML, load playlist via `loadPlaylist()` on player ready, shuffle and jump to random video on playlist cue. Falls back to single `<video>` ID if no playlist defined. Event handlers switched to arrow functions to preserve `this` context.

### Repo hygiene
- **`.gitignore`**: Created (covers swap files, node_modules, package json files)
- **`package.json` / `package-lock.json`**: Created by `npm install --save-dev sass`

## What was NOT changed
- jQuery, Moment.js, and bettermarquee.js were left in place (they work cross-browser; old but not broken)
- `-webkit-text-stroke` left as-is (supported in all modern browsers including Firefox 49+)
- YouTube embed cookie/policy/CSP warnings in Firefox console are from Google's scripts, not fixable on our end
