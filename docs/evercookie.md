---
title: Evercookie — Zombie Tracking
---

[Docs home](index.md) · [How cookies work](how-cookies-work.md)

# Evercookie — zombie tracking

A regular cookie can be deleted. An evercookie comes back.

Evercookie is an open-source JavaScript library created by security researcher [Samy Kamkar](https://samy.pl/) in 2010 to demonstrate how a tracking identifier can survive deletion by storing copies of itself across every storage mechanism a browser exposes. Delete it from one place and it respawns from another.

The technique is also called a **zombie cookie** or **supercookie**.

## How it works

When you visit a site using evercookie, it writes the same unique identifier to as many as 17 separate storage locations simultaneously:

| Storage mechanism | Notes |
|---|---|
| HTTP cookies | The standard target of "clear cookies" |
| `localStorage` / `sessionStorage` | HTML5 key-value stores, separate from cookies |
| `IndexedDB` | Structured browser database |
| Flash Local Shared Objects (LSOs) | Adobe Flash cookies, stored outside browser profile |
| `ETags` | HTTP caching headers repurposed as identifiers |
| Web cache | Resources cached with a unique URL encoding an ID |
| `window.name` | Persists across page navigations in the same tab |
| CSS `:visited` history | Reads which URLs the browser has visited |
| PNG pixel via `<canvas>` | Encodes ID in image data stored in cache |
| Silverlight isolated storage | Microsoft plugin storage, now obsolete |
| HTTP Auth cache | Browser credential cache |
| Browser favicon cache | Per-site icon cache, rarely cleared by privacy tools |

If you clear cookies but leave localStorage alone, the next visit reads the ID from localStorage and writes it back into cookies. If you clear both, the ETag cache restores it. If you clear everything the browser offers, a Flash cookie sitting outside the browser's profile directory may still hold the ID.

## Why deleting cookies is not enough

This is the exact problem evercookie illustrates. Safari Cookie Cleaner removes cookies, which eliminates the most common tracking store. But if a site has written an evercookie, the ID can survive in localStorage, IndexedDB, ETags, or cached resources and respawn into a fresh cookie on the next visit.

To fully clear an evercookie you need to:

1. Clear cookies
2. Clear site data (localStorage, sessionStorage, IndexedDB, cache)
3. Clear the browser cache
4. Remove any Flash or Silverlight plugins (both are now obsolete and unsupported in Safari)

In Safari: **Settings → Safari → Advanced → Website Data → Remove All Website Data** removes cookies and site storage together. **History → Clear History** clears the cache. For most modern sites this is sufficient; Flash is dead and ETags are the remaining edge case.

## Sites caught using evercookie-style tracking

### KISSmetrics — Hulu, Spotify (2011)

In July 2011, researchers at UC Berkeley crawled the top 100 US websites and found KISSmetrics, a marketing analytics provider, using HTTP cookies, Flash cookies, and ETags in combination to respawn deleted identifiers. Hulu and Spotify were among the sites running KISSmetrics scripts.

Both Hulu and Spotify suspended KISSmetrics the same day the report was published. Two class-action lawsuits followed. In October 2012 KISSmetrics settled for over $500,000 and pledged to stop using the technique. Hulu separately settled an FTC action for $2.4 million.

### Microsoft (2011)

Two supercookie mechanisms were found on Microsoft properties in 2011, including cookie syncing that respawned `MUID` tracking cookies. Microsoft disabled the code after media coverage.

### Verizon UIDH / Turn Inc. (2012–2016)

Verizon Wireless injected a unique identifier header (UIDH) into every HTTP request its mobile customers made, at the network level, without their knowledge. The header persisted regardless of what users cleared in the browser because it was added by Verizon's infrastructure, not the browser.

Ad company Turn exploited the UIDH to resurrect deleted tracking cookies: if it detected the Verizon header, it could re-associate a cleared browser with the old identifier. The Electronic Frontier Foundation and ProPublica exposed the practice in 2015. In 2016 Verizon paid the FCC $1.35 million to settle. Turn faced separate FTC charges.

### Chinese and Russian top sites (2014)

A Princeton University study crawling the top 100,000 Alexa sites found evercookie-style mechanisms in use on several major Chinese platforms including `weibo.com`, `sina.com.cn`, `hao123.com`, `sohu.com`, `youku.com`, and others, as well as `yandex.ru`. The study also documented the first known commercial use of IndexedDB as a tracking store, found on `weibo.com`.

### NSA / Tor tracking (2013)

An internal NSA presentation revealed by Edward Snowden in 2013 described using evercookie to track users of the Tor anonymity network. The Tor Project responded that Tor Browser and the Tails OS provide strong protections against evercookie because they wipe all storage on exit.

### Favicon cache tracking (2021)

Researchers demonstrated a technique where a site encodes a unique identifier in the browser's favicon cache by loading a specific sequence of favicons. The favicon cache is stored separately from cookies and site data and is not cleared by most privacy tools, including "Clear History" in Safari. The technique works even in private browsing in some browsers.

## What Safari does to help

Safari's Intelligent Tracking Prevention (ITP) limits several evercookie vectors:

- Third-party cookies are blocked by default
- `localStorage` and `IndexedDB` for cross-site trackers are partitioned or purged after 7 days of no interaction
- ETags are partitioned per-site so they cannot be used for cross-site tracking
- Flash and Silverlight are not supported at all

ITP does not protect against first-party evercookies, where the site itself (not a third-party script) is doing the respawning.

## Sources

- Samy Kamkar, [evercookie project](https://github.com/samyk/evercookie) (2010)
- Ayenson et al., UC Berkeley, [Flash Cookies and Privacy II](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=1898390) (2011)
- Englehardt & Narayanan, Princeton, [Online Tracking: A 1-million-site Measurement and Analysis](https://www.cs.princeton.edu/~arvindn/publications/OpenWPM_1_million_site_tracking_measurement.pdf) (2016)
- ProPublica, [Zombie Cookie: The Tracking Cookie That You Can't Kill](https://www.propublica.org/article/zombie-cookie-the-tracking-cookie-that-you-cant-kill) (2015)
- Wikipedia, [Evercookie](https://en.wikipedia.org/wiki/Evercookie)
