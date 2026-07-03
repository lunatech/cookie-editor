# How Cookie-Editor works on Safari

Safari doesn't let you install a WebExtension directly the way Chrome or
Firefox do. Apple requires every Safari Web Extension to be wrapped inside a
native app (macOS and/or iOS) built with Xcode and distributed through the
App Store. This document explains how that wrapping works in this repo and
how it affects the extension's code.

## The moving parts

The Safari version of Cookie-Editor is made of two things:

1. **The extension itself** — the same `interface/` and `cookie-editor.js`
   code used by the Chrome/Firefox/Edge/Opera builds, packaged with
   `manifest.safari.json`.
2. **A native wrapper app** — the Xcode project at
   [`safari/Cookie-Editor`](../safari/Cookie-Editor), which contains the app
   and extension targets for both macOS and iOS. Apple's `safari-web-extension-converter`
   tool was used to scaffold this project from the web extension source.

Xcode target layout:

```
safari/Cookie-Editor/
├── macOS (App)/           # Wrapper app shown in Applications / Launchpad
├── macOS (Extension)/     # Safari App Extension target for macOS
├── iOS (App)/              # Wrapper app for iPhone/iPad
├── iOS (Extension)/        # Safari App Extension target for iOS
└── Shared (App)/           # Code/UI shared by the macOS and iOS wrapper apps
    ├── ViewController.swift
    └── Resources/ (Main.html, Script.js, Style.css)
```

## The manifest

`manifest.safari.json` is a Manifest V3 file, same shape as the other
browsers', with one Safari-specific addition:

```json
"browser_specific_settings": {
  "safari": {
    "strict_min_version": "15.4"
  }
}
```

This declares the minimum Safari version (15.4) that supports the
WebExtensions APIs Cookie-Editor relies on (`cookies`, `tabs`, `storage`,
and optional host permissions for `<all_urls>`).

## Building the Safari version

The regular `grunt` task builds Chrome/Firefox/Edge/Opera as zip files. Safari
is different — see `Gruntfile.js`'s `build-safari` task:

```
json-replace:safari → clean:safari → copy:safari → replace:safari
```

This copies the shared extension source into `build/safari/` and stamps the
manifest version, but it deliberately **skips** the `removelogging` step so
`console.log` output stays available for debugging in Safari's Web Inspector.

That `build/safari/` output isn't installable on its own — per the README,
"Safari needs to be built in Xcode." The Xcode project embeds the built web
extension as a Safari App Extension resource and produces the native app
bundle that actually gets installed/distributed.

## Runtime browser detection

Because the extension code is shared across browsers, it needs to know at
runtime that it's running in Safari:

- `interface/lib/env.js` contains a placeholder, `Env.browserName = '@@browser_name'`,
  that Grunt's `replace:safari` step rewrites to `'safari'` during the build.
- `interface/lib/browserDetector.js` exposes `isSafari()`, which just checks
  `Env.browserName === Browsers.Safari` (see `interface/lib/browsers.js`).

Code elsewhere in the extension calls `browserDetector.isSafari()` when it
needs to branch on Safari-specific behavior.

## Safari-specific accommodations in the extension code

Safari's WebExtension implementation has a couple of gaps compared to
Chrome/Firefox, which `interface/lib/permissionHandler.js` works around:

- **No `permissions` API in the DevTools panel.** Safari's DevTools context
  can't access `browser.permissions`, so `checkPermissions()` treats a
  missing `permissions` API as "permission already granted" instead of
  failing:

  ```js
  // If we don't have access to the permission API, assume we have
  // access. Safari devtools can't access the API.
  if (typeof this.browserDetector.getApi().permissions === 'undefined') {
    return true;
  }
  ```

- **`safari-web-extension:` pages can't be granted permissions.** The
  extension's own internal pages use this URL scheme, so it's included in
  `impossibleUrls` alongside `about:`, `moz-extension:`, `chrome:`,
  `chrome-extension:`, and `edge:` — pages Cookie-Editor should never try to
  request cookie access for.

## The native wrapper app

Safari App Extensions can't be turned on directly from the App Store — the
user has to open the companion app once and enable the extension from
Safari's settings. `ViewController.swift` and the shared
`Main.html`/`Script.js` implement this "quick start" screen:

- On launch, it loads `Main.html` in a `WKWebView`.
- On macOS, it calls `SFSafariExtensionManager.getStateOfSafariExtension(withIdentifier:)`
  to check whether the extension (bundle ID `ca.cgagnier.cookie-editor.Extension`)
  is already enabled, and reflects that state (on/off/unknown) in the UI.
- A button in that UI ("Open Safari Settings…") sends a `postMessage` back to
  native code via `webkit.messageHandlers.controller`, which calls
  `SFSafariApplication.showPreferencesForExtension(withIdentifier:)` to jump
  straight to the Extensions pane in Safari's settings, then quits the
  wrapper app.
- On iOS there's no equivalent settings deep-link API, so the app just shows
  instructions for enabling the extension manually in Settings.

## Native messaging handler

`SafariWebExtensionHandler.swift` implements the `NSExtensionRequestHandling`
protocol, which is the bridge Safari provides for
`browser.runtime.sendNativeMessage()` calls from the extension to native
Swift code. In this project it's currently just a stub that logs and echoes
the message back — Cookie-Editor doesn't need native messaging for its
cookie-editing functionality, since the standard `cookies`/`tabs`/`storage`
WebExtension APIs are enough. The handler exists because Xcode's Safari Web
Extension template scaffolds it by default.

## Distribution

Unlike the other browsers (published directly to each browser's extension
store), the Safari build is distributed as a single universal app through
the **Mac App Store** and **iOS App Store**, listed as one app that bundles
both the macOS and iOS extension targets. See the "Install on Safari"
section of the [README](../README.md) for the store links.
