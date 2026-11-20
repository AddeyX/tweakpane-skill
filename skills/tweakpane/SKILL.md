---
name: tweakpane
description: >
  Source of truth for Tweakpane 4.0.5 (cocopon/tweakpane): creating panes,
  addBinding inputs and readonly monitors, folders/tabs/buttons/unbound blades,
  events, refresh() echo guards, state import/export, CSS theming, plugins, and
  v3-to-v4 migration. Use whenever adding, editing, debugging, or theming a
  Tweakpane UI, writing a Tweakpane plugin, or replacing deprecated v3 calls
  (addInput, addMonitor, addSeparator, exportPreset).
metadata:
  tweakpane: "4.0.5"
  core: "2.0.5"
---

# Tweakpane 4

Verified against [cocopon/tweakpane](https://github.com/cocopon/tweakpane) **4.0.5** (`@tweakpane/core` 2.0.5). Public docs: <https://tweakpane.github.io/docs/>.

- **ESM only** (no CommonJS build): `import { Pane } from 'tweakpane'`.
- **TypeScript**: install `@tweakpane/core` as a devDependency — `tweakpane`'s `.d.ts` imports its types, though no runtime code comes from it.

## Lifecycle

1. **Construct in the browser** once `document` exists (`new Pane()` reads `globalThis.document` in the constructor):

   ```ts
   const pane = new Pane({
     container: element, // omit to append .tp-dfwv to document.body
     title: 'Parameters', // makes the root collapsible
     expanded: true,
   });
   ```

   - Without `container`, Tweakpane appends `.tp-dfwv` (`position: absolute; top: 8px; right: 8px; width: 256px`, no `z-index`). Pass a `container` you own whenever you need fixed positioning, z-index, or framework-managed teardown.
   - Default CSS is injected once into `document.head` as `<style data-tp-style="plugin-default">`, so CSP `style-src` must permit inline styles.

2. **Populate containers.** `Pane`, `FolderApi`, and `TabPageApi` share the container API: `addBinding`, `addButton`, `addFolder`, `addTab`, `addBlade`, `add`, `remove`, `children`, `refresh`.

3. **Listen to events.** Bindings emit `change` (`TpChangeEvent`); containers bubble `change` from every child binding and unbound value blade.

   ```ts
   const binding = pane.addBinding(params, 'speed', { min: 0, max: 10 });
   binding.on('change', (ev) => {
     ev.value;      // new external value
     ev.last;       // true once a drag/edit commits
     ev.target.key; // 'speed' (on BindingApi)
   });
   ```

   - On a container listener (`pane.on('change', ev)`), `ev.target` is typed as `BladeApi` (unbound blades have no `.key`); narrow with `'key' in ev.target` or check `tag`.
   - `off(event, handler)` requires the exact function reference passed to `on`.

4. **Dispose once.** Call `pane.dispose()` (or `blade.dispose()`) exactly once. A second call throws `View has been already disposed`. `dispose()` removes the default `.tp-dfwv` wrapper, leaves a custom `container` in the DOM, and leaves injected `<style>` tags in `<head>`.

## Bindings & `refresh()` echo

`addBinding(object, key, params?)` binds to `object[key]`:

- `object` must be a non-null object (or function); otherwise throws `Value is not bindable`.
- `object[key]` must be non-null and defined at bind time — Tweakpane infers the widget from the initial value. `null`, `undefined`, or an unhandled value/params pair throws `No matching controller for '<key>'`.
- Pass the nested object itself when binding a point (`{ x, y }`) or color object (`{ r, g, b }`); Tweakpane mutates its axes/channels in place.
- **No automatic observation.** After mutating `object[key]` from outside Tweakpane, call `binding.refresh()` or `pane.refresh()`.
- **`refresh()` emits `change` on inputs.** When an input's fetched value differs, `refresh()` writes it back to `object[key]` and emits `change` (`last: true`). Guard external syncs with a `syncing` flag so your `change` handler ignores refresh echoes:

  ```ts
  let syncing = false;
  binding.on('change', (ev) => {
    if (syncing) return;
    applyChange(ev.value);
  });

  function syncFromState() {
    syncing = true;
    try {
      pane.refresh();
    } finally {
      syncing = false;
    }
  }
  ```

- **Monitors (`{ readonly: true }`)** poll `object[key]` every `interval` ms (default `200`). Set `interval: 0` to stop polling and drive updates strictly via `refresh()`.
- **Common binding params**: `label` (multiline via `\n`; set `binding.label = null` or `{ label: undefined }` to hide the label column), `disabled`, `hidden`, `index` (insertion order), `tag` (inputs only in params; live property on `BindingApi`), `view`. `label`, `disabled`, `hidden`, and `tag` are live properties on the returned API.

## v3 → v4 quick reference

| Removed v3 API | v4 replacement |
| --- | --- |
| `<script>` global `Tweakpane.Pane` | `import { Pane } from 'tweakpane'` (ESM) |
| `pane.addInput(obj, key, params)` | `pane.addBinding(obj, key, params)` |
| `pane.addMonitor(obj, key, params)` | `pane.addBinding(obj, key, { readonly: true, ...params })` |
| `pane.addSeparator()` | `pane.addBlade({ view: 'separator' })` |
| `importPreset` / `exportPreset` | `importState` / `exportState` |
| `ev.presetKey` | `ev.target.key` |
| Monitor `'update'` event | `'change'` |
| `SliderApi` (`minValue` / `maxValue`), `ListApi`, `TextApi` | `SliderBladeApi` (`min` / `max`), `ListBladeApi`, `TextBladeApi` |

## Reference files

Read the matching sibling file when needed:

- Widget selection rules, number/color/point/monitor params, live binding APIs → [bindings.md](bindings.md)
- Folders, buttons, tabs, unbound blades, state trees, `--tp-*` theme variables → [blades.md](blades.md)
- `registerPlugin`, third-party views (`fpsgraph`, `cameraring`), writing plugins → [plugins.md](plugins.md)
