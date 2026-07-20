# Changelog

## v1.2.0 (2026-07-20)

Bug-fix release addressing an external user's full functional audit of all 121 tools (each invoked against a live Aseprite instance and verified by pixel read-back / filesystem inspection). Huge thanks to the reporter. All fixes were re-verified against Aseprite 1.3.x, and the three Godot resource generators were confirmed to load correctly in Godot 4.6.

### Fixed — systemic root causes
- **Decoded-JSON array guards**: `draw_brush_stroke` and `set_palette` rejected every valid array because Aseprite's `json.decode` returns *userdata*, not a Lua table, and a `type(x) ~= "table"` guard failed on it. Both tools work now. Integer params that arrive as floats are coerced with `math.floor`.
- **Unchecked file writes reported false success**: 7 text-resource writers (`.tres`/`.txt`) returned success even when `io.open` failed because the target directory did not exist. All writers now create the parent directory first and return a real error on failure (`export_for_godot`, `export_atlas_texture`, `export_9patch_for_godot`, `export_tileset_for_godot`, `generate_spriteframes_tres`, `sync_to_godot_project`, `generate_animation_player_tres`).
- **Pixel writes silently discarded outside the cel**: Aseprite bounds cels to their content, so after a partial draw, `put_pixel`/`put_pixels`/`draw_symmetry`/`clear_image(color)` dropped any write outside the cel rectangle while reporting success. Cels now grow to include the target pixels. `put_pixels` reports the real written count.

### Fixed — blockers (previously unusable tools)
- **`reverse_frames`** and **`link_cels`** crashed with a `__setters` error (index-assigning into `app.range.frames`). Both are reimplemented against the Sprite API directly; `reverse_frames` also reverses frame durations.
- **`select_ellipse`** called a non-existent `SelectEllipse` command. It now builds the selection from per-row ellipse spans and honours the `mode` parameter.
- **`set_tile`** refused to run because no tool created a tilemap cel. It now creates the tilemap cel (and grows the tileset) on demand.
- **`add_layer(type:"tilemap")`** created a normal layer; it now creates a real tilemap layer and honours `parent` / `below`.
- **`create_tilemap_layer`** leaked an orphan empty layer on every call.

### Fixed — silent data loss / wrong results
- **POSIX paths silently written to the wrong filesystem** (WSL/remote): `save_sprite`, `export_png`, `export_sprite_sheet` now return the normalized absolute path Aseprite actually used plus an `exists` flag.
- **`export_for_godot`** now writes the sheet JSON that `json_path` advertised (or omits the field), and accepts an explicit `texture_path`.
- **`export_layers`** returned paths that did not match the files written on multi-frame sprites; it now renders each frame explicitly with deterministic, sanitized filenames and returns the real list.
- **`batch_export_for_godot`** overwrote unsaved sprites (all named "Sprite") onto the same files; output names are now de-duplicated.
- **`export_gif`** hung on modal dialogs; it now uses the non-interactive `SaveFileCopyAs{ ui=false }`.
- **Selection `mode`** (add/subtract/intersect) is now applied by `select_rect` / `select_ellipse` / `select_by_color` instead of always replacing.
- **`add_frame` / `duplicate_frame`** reported the reference frame number instead of the created one.
- **Reconnect**: the plugin now tracks a last-activity timestamp (driven by server pings) and force-reconnects when the peer vanishes without a CLOSE/ERROR event.
- **Port scanning**: failed reconnects now rotate through ports 6515–6519 instead of dialing a single hard-coded port.

### Fixed — Godot resource generators (verified loading in Godot 4.6)
- **`SpriteFrames` .tres**: frame regions were emitted as an unsupported `"region"` key, so every frame showed the whole sheet. Regions now come from per-frame `AtlasTexture` sub-resources; animation names are `StringName`s and `speed` (FPS) is derived from frame durations.
- **`generate_animation_player_tres`**: emitted `_data` once per animation (only the last survived) with a wrong `load_steps` and an unused `ext_resource`. Now emits a single merged `_data` dictionary with correct `load_steps`.
- **`export_tileset_for_godot`**: produced a TileSet with no sources (zero tiles). Now emits a `TileSetAtlasSource` sub-resource with one entry per tile, and derives the `res://` texture path (accepts `texture_path`).
- **`export_9patch_for_godot`**: wrote a commented-out `.txt`; now emits a usable `.tscn` containing a configured `NinePatchRect`.

### Improved — ergonomics
- Enum fields (`color_mode`, `blend_mode`, `ani_dir`) now serialize as names (`"rgba"`, `"multiply"`, `"ping_pong"`) instead of raw integers, so read values can be written back.
- `set_image_data` implements the advertised `hex_array` format (round-trips with `get_image_data`).
- `find_unused_colors` adds `max_colors`, collapses duplicates, and reports `total_unused`.
- `generate_color_ramp` now returns exactly `steps` colors.
- `save_sprite` gains an optional `rename` for a real Save As.
- Returned paths use `app.fs.joinPath` (no more mixed `/`/`\` separators).
- New `get_server_info` tool reports extension version, tool count, Aseprite version and `apiVersion`.
- **Version metadata unified to 1.2.0** across `plugin.lua`, `extension/package.json`, and the server.

## v1.0.0 (2026-03-31)

### Initial Release

**96 tools across 13 categories:**

#### Sprite Tools (12)
- create_sprite, open_sprite, save_sprite, close_sprite
- get_sprite_info, list_open_sprites, set_active_sprite
- resize_sprite, crop_sprite, flatten_sprite
- get_sprite_screenshot, set_sprite_grid

#### Layer Tools (10)
- get_layers, add_layer, delete_layer, rename_layer
- set_layer_visibility, set_layer_opacity, set_layer_blend_mode
- reorder_layer, duplicate_layer, merge_layer_down

#### Frame Tools (8)
- get_frames, add_frame, delete_frame
- set_frame_duration, set_frame_range_duration
- set_active_frame, duplicate_frame, reverse_frames

#### Cel Tools (5)
- get_cel, set_cel_position, set_cel_opacity
- clear_cel, link_cels

#### Drawing Tools (13)
- put_pixel, put_pixels, get_pixel
- get_image_data, set_image_data
- draw_line, draw_rect, draw_ellipse
- flood_fill, draw_brush_stroke
- clear_image, replace_color, outline

#### Selection Tools (7)
- select_rect, select_ellipse, select_all, deselect
- select_by_color, get_selection, invert_selection

#### Palette Tools (10)
- get_palette, set_palette_color, set_palette
- add_palette_color, resize_palette
- load_palette, save_palette
- set_fg_color, set_bg_color, generate_palette_from_sprite

#### Tag Tools (6)
- get_tags, create_tag, delete_tag
- rename_tag, set_tag_color, set_tag_range

#### Slice Tools (5)
- get_slices, create_slice, delete_slice
- set_slice_9patch, set_slice_pivot

#### Tilemap Tools (5)
- create_tilemap_layer, get_tileset
- set_tile, get_tile, get_tilemap_info

#### Export Tools (6)
- export_png, export_sprite_sheet, export_gif
- export_tileset, export_layers, export_tags_as_sheets

#### Godot Integration Tools (8)
- export_for_godot, export_atlas_texture
- export_9patch_for_godot, export_tileset_for_godot
- get_animation_data, generate_spriteframes_tres
- sync_to_godot_project, batch_export_for_godot

#### Analysis Tools (5)
- get_color_stats, compare_frames, find_unused_colors
- get_sprite_bounds, validate_animation
