# Hermes Desktop: Linux window regression after Electron 44

Public, narrowly scoped patch and verification evidence for [NousResearch/hermes-agent#143546](https://github.com/NousResearch/hermes-agent/issues/143546).

**Status: local mitigation, not a complete restoration of the pre-upgrade desktop layout.** After rebuilding and restarting, the reporter confirmed an opaque and usable window, but explicitly reported that the interface was still not equivalent to its pre-upgrade state. Upstream investigation of icon placement, element layout, and interaction compatibility remains necessary.

## Scope and provenance

- Upstream project: [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent).
- Affected baseline and patch base: `8edc5ea255aaf17b1f1f944bc2dcce52fce06fa8` on `main`.
- Previous local baseline: `eb3f8c7c1976ed4095877476f078fb02138e6434`.
- Runtime transition: Electron 40.10.2 to 44.9.3.
- Reproduction environment: Debian 13, KDE Plasma/KWin 6.3.6, X11, x86_64.
- Related opacity reports: [#143426](https://github.com/NousResearch/hermes-agent/issues/143426), [#143535](https://github.com/NousResearch/hermes-agent/issues/143535). Upgrade tracker: [#143277](https://github.com/NousResearch/hermes-agent/issues/143277).

This repository has an independent history. It contains only the selected patch, English documentation, and small isolated-test JSON results. It does not publish the private configuration repository, its history, credentials, conversations, user screenshots, or full diagnostic logs.

## Symptoms and measured causes

### Unexpected whole-window transparency

Hermes hides its translucency setting on Linux, yet a non-Windows default can resolve to `clear` mode with intensity 29. With the same options on the same host, Electron 40 ignored the native opacity request, whereas Electron 44 applied it. The affected window and the Electron 44 fixture carried `_NET_WM_WINDOW_OPACITY=4025479151`, approximately 0.9373 at the X11 property level.

### Native controls overlap application toolbar icons

The existing GTK decoration layout was `close,maximize,minimize:icon`. Electron 44 honored that leading button order. On a 1000px fixture viewport, its Window Controls Overlay (WCO) safe rectangle started at x=72. Hermes still placed its first 24px application tool at x=14, occupying 14-38px inside the native leading reservation.

The old hook measured only the right-side reservation and discarded an authoritative zero. Both the leading reservation and real zero values need to reach the actual toolbar layout.

The same runtime differential persisted when the isolated GTK module list was changed between empty and `colorreload-gtk-module`. The real desktop's separate GTK reload-module crash workaround and system button order were not changed.

## Patch

[Download or inspect the complete source patch](patches/20261023-linux-window-ui.patch).

The patch includes source changes and regression tests for:

1. Explicit native-opacity platform policy at the shared mapper, window constructor, runtime application, and IPC comparison boundaries. Linux stays opaque even with an old nonzero stored or renderer value. The macOS/Windows clear ramp and glass fade are retained.
2. Both WCO reservations, preserving an authoritative right-side zero, and application tools positioned after leading native controls. Existing unavailable-WCO/macOS/WSLg fallbacks are retained.

It does not include the unrelated pre-existing rich-editor IME patch. It does not change GTK settings, global theme automation, or the user's system button order.

From an appropriate Hermes source checkout:

```sh
git apply --check /path/to/20261023-linux-window-ui.patch
git apply /path/to/20261023-linux-window-ui.patch
```

To check whether it is already applied:

```sh
git apply --reverse --check /path/to/20261023-linux-window-ui.patch
```

Do not force this patch onto a different upstream revision. Check whether upstream has already fixed or changed the affected code first. Quit the old Desktop before using the official `hermes desktop` build/install/relaunch flow. A source patch does not modify an already running renderer.

## Verification

- The new behavior tests were observed failing before implementation: app tool x=14 instead of 86, missing leading WCO reservation, and a faded Linux constructor/runtime value instead of opacity 1.
- 149 targeted tests passed across opacity, titlebar/hook/panel lifecycle, wiring, and existing IME coverage. Broader suites that timed out are not represented as passing.
- Full Desktop TypeScript typecheck passed with process-local `NODE_OPTIONS=--max-old-space-size=6144` after the default 2GB Node heap ran out of memory. Changed-file lint, formatting, and diff checks passed.
- Compilation and electron-builder packaging succeeded. The final staged product's source/content freshness check returned true.
- [Native verification results and their limits](verification/README.md) document the Electron 44 isolated tests using the actual modified helper functions.
- Independent reviews found no definite logic or security error in the two scoped fixes.

These results do not establish visual parity or exhaustive real-window click/drag acceptance. The reporter's post-restart feedback is **usable, not fully restored**. Pop-out windows, drag regions, scaling, and maximize/fullscreen behavior remain appropriate upstream audit areas; they are not all separately reproduced failures.

## License

The patch contains modifications to Hermes Agent, licensed under the [MIT License](LICENSE). The upstream copyright notice is preserved. Investigation and patch preparation used Hermes Agent; the reporter authorized the mitigation and supplied the post-restart usability assessment.
