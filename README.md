# HOTS assets

Heroes of the Storm assets that [SymbioteGG](https://github.com/symbiote-gg/symbiote-app)
downloads at build time and that
[HeroesToolChest](https://github.com/HeroesToolChest) doesn't provide. For now
that's the battleground minimaps the replay death map draws on.

## Layout

```
battlegrounds/
    minimaps.json        index: which minimap belongs to which map, and where
    <map_id>.svg         one or more minimaps per map
```

## `minimaps.json`

Each map id points to a list of layers. A layer is a minimap and the map area it
covers:

```json
{
  "haunted_mines": [
    {
      "file": "haunted_mines.svg",
      "view": [17.5, 237.5, 239.5, 130.5]
    },
    {
      "file": "haunted_mines_bottom.svg",
      "view": [99, 110, 188, 16.5]
    }
  ]
}
```

- The map id is the one SymbioteGG stores for the battleground, in lowercase
  snake case (`cursed_hollow`, `towers_of_doom`).
- `view` is `[left, top, right, bottom]`: the map units at the left, top, right
  and bottom edges of the SVG's `viewBox`. In game coordinates y grows upward,
  so `top` is larger than `bottom`.
- Layer order matters. A position goes on the first layer whose `view` contains
  it, and on the first layer if none does. Haunted Mines lists the surface first,
  then the mines underneath.

A layer's file is an `.svg` traced with SymbioteGG's
`scripts/assets/trace-minimap.mjs` from the map's own minimap image. It has a
`viewBox` starting at `0 0` and one path per flat colour of the minimap. The
colours and the jungle's faded edge are in the file, so it looks right opened on
its own; SymbioteGG reads only the paths and paints them itself:

- `<path id="ground">`: the map's silhouette, always there
- `<path id="jungle">`: the lighter ground off the lanes
- `<path id="walls">`: impassable terrain
- `<path id="bright">`: what the minimap paints in its lighter tone, the ornaments
  by the cores and on some maps the walls

SymbioteGG checks the index when it fetches it, and refuses it if:

- a map id doesn't match `^[a-z0-9_]+$`
- a map has no layers
- a `file` isn't a plain `.svg` name (no path separators, no leading
  dot)
- an `.svg` has no `viewBox` or no `ground` path
- a `view` isn't four finite numbers with `left != right` and `top != bottom`

## Using it from SymbioteGG

Pin a commit and the folder in SymbioteGG's `scripts/asset-pins.json`:

```json
"minimaps": {
    "repo": "symbiote-gg/hots-assets",
    "commit": "<sha>",
    "path": "battlegrounds"
}
```

To try changes before committing them, point the fetcher at a local folder
instead:

```sh
MINIMAPS_DIR=../hots-assets/battlegrounds pnpm assets
```

## Credits

The minimaps are traced from the images Heroes of the Storm ships with each
battleground. The tracing and the `view` placement were made for this repository.

## Blizzard content

This is a fan-made project and is not affiliated with, endorsed by, or officially
connected to Blizzard Entertainment or Heroes of the Storm. Map names, artwork and
other game assets are the property of Blizzard Entertainment. This repository
grants no rights in anything belonging to Blizzard Entertainment.

Heroes of the Storm™ ©2014 Blizzard Entertainment, Inc. All rights reserved.
Heroes of the Storm is a trademark of Blizzard Entertainment, Inc.

Blizzard Entertainment is a trademark or registered trademark of Blizzard
Entertainment, Inc. in the U.S. and/or other countries. All rights reserved.

## License

The placement data and documentation are licensed under the [MIT License](LICENSE).
It doesn't cover the battleground minimaps - see [Blizzard content](#blizzard-content).
