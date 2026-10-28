# hots-assets

Heroes of the Storm assets that [SymbioteGG](https://github.com/symbiote-gg/symbiote-app)
downloads at build time and that
[HeroesToolChest](https://github.com/HeroesToolChest) doesn't provide. For now
that's the top-down battleground renders the replay death map draws on.

## Layout

```
battlegrounds/
    minimaps.json        index: which render belongs to which map, and where
    <map_id>.webp        one or more renders per map
```

## `minimaps.json`

Each map id points to a list of layers. A layer is a render and the map area it
covers:

```json
{
    "haunted_mines": [
        {
            "file": "haunted_mines.webp",
            "view": [5.79, 241.59, 249.5, 125.88]
        },
        {
            "file": "haunted_mines_bottom.webp",
            "view": [95.83, 113.08, 192.02, 12.79]
        }
    ]
}
```

- The map id is the one SymbioteGG stores for the battleground, in lowercase
  snake case (`cursed_hollow`, `towers_of_doom`).
- `view` is `[left, top, right, bottom]`: the map units at the render's left,
  top, right and bottom edges. In game coordinates y grows upward, so `top` is
  larger than `bottom`. The renders show scenery around the playable area, so
  the edges can fall outside it, and they can be negative.
- Layer order matters. A position goes on the first layer whose `view` contains
  it, and on the first layer if none does. Haunted Mines lists the surface first,
  then the mines underneath.

SymbioteGG checks the index when it fetches it, and refuses it if:

- a map id doesn't match `^[a-z0-9_]+$`
- a map has no layers
- a `file` isn't a plain `.webp` name (no path separators, no leading dot)
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

## Adding or recalibrating a map

1. Add the render to `battlegrounds/` as WebP, and an entry to `minimaps.json`.
2. Check the `view` against real positions: draw deaths from a few replays and
   structure positions (cores, keeps, forts) from the tracker events on top of the
   render. Move the edges until they line up.
3. Fetch with `MINIMAPS_DIR` and check the death map in the app.
4. Commit here, then pin the new commit in SymbioteGG.

## Credits

The battleground renders come from the
[Heroes of the Storm Wiki](https://heroesofthestorm.fandom.com) on Fandom,
re-encoded to WebP. The `view` calibration was made for this repository.

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

The calibration data and documentation are licensed under the [MIT License](LICENSE).
It doesn't cover the battleground renders - see [Blizzard content](#blizzard-content).
