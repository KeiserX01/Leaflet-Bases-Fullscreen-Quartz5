# Quartz Leaflet Bases (Fullscreen) Plugin

This is the [Quartz](https://github.com/jackyzha0/quartz) counterpart of the
[Obsidian Leaflet Bases](https://github.com/Requiae/obsidian-leaflet-bases-plugin) plugin, with an
added **fullscreen** button on every map (mirrors the fullscreen feature from the modified Obsidian
plugin). It adds a custom Leaflet map implementation for websites built using Quartz 5.

## Demo

A live demo can be found [here](https://requiae.github.io/quartz-leaflet-map-plugin-demo/) and its
source [here](https://github.com/Requiae/quartz-leaflet-map-plugin-demo).

## How to add it to your quartz

### Quartz v5

Install using the quartz plugin API:

```bash
npx quartz plugin add github:KeiserX01/Leaflet-Bases-Fullscreen-Quartz5
```

Or add the plugin to the `plugins` list in your `quartz.config.yaml`:

```yaml
plugins:
    - source: github:KeiserX01/Leaflet-Bases-Fullscreen-Quartz5
      enabled: true
      options:
          enableCopyTool: true
          enableFullscreenTool: true
```

> This plugin also requires the [bases-page](https://github.com/quartz-community/bases-page) plugin
> to function.

#### Plugin Options

| Option               | Type    | Default | Description                                                                                  |
| -------------------- | ------- | ------- | -------------------------------------------------------------------------------------------- |
| enableCopyTool       | boolean | `false` | Enables the copy tool from the Obsidian Leaflet Bases plugin.                                |
| enableFullscreenTool | boolean | `true`  | Enables the fullscreen button on the map (mirrors the Obsidian plugin, see below for detail). |

### Quartz v4

> Quartz v4 does not have a bases implementation. As such it is highly recommended (read
> 'basically required') to use the `mapName` feature if your vault has multiple maps.

Ensure you have a release tagged for Quartz v4

- Add file `leafletMapPlugin.ts` to your `quartz\plugins\transformers\`
- Append line `export { LeafletMap } from "./leafletMapPlugin"` to your
  `quartz\plugins\transformers\index.ts`
- Place line `Plugin.LeafletMap(),` to your `quartz.config.ts` in the end of
  `plugins: { transformers:` section
    - Optional: To enable the copy tool from the Obsidian plugin, add
      `Plugin.LeafletMap({ enableCopyTool: true }),` instead

## Fullscreen support

When `enableFullscreenTool` is `true` (the default) a fullscreen toggle button is rendered at the
top-right corner of every map. Clicking it expands the map to fill the whole viewport so you can
pan and zoom without scrolling around the surrounding page.

### How it works under the hood

- The button is provided by [leaflet.fullscreen](https://github.com/brunob/leaflet.fullscreen)
  (`v5.3.3`), pinned and loaded at runtime from `cdn.jsdelivr.net` (with `unpkg.com` as a
  fallback). Both the CSS and the UMD bundle are fetched.
- The plugin is initialised with
  `L.control.fullscreen({ position: "topright", pseudoFullscreen: true, ... })`. The
  `pseudoFullscreen` mode expands the map to `100vw × 100vh` via CSS, so it works in iframes and
  sandboxes where the browser's native `Element.requestFullscreen()` is blocked.
- The script only loads `leaflet.fullscreen` if `L.control.fullscreen` is not already present, so
  pages that load it once never re-download it on subsequent renders.

### Why the UMD build?

`leaflet.fullscreen@5.x` ships two JavaScript bundles:

- `Control.FullScreen.js` — ES module (uses top-level `import`). Throws `SyntaxError: import
  declarations may only appear at top level of a module` when loaded as a classic `<script>`.
- `Control.FullScreen.umd.js` — UMD bundle. Attaches itself to the global `L` namespace and is safe
  to load via `document.createElement("script")`.

This plugin uses the UMD bundle for that reason.

## How to use

### How to add a map to a note

Adding a map to a note is done by adding the following block of code to where you want the map to
appear.

````markdown
```base
views:
  - type: leaflet-map
    name: Map
    mapName: test
    image: assets/Locke.png
    height: 400
    minZoom: -1.5
    maxZoom: 2
    defaultZoom: -1.5
    zoomDelta: 0.25
    scale: "0.2"
    unit: km
```
````

| Setting     | What it does                                                                                              |
| ----------- | --------------------------------------------------------------------------------------------------------- |
| type        | The type of base, don't change this (from Obsidian bases)                                                 |
| name        | What the view is called (from Obsidian bases)                                                             |
| image       | The image the map should show. It also accepts wiki links. Can be any image supported by Quartz           |
| mapName     | Optional identifier for the map. Useful if you want to reuse a note across several maps                   |
| defaultZoom | The zoom value the map opens with. Defaults to `minZoom`                                                  |
| minZoom     | How far you can zoom out. Defaults to 0. This value is allowed to be a decimal number and can be negative |
| maxZoom     | How far you can zoom in. Defaults to 2. This value is allowed to be a decimal number and can be negative  |
| zoomDelta   | How granular zooming is. This value is allowed to be a decimal number.                                    |
| scale       | How much to scale the result of the measure tool. This value is allowed to be a decimal number            |
| unit        | The unit the measure tool uses (think km, mi, hours)                                                      |

> Technically only `type`, `name`, and `image` are required for the map view to work. However you'll
> likely end up using most of the other settings.

> In v4 only `image` is required for the map view to work.

- `mapName` can be replaced by how you'd like to call your map. You should remember this value as
  we'll need it to add markers to you map later on. The map name does not need to be unique, but be
  sure the images are either identical or compatible for marker placement.
- `minZoom`, `maxZoom` are optional boundaries on how much you'll be able to zoom the map. Depending
  on your map image you might need to fiddle with these, or remove them altogether since they are
  optional.

### How to add a marker to a map

Adding a marker to a map is done by adding the following block of code to the frontmatter of the
note you'd want the marker to link to. The example adds two markers.

```yaml
---
marker:
    - coordinates: 100, 300
      icon: lucide-tree-pine #: Lucide icon
      colour: "#039c4b"
      minZoom: 1
    - coordinates: 200, 300
      icon: mdi:alien #: Material design icon using Iconify
    - coordinates: 5, 5
      mapName: mapName
      colour: "#bdf123"
---
```

mdi:alien

| Setting      |             | What it does                                                                                                                                                                                                                                        |
| ------------ | ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Map name     | mapName     | If you want this marker to only show for a certain map, set this to the mapname of that map                                                                                                                                                         |
| Coordinates  | coordinates | Where the marker is placed on the map. Format is "latitude, longitude"                                                                                                                                                                              |
| Icon         | icon        | Which icon to use for the marker. Can be any [Lucide icon](https://lucide.dev/icons/) (prefixed by `lucide-` as obsidian does) or [Iconify icon](https://icon-sets.iconify.design/) (prefixed by `ICON-SET:` as is already included on their site). |
| Colour       | colour      | Which colour the marker will be                                                                                                                                                                                                                     |
| Minimal zoom | minZoom     | How far zoomed in the map should be before the marker becomes visible                                                                                                                                                                               |

> Technically only 'coordinates' is required for the marker to be valid. However you'll likely end
> up using most of the other settings.

## Troubleshooting

### Map renders but the fullscreen button is missing

Open the browser DevTools console. The plugin logs specific errors when something goes wrong
loading dependencies, for example:

- `[leaflet-map] Failed to load leaflet.fullscreen CSS:` — the stylesheet failed to load from both
  CDNs. Check your network, ad-blocker, or that `cdn.jsdelivr.net` and `unpkg.com` are reachable.
- `[leaflet-map] Failed to load leaflet.fullscreen JS:` — the script bundle failed to load. Same
  diagnostics as above.
- `The resource from "..." was blocked due to MIME type ("text/plain") mismatch` — the CDN URL you
  are hitting returns a 404 and serves a plain text error page. This happens when the URL points to
  a non-existent version or path. This plugin pins `leaflet.fullscreen@5.3.3`; if you change that
  version, make sure the corresponding files exist at `dist/Control.FullScreen.umd.js` and
  `dist/Control.FullScreen.css` on `cdn.jsdelivr.net` and `unpkg.com`.
- `Uncaught SyntaxError: import declarations may only appear at top level of a module` — the
  `Control.FullScreen.js` (ESM) bundle was loaded instead of the `Control.FullScreen.umd.js`
  bundle. This is a build issue; regenerate the bundle with `npm run build` from
  `quartz-leaflet-bases-plugin/` and ensure your build is up to date.

### If you use the Obsidian and your markers don't show up

Obsidian doesn't support nested YAML via the Note Properties plugin
([source](https://help.obsidian.md/properties#Not+supported)). However the
[Obsidian Leaflet Bases](https://github.com/Requiae/obsidian-leaflet-bases-plugin) plugin adds
tools to simply making and editing markers.

### If you use Quartz Syncer and your markers don't show up

Chances are that your Syncer settings don't include all frontmatter/note properties. Try enabling
the setting in Quartz Syncer to include all frontmatter/note properties.

## Development

This plugin is a TypeScript project bundled with `tsup` and built to a single ES module. The
inline runtime script is extracted from `src/scripts/leaflet-map.inline.ts` into the bundle
automatically.

### Prerequisites

- Node.js `>=22` (the `engines.node` field in `package.json`).
- npm `>=10.9.2`.

### Setup

```bash
git clone https://github.com/KeiserX01/Leaflet-Bases-Fullscreen-Quartz5.git
cd Leaflet-Bases-Fullscreen-Quartz5
npm install
```

### Common scripts

| Script           | Purpose                                                                          |
| ---------------- | -------------------------------------------------------------------------------- |
| `npm run build`  | Bundle the source (TypeScript + SCSS + inline scripts) into `dist/`.             |
| `npm run dev`    | Same as `build` but in watch mode for local iteration.                           |
| `npm run lint`   | Run ESLint over the source.                                                      |
| `npm run format` | Check Prettier formatting (use `npx prettier --write .` to apply changes).       |
| `npm run check`  | `typecheck && lint && format`. Run this before opening a PR.                     |
| `npm run typecheck` | Run the TypeScript compiler without emitting files.                          |

### Pointing your local Quartz at this checkout

In your `quartz.config.yaml`:

```yaml
plugins:
    - source: /absolute/path/to/Leaflet-Bases-Fullscreen-Quartz5
      enabled: true
      options:
          enableFullscreenTool: true
```

After any change to the source, re-run `npm run build` and restart `npx quartz build --serve`. A
hard refresh (`Ctrl+Shift+R` / `Cmd+Shift+R`) is required in the browser to bypass Quartz's hashed
asset cache.

## Credits

- [Quartz](https://github.com/jackyzha0/quartz) for which this plugin is for.
- [Lucide](https://lucide.dev/) for the API this plugin uses to load its icons.
- [Leaflet](https://github.com/Leaflet/Leaflet) which makes the whole plugin even possible.
- [Leaflet.fullscreen](https://github.com/brunob/leaflet.fullscreen) by Bruno B. for the fullscreen
  control.

## Make a new release

```shell
git tag -a 1.0.1 -m "1.0.1"
git push origin 1.0.1
```
