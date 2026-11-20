# Plugins

Any `view` outside [bindings.md](bindings.md) and [blades.md](blades.md) (such as `fpsgraph` or `cameraring`) comes from a plugin registered on the pane.

## Using third-party plugins

```ts
import { Pane } from 'tweakpane';
import * as EssentialsPlugin from '@tweakpane/plugin-essentials';

const pane = new Pane();
pane.registerPlugin(EssentialsPlugin);

const fps = pane.addBlade({ view: 'fpsgraph', label: 'fps' });
```

| Package | Provides |
| --- | --- |
| `@tweakpane/plugin-essentials` | `fpsgraph`, interval slider, radio/button grid, cubic bezier |
| `@tweakpane/plugin-camerakit` | `cameraring`, `camerawheel` |
| `@tweakpane/plugin-template` | Starter repo for authoring custom plugins |

Search `#tweakpane` on GitHub for community plugins. Full authoring walkthrough: <https://tweakpane.github.io/docs/plugins/dev/>.

## Bundle contract

```ts
import type { TpPluginBundle } from 'tweakpane';

const bundle: TpPluginBundle = {
  id: 'counter',
  css: '.tp-counter { display: flex; }', // injected once as <style data-tp-style="plugin-counter">
  plugins: [CounterInputPlugin],         // or single `plugin: CounterInputPlugin`
};
pane.registerPlugin(bundle);
```

Build plugins with `createPlugin(...)` from `@tweakpane/core` so `core: VERSION` is stamped automatically. A core version mismatch throws `Not compatible with  plugin '<bundleId>.<pluginId>'`.

## Plugin types & `accept()` signatures

| `type` | Entry point | `accept` signature | Return on match |
| --- | --- | --- | --- |
| `'input'` | `addBinding` (default) | `accept(value, params)` | `{ initialValue, params }` |
| `'monitor'` | `addBinding` with `readonly: true` | `accept(value, params)` | `{ initialValue, params }` |
| `'blade'` | `addBlade` | `accept(params)` | `{ params }` |

Return `null` from `accept` to pass to the next plugin.

- **Priority**: `registerPlugin` prepends (`unshift`) each plugin to its type list, so newer plugins are asked before builtins. Gate custom plugins on a unique `params.view` so they do not hijack builtin widgets.
- **Primitive passthrough** for an input plugin:

  ```ts
  binding: {
    reader: () => (value) => Number(value),
    writer: () => (target, value) => target.write(value),
  }
  ```

- **Controller & view**: `controller(args)` returns an MVC controller whose `view.element` is mounted in the row. Mutating `args.value.rawValue` emits `change` and (for inputs) writes back to the bound object.
- **Builtin input check order** (after registered plugins): `input-bool`, `input-color-number`, `input-color-object`, `input-color-string`, `input-number`, `input-string`, `input-point4d`, `input-point3d`, `input-point2d`.
