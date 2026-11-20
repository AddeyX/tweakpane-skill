# Bindings

`container.addBinding(object, key, params?)` asks registered plugins in priority order and uses the first match.

- **Input check order** (after any custom plugins): boolean → number color → color object → color string → number → string → point4d → point3d → point2d.
- **`{ readonly: true }`** bypasses inputs and checks monitor plugins (number, string, boolean).

## Input widget matrix

| Initial `object[key]` | Params | Widget |
| --- | --- | --- |
| `number` | none | text input |
| `number` | `step` only | stepped text input |
| `number` | `min` and `max` both set | slider + text (`step` optional) |
| `number` | `options` | list (overrides slider) |
| `number` | `view: 'color'` or `color: { ... }` | hex color picker (`0xRRGGBB` / `0xRRGGBBAA`) |
| `string` | none | text input |
| `string` | hex (`#rgb`, `#rrggbb`, `#rgba`, `#rrggbbaa`, `0x…`) or `rgb()`/`rgba()`/`hsl()`/`hsla()` | color picker |
| `string` | `view: 'text'` | text input (forces non-color for color-like strings) |
| `string` | `options` (add `view: 'text'` if initial value parses as color) | list |
| `boolean` | none | checkbox |
| `boolean` | `options` | list |
| `{ r, g, b }` or `{ r, g, b, a }` | optional `color: { type: 'int' \| 'float' }` | color picker (`a` enables alpha) |
| `{ x, y }` | optional axis constraints / `picker` | point2d pad + text |
| `{ x, y, z }` | optional axis constraints | point3d text |
| `{ x, y, z, w }` | optional axis constraints | point4d text |

CSS named colors (`'red'`) and modern space-separated syntax (`rgb(255 0 0)`, `oklch(...)`) do not parse as color strings and fall through to a text input.

### Shared input params

- **`options`**: `{ [label]: value }` or `{ text: string; value: T }[]`. In the object form, keys are display labels.
- **`format: (v: number) => string`**: display formatter for number inputs (typed entry still parses as a number).
- **`keyScale` / `pointerScale`**: numeric multipliers for arrow-key stepping and drag sensitivity.

### Live input APIs

Cast the returned `BindingApi` to mutate widget-specific props at runtime:

- `SliderInputBindingApi`: live `.min` and `.max` getters/setters.
- `ListInputBindingApi<T>`: live `.options` getter/setter — requires the array form `{ text: string; value: T }[]` (not the `{ [label]: value }` shorthand).

## Colors

### Number colors

Plain numbers stay numeric unless you opt in with `view: 'color'` or a `color` option:

```ts
pane.addBinding(params, 'bg', { view: 'color' }); // 0xff0055
pane.addBinding(params, 'tint', {
  view: 'color',
  color: { alpha: true }, // 0x00ffd644
});
```

### Color objects & strings

```ts
pane.addBinding(params, 'shaderColor', {
  color: { type: 'float' }, // 0–1 components; default 'int' is 0–255
});

pane.addBinding(params, 'accent', {
  picker: 'inline', // 'popup' (default) | 'inline'
  expanded: true,   // start open
});
```

- Presence of `a` on `{ r, g, b, a }` enables alpha automatically; channels write back onto the same object via `writeProperty`.
- Color strings match `StringColorInputPlugin` before `StringInputPlugin`. Pass `view: 'text'` to keep a color-like string (`'#0088ff'`) as a text field or list.

## Points (2D / 3D / 4D)

Top-level `min`, `max`, `step`, `format`, `keyScale`, `pointerScale` apply to all axes; per-axis `x`, `y`, `z`, `w` override them. Point2d also accepts `picker: 'inline' | 'popup'`, `expanded`, and `y: { inverted: true }`. Point3d and Point4d have text fields only.

```ts
pane.addBinding(params, 'offset', {
  x: { step: 20 },
  y: { min: 0, max: 100, inverted: true },
  picker: 'inline',
  expanded: true,
});
```

Edits mutate `x`/`y`/`z`/`w` on the existing object (`writeProperty`). Replacing the object with one missing an axis throws `Property '<axis>' not found`.

## Monitors (`{ readonly: true }`)

Supported value types: `number`, `string`, `boolean`.

| Param | Applies to | Behavior |
| --- | --- | --- |
| `interval` | all | Poll interval in ms (default `200`). `0` disables the timer so only `refresh()` updates the monitor. |
| `bufferSize` | all | Samples kept (default `1`; `64` when `view: 'graph'`). |
| `rows` | multiline, `bufferSize > 1`, or `view: 'graph'` | Height in rows (default `3`). Ignored by single-line monitors. |
| `multiline` | `string` | Multi-line text area (uses `rows`). |
| `view: 'graph'` | `number` | Sparkline graph; pass `min` and `max`. |
| `format` | `number` | Display formatter `(v: number) => string`. |

```ts
pane.addBinding(state, 'signal', {
  readonly: true,
  view: 'graph',
  min: -1,
  max: 1,
  interval: 200,
});
```

- Monitor `change` emits on **every** tick and every `refresh()`, even when the sampled value is unchanged (each fetch pushes a new buffer array). Monitors never write back to `object[key]`.
- Cast a graph monitor to `GraphLogMonitorBindingApi` (`@tweakpane/core`) for live `.min` and `.max`.
