# Blades, state, theme

Every row in a pane is a **blade** (`BladeApi`). Shared params on all blades: `disabled`, `hidden`, `index`. `disabled` and `hidden` are live properties on every `BladeApi`.

## Folders

```ts
const folder = pane.addFolder({ title: 'Advanced', expanded: false });
folder.addBinding(params, 'acceleration');
folder.expanded = true;
folder.title = 'Physics';
folder.on('fold', (ev) => ev.expanded);
folder.on('change', (ev) => ev.value); // bubbles from children
```

Folders nest via `folder.addFolder(...)`.

## Buttons

```ts
const btn = pane.addButton({ title: 'Reset', label: 'actions' });
btn.on('click', (ev) => ev.native); // native MouseEvent
btn.title = 'Done';
```

`title` is required; `label` is optional. Button clicks do not emit container `change`.

## Tabs

```ts
const tab = pane.addTab({
  pages: [{ title: 'Parameters' }, { title: 'Advanced' }],
});
tab.pages[0].addBinding(params, 'speed');
tab.pages[1].selected = true; // switch tab programmatically
tab.addPage({ title: 'Extra', index: 1 });
tab.removePage(2);
tab.on('select', (ev) => ev.index);
```

Each entry in `tab.pages` is a `TabPageApi` container (`title`, `selected`, plus all container methods). `tab.on('change', ...)` bubbles from every page.

## Unbound blades (`addBlade`)

Unbound blades hold their own `.value` instead of mutating a target object. `addBlade()` returns `BladeApi`; cast to the specific blade API to read/write fields. Missing required params throw `No matching view for '<json>'`.

```ts
pane.addBlade({ view: 'separator' });

const slider = pane.addBlade({
  view: 'slider',
  label: 'brightness',
  min: 0,        // required
  max: 1,        // required
  value: 0.5,    // optional, default 0
  format: (v) => v.toFixed(2),
}) as SliderBladeApi;
slider.value = 0.8;
slider.on('change', (ev) => ev.value);

const list = pane.addBlade({
  view: 'list',
  label: 'scene',
  options: [     // required; object or { text, value }[]
    { text: 'loading', value: 'LDG' },
    { text: 'menu', value: 'MNU' },
  ],
  value: 'LDG',  // required
}) as ListBladeApi<string>;
// Live update requires { text, value }[]:
list.options = [{ text: 'field', value: 'FLD' }];

const text = pane.addBlade({
  view: 'text',
  label: 'name',
  parse: (v) => String(v), // required
  value: 'sketch-01',      // required
  format: (v) => String(v),
}) as TextBladeApi<string>;
```

- Setting `.value` programmatically on an unbound blade emits `change`.
- Unbound value blades (`SliderBladeApi`, `ListBladeApi`, `TextBladeApi`) emit `TpChangeEvent` with `ev.last === true` on every change (unlike `addBinding` sliders, which set `ev.last = false` during drag).
- Move or detach blades with `container.add(api, index?)`, `container.remove(api)`, or destroy with `api.dispose()`.

## State import / export

`exportState()` returns a nested `BladeState` tree (`children` on containers; `binding: { key, value }` on bindings), not a flat preset map. `importState(state)` applies to a matching blade structure and returns `false` if the shape does not match.

```ts
const state = pane.exportState();
const ok = pane.importState(state);
```

To extract a flat key/value preset from a state tree (when keys collide across folders, set `tag` on the bindings or read `state.tag`):

```ts
import type { BladeState } from '@tweakpane/core';

function stateToPreset(state: BladeState): Record<string, unknown> {
  if (Array.isArray(state.children)) {
    return (state.children as BladeState[]).reduce(
      (acc, child) => ({ ...acc, ...stateToPreset(child) }),
      {},
    );
  }
  const binding = state.binding as { key?: string; value?: unknown } | undefined;
  if (binding && typeof binding.key === 'string' && 'value' in binding) {
    return { [binding.key]: binding.value };
  }
  return {};
}
```

## Theming (`--tp-*` CSS variables)

Override `--tp-*` custom properties on the pane's container (or `:root`). The doc-site theme names (`default`, `iceberg`, `jetblack`, `light`, `retro`, `translucent`, `vivid`) are CSS presets, not a JS API.

- **Base**: `--tp-base-background-color`, `--tp-base-shadow-color`, `--tp-base-border-radius`, `--tp-base-font-family`
- **Blade**: `--tp-blade-border-radius`, `--tp-blade-horizontal-padding`, `--tp-blade-value-width`
- **Button**: `--tp-button-background-color`, `--tp-button-background-color-hover`, `--tp-button-background-color-focus`, `--tp-button-background-color-active`, `--tp-button-foreground-color`
- **Container**: `--tp-container-background-color`, `--tp-container-background-color-hover`, `--tp-container-background-color-focus`, `--tp-container-background-color-active`, `--tp-container-foreground-color`, `--tp-container-horizontal-padding`, `--tp-container-vertical-padding`, `--tp-container-unit-spacing`, `--tp-container-unit-size`
- **Input**: `--tp-input-background-color`, `--tp-input-background-color-hover`, `--tp-input-background-color-focus`, `--tp-input-background-color-active`, `--tp-input-foreground-color`
- **Label & groove**: `--tp-label-foreground-color`, `--tp-groove-foreground-color`
- **Monitor**: `--tp-monitor-background-color`, `--tp-monitor-foreground-color`
