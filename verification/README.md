# Native verification evidence

These JSON files record isolated Electron 44.9.3 tests after the local patch, using the actual modified Hermes opacity and titlebar-layout functions. They are historical measurements, not a test runner, a general installer, or a claim of full desktop acceptance.

The fixture used a separate Xvfb display and user-data directories, a 1000px renderer viewport, a hidden native titlebar, and an overlay height of 34px. It did not attach to the user's sessions or click the user's real desktop. Existing stored intensity 29 was supplied without editing the real renderer's storage.

| GTK decoration layout | Native reservations | First application button | Result file |
| --- | --- | --- | --- |
| `close,maximize,minimize:icon` | left 72 / right 0 | x=86 to 110 | [left.json](20261023-linux-window-ui/left.json) |
| `icon:minimize,maximize,close` | left 0 / right 72 | x=14 to 38 | [right.json](20261023-linux-window-ui/right.json) |
| `close:minimize,maximize` | left 28 / right 50 | x=42 to 66 | [both.json](20261023-linux-window-ui/both.json) |

In every recorded case:

- `constructorOpacity` was 1 with the legacy nonzero setting.
- `staleRendererOpacity` was 1 after calculating the native opacity for another nonzero renderer setting.
- `restoredOpacity` was 1 after deliberately fading the isolated window and applying the production opacity mapping/reset predicate.
- The first application button stayed outside the native left/right reservations.

`pass` means that the fixture's geometry and opacity assertions passed. It does **not** mean that every toolbar icon, full application route, native hit target, drag region, scale, or platform passed end-to-end interaction testing. The fixture used production helper functions with a small test page rather than mounting the whole production Desktop.

The real user subsequently restarted the patched Desktop and confirmed an opaque, usable interface, while explicitly stating that the layout was still not equivalent to the pre-upgrade version. That unresolved visual/interaction scope is reported in [upstream issue #143546](https://github.com/NousResearch/hermes-agent/issues/143546).

Only the small result files are published here. Machine-specific fixture source imports, private configuration/history, screenshots, full logs, and local paths were intentionally omitted. Test source for the regression contracts is included in the [patch](../patches/20261023-linux-window-ui.patch).
