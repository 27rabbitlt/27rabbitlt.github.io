# Niri + DMS: Random Anime Wallpaper (No Characters)

## Background

I use **Niri** (Wayland compositor) + **DMS** as my desktop environment. I had a keybinding `Super + Shift + F10` that downloads a random anime wallpaper from the Alcy API (`https://t.alcy.cc/pc/`) and applies it via `dms ipc call wallpaper set`.

The problem: **Alcy returns beautiful anime wallpapers, but they almost always feature anime girls.** I wanted the anime art style (landscapes, cities, buildings, nature) without the characters.

## Solution

Replaced the Alcy API with the **Wallhaven API**, which supports:

- **Category filtering**: `categories=010` returns Anime-only results
- **Keyword search**: e.g. `anime scenery`, `anime city`, `anime landscape`
- **Negative keywords**: `-girl -boy -character -person -people` to exclude human characters
- **Resolution filtering**: `atleast=2560x1440` to match my monitors (2560x1600 laptop + 2560x1440 external)

No API key needed for SFW content.

## The Script

Located at `~/.local/bin/random-nature-wallpaper-dms`:

```bash
#!/bin/bash
set -euo pipefail

WALLHAVEN_API="https://wallhaven.cc/api/v1/search"

# Search terms pool - anime scenery/architecture, no characters
SEARCH_TERMS=("anime scenery" "anime landscape" "anime city" "anime buildings"
              "anime nature" "anime sky" "anime sunset" "anime rain"
              "anime forest" "anime ocean" "anime street" "anime village"
              "anime night sky" "anime countryside")

# categories=010 -> Anime only
# purity=100    -> SFW
# sorting=random
# atleast=2560x1440 -> fit both monitors
WALLHAVEN_PARAMS="categories=010&purity=100&sorting=random&atleast=2560x1440"
EXCLUDE_TAGS="-girl+-boy+-character+-person+-people"

SAVE_DIR="$HOME/Pictures/Wallpapers/api-random-download"
```

### How it works

1. Randomly picks a search term from the pool
2. Queries Wallhaven API with category/purity/resolution filters + exclusion tags
3. Parses the JSON response with `jq` to extract the image URL
4. Downloads the image, validates (size >= 20KB, MIME type is `image/*`)
5. Converts to PNG via ImageMagick
6. Applies via `dms ipc call wallpaper set <path>`
7. Async post-hooks: runs `matugen-update.sh` (Material You color generation) and `niri_set_overview_blur_dark_bg.sh`
8. Auto-cleanup: keeps only the 40 most recent wallpapers

### Dependencies

```
curl, jq, file, notify-send, ImageMagick (magick or convert)
```

## Niri Keybinding

In `~/.config/niri/dms/binds.kdl`:

```kdl
// Random anime scenery wallpaper (no characters)
Mod+Shift+F10 hotkey-overlay-title="随机下载壁纸 Random wallpaper" {
    spawn "~/.local/bin/random-nature-wallpaper-dms";
}
```

## Wallhaven API Quick Reference

```
https://wallhaven.cc/api/v1/search?categories=010&purity=100&sorting=random&atleast=2560x1440&q=anime+scenery+-girl+-boy
```

| Parameter    | Value | Meaning                          |
| ------------ | ----- | -------------------------------- |
| `categories` | `010` | Anime only (General=1, Anime=2, People=3) |
| `purity`     | `100` | SFW only                         |
| `sorting`    | `random` | Random order                  |
| `atleast`    | `2560x1440` | Minimum resolution          |
| `q`          | `anime scenery -girl ...` | Search + exclusions |
