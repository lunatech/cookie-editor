---
title: Safari Cookie Cleaner
---

# Safari Cookie Cleaner

Safari Cookie Cleaner is a small Safari-focused tool for clearing site tracking state. This docs site has two jobs: help people contribute to the project, and help people understand how cookies and related browser storage get used in practice.

## On this page

* TOC
{:toc}

## Choose your path

### For contributors

Build the extension, understand the codebase, and work within the product's scope.

- [Contributing](contributing.html) — where to start, what kinds of changes help, and how the docs are organized
- [Development](development.html) — install dependencies, run tests, build the Safari payload, and load the app locally
- [Architecture](architecture.html) — how the extension, wrapper app, and cleanup logic fit together
- [Publishing](publishing.html) — how to ship your own signed build through TestFlight or App Review

### The Web Cookie Book

Learn how cookies work, how they get stretched into tracking systems, and why deleting a cookie is not always the end of the story.

- [The Web Cookie Book](web-cookie-book.html) — start here for the guided reading path
- [How cookies work](how-cookies-work.html) — the mechanics: `Set-Cookie`, scope, expiry, and browser inspection
- [World famous cookies](world-famous-cookies.html) — recognizable examples from large web platforms
- [Evercookie tracking](evercookie.html) — how identifiers get copied across multiple browser stores
- [Respawning cookies](respawning-cookies.html) — how Safari Cookie Cleaner detects repeated identifiers and clears site data

## Project scope

Safari Cookie Cleaner is intentionally narrow:

- Safari only — no Chrome, Firefox, or other browsers
- cleanup focused, not a general-purpose cookie viewer or editor
- no account, cloud sync, or telemetry
- no download link today; you build it yourself from source with your own Apple Developer account and signing identity

Scheduled cleanup works best on macOS. On iPhone and iPad, Safari may delay background runs until the system wakes the extension again.

## Source and issues

The source is on [GitHub](https://github.com/lunatech/safariCookieCleaner). To report a bug or suggest a change, [open an issue](https://github.com/lunatech/safariCookieCleaner/issues).

Safari Cookie Cleaner is published under [GPL-3.0](https://github.com/lunatech/safariCookieCleaner/blob/master/LICENSE).
