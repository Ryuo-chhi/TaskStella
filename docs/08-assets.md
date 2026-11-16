# 08 Assets

Everything the shop sells is bundled with the app, because the app must work offline. Screens never hard-code an asset path. They look items up by id in a catalog.

## Folder layout

```
assets/
  fonts/      Nunito-Regular, Nunito-SemiBold, Nunito-Bold
  sounds/     rain, cafe, forest, lofi, ocean, white_noise, fireplace
  themes/     one folder per theme with its scene illustration
  Companions/     one folder per Companion, one image per state
  icons/      only if the chosen icon set is not built in
```

Register every folder under `assets:` in `pubspec.yaml`.

## Naming

- Lowercase with underscores: `rain.mp3`, `moonlit_lavender/scene.png`.
- Companion states use the same four names for every Companion: `idle`, `studying`, `cheering`, `sleeping`.
- Raster images need 1x, 2x and 3x versions in Flutter's density folders (`2.0x/`, `3.0x/`). Prefer SVG or vector where possible, but check the package is allowed first.

## The catalog

The catalog lists every shop item. It can be a Dart list of ShopItem objects in `lib/data/catalog/` or a bundled JSON file read at startup. Either way, each item has:

| Field     | Example                  |
| --------- | ------------------------ |
| id        | `sound_rain`             |
| name      | Rain                     |
| type      | sound                    |
| price     | 0                        |
| assetPath | `assets/sounds/rain.mp3` |
| isStarter | true                     |

Starter items (free, always owned): Rain sound, Cream Meadow theme, the starter Companion.

## Adding a new shop item

1. Add the asset files following the naming rules.
2. For a theme: define its role tokens and check every text pair against the contrast table in the spec. A theme that fails contrast does not ship.
3. Add one entry to the catalog.
4. Run the app and check the item in the Shop, the Purchase sheet and My Collection.

No screen code should need to change.

## Audio

- Sounds loop during a focus session and stop when the session ends, pauses, or the user leaves the Focus Timer.
- Short previews play in Focus Setup and the Purchase sheet. Stop any preview before starting another.
- Release the audio player in `dispose`.

## Asset status

Track which art and sounds are still missing so placeholders are never shipped by accident.

| Asset                        | Status | Owner |
| ---------------------------- | ------ | ----- |
| Starter Companion (4 states) |        |       |
| Cream Meadow scene           |        |       |
| Rain sound                   |        |       |
