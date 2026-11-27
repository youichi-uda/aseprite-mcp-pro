# Changelog

## v1.3.0 (2026-11-27)

Aseprite 1.3.18 integration release: Godot auto-sync on save, a native "Godot SpriteFrames" export format, an eyedropper tool and tile-rect tools — plus fixes to the Godot exporters and the end of the Console window popping up on every connect. **129 tools** (66 in `--lite` mode). Tested live against Aseprite 1.3.18.6; generated resources verified loading in Godot 4.6.2. Every new Aseprite API is feature-detected, so older Aseprite 1.3.x keeps working.

### Added
- **Godot auto-sync on save** — `enable_godot_autosync`, `disable_godot_autosync`, `get_godot_autosync_status`. After every save of a `.aseprite` file (Ctrl+S in Aseprite, or `save_sprite`), the extension re-exports the sheet PNG + SpriteFrames `.tres` into `<godot_project_path>/<target_dir>` with the same options as `sync_to_godot_project`. The configuration is persisted per file path in the extension preferences and re-attaches when the file is reopened (also after restarting Aseprite). The export never re-saves the source sprite. **Requires Aseprite 1.3.18.2+** (API 41, sprite `save` event); older versions get a clear error.
- **"Godot SpriteFrames" file format** — File › Export As… `name.spriteframes` (or `aseprite -b x.aseprite --save-as x.spriteframes`) writes `name.tres` + `name.png` next to it, plus a small JSON manifest in the `.spriteframes` file. The texture path is `res://…` when a `project.godot` is found above the folder, else relative to the `.tres` (Godot resolves it). Opening a `.spriteframes` file reports "import not supported". A dedicated extension is used because Aseprite matches only the last extension (so `.spriteframes.tres` cannot work) and claiming `.tres` would hijack a generic Godot resource extension. **Requires Aseprite 1.3.18+** (API 40, `plugin:newFileFormat`); not registered on older versions.
- **`pick_color`** — samples the composited sprite (all visible layers) at x,y on any frame, returning `#RRGGBBAA`, and can set the result as FG or BG (left untouched by default). Uses the native `app.useTool{ tool="eyedropper" }` on **Aseprite 1.3.18+**, and an equivalent Lua composite on older versions (`method: "composite"` forces it).
- **`fill_tiles`** — fills a tile rectangle with a tile index (optional x/y/diagonal flip flags); creates or grows the tilemap cel (also left/up) and appends empty tiles if the index does not exist yet.
- **`copy_tiles`** — copies a tile rectangle (indices + flip flags) to another tilemap layer, frame or position. Same tileset: copied as-is; different tilesets: each tile is remapped to an identical tile image in the destination, missing ones are appended (`add_missing_tiles`, default true) and reported. Overlapping areas are safe; one undo step. Works on Aseprite 1.3+ (it does not use the clipboard, whose tilemap copy/paste between layers Aseprite fixed in 1.3.18.4–1.3.18.6).
- `get_server_info` now also returns `recent_log`, `connected`, `verbose_logging`, `autosync_entries` and a `features` map (which 1.3.18 integrations are active on this Aseprite).
- Godot exporters gain `columns` / `rows` (and `get_animation_data` / `generate_spriteframes_tres` / `sync_to_godot_project` / `batch_export_for_godot` gain `scale` / `trim`), all sharing one code path.

### Fixed
- **Console window popping up over the canvas**: the extension `print()`ed on every connect attempt, reconnect and stale cycle, which opens Aseprite's Console for every user. All routine messages now go to a silent 50-line ring buffer (Help › MCP Pro: Status › Show Log, or `get_server_info.recent_log`). A new **Verbose** checkbox in Help › MCP Pro: Settings echoes them to the Console when wanted.
- **`export_for_godot` ignored `scale` and `trim`** (read but never applied). Scale now resizes every frame (nearest neighbour) and the `.tres` regions are in scaled pixels; trim crops each frame to its opaque bounds and stores the offset in the AtlasTexture `margin`, so trimmed animations keep their exact placement in Godot (frames still report the full size).
- **`sheet_type: "rows"` / `"columns"` silently fell back to horizontal** in `export_for_godot` (and `sync_to_godot_project` / `batch_export_for_godot` / `generate_spriteframes_tres` disagreed with each other about the layout). Both are implemented: `rows` = row-major grid with `columns` cells per row, `columns` = column-major grid with `rows` cells per column (default ceil(√frames)). Unknown values are rejected instead of falling back.
- **Tag direction was lost in the SpriteFrames `.tres`** (Godot has no playback direction): reverse tags now list their frames in reverse; ping-pong tags append the reversed inner frames (a..b, b-1..a+1), ping-pong-reverse the mirror of that.
- `batch_export_for_godot` / `export_for_godot` referenced the texture as `res://<name>.png` even inside a project subfolder; `batch_export_for_godot` now uses `res://<target_dir>/<name>.png`, and `export_for_godot` derives the `res://` path from the enclosing Godot project when `texture_path` is omitted.
- `create_tilemap_layer` (and `add_layer(type:"tilemap")`) inherited the tile size of the active tilemap layer instead of the requested one.
- Animation names containing `"` or `\` no longer break the generated `.tres`.

### Fixed — newer Aseprite permission system (1.3.19 beta, aseprite#5965)
- Tested on the Aseprite beta 1.3.18.6-195 (per-extension `permissions.json` prompts, curl-based WebSocket): all features pass once the permissions are granted.
- Before this fix, denying the network prompt raised an error inside `init()`, so the MCP Pro menu commands were never registered. The first connect now runs after `init()`, and WebSocket creation is guarded. After a denial, auto-reconnect pauses (no re-prompt every 3 s, no port hunting) until Help › MCP Pro: Reconnect; the Status dialog shows the state.
- No second connect is started while a permission prompt is still open (timers keep ticking under modal dialogs).
- CLI/batch runs (e.g. `--save-as x.spriteframes`) no longer open a socket that could replace the GUI instance's MCP connection.
- `open_sprite` on a missing file returns "File not found" instead of making Aseprite pop its Console.

### Changed
- The startup log line counts registered tools instead of printing a stale hard-coded number ("109 tools"); README / INSTALL / package descriptions now state 129 tools (66 lite).
- `save_sprite` reports the auto-sync result (`autosync`) when the saved file has an auto-sync configuration, and returns an error instead of a Lua exception when the save fails.
- MCP Pro: Settings only reconnects when the port actually changed.
- Version 1.3.0 across `plugin.lua`, `extension/package.json` and the server. The extension manifest keeps `api: 27`, so Aseprite 1.3.x without the 1.3.18 APIs still loads it.

### Known Aseprite behaviour
- After a custom-format save Aseprite marks the document as saved, even for File › Export. The extension restores the "unsaved changes" mark right after a `.spriteframes` export (a tiny undoable no-op), so closing Aseprite still prompts to save the `.aseprite`.

## v1.2.2 (2026-07-21)

Self-review pass over v1.2.1, plus a retry of the one deferred item — which turned out to already work.

### Selection clipping: fully covered after all
- **`draw_rect` / `draw_ellipse` / `flood_fill` / `draw_line` / `draw_brush_stroke` natively honor the active selection** — Aseprite's `app.useTool` pipeline clips to the mask (verified in-engine with per-pixel checks: inside painted, outside preserved; flood fill seeded outside a selection does not leak in, and vice versa). The v1.2.1 changelog wrongly stated these "still draw unclipped"; that was a mis-diagnosis caused by Aseprite trimming the cel to the drawn region and out-of-bounds reads returning garbage during testing. Combined with v1.2.1's `replace_color` / `clear_image` support, the whole drawing group now respects the selection.

### Fixed
- **`get_autotile_rules` returned `rules: null`**: the decoded rules (Aseprite JSON userdata) were embedded directly in the response, and re-encoding nested userdata silently yields null. Rules are rebuilt as plain tables (`index`, `col`, `row`, `bitmask`/`edge_mask`).
- **Port hunting never actually rotated on normal networking**: a dead port fails instantly with ECONNREFUSED, and the reconnect loop retried the same port forever (the rotation only fired via the 20 s stale timer, i.e. effectively only under WSL mirrored networking). Failed connects now rotate to the next port until the first session is established; after that, the port that carried the session is preferred across a few failures before rotating (and the scan cycles back to it), preserving v1.2.1's anti-drift behaviour.

## v1.2.1 (2026-07-21)

Follow-up to the v1.2.0 audit: the reporter re-tested all 121 tools against v1.2.0 and flagged a small set of remaining issues. Thanks again.

### Fixed — connection stability
- **Stale-socket CLOSE tore down the live connection** (the "MCP connection keeps dropping" flap on a single client): each socket is now tagged with a generation number and ignores its own events once superseded, and CLOSE/ERROR close *that* socket rather than the shared handle. This removes the self-sustaining reconnect/close storm.
- **Port drift under WSL mirrored networking**: once a connection is established the extension stays on that port instead of rotating away when a reconnect is needed (a dead-port connect hangs rather than erroring under WSL mirrored mode, which previously caused it to drift off a live server). It only rotates while still hunting for a listener, or after several consecutive stale cycles. An out-of-range inherited port preference is clamped back into 6515–6519.
- **Server: `ASEPRITE_MCP_PORT` now pins the port** (binds exactly that port with no scan), so a user can fix the port at both ends — the reliable single-owner setup. Without it, the free-port scan is unchanged. Note: running multiple MCP servers against one Aseprite still needs a single-owner broker in front of one socket; that architectural change is not included.

### Fixed — remaining false success / correctness
- **`export_tags_as_sheets(json: true)`** now writes a per-tag sheet JSON (frame regions, durations, loop, direction) — the flag was read but never used.
- **`export_png` / `save_sprite`** now include a `path_rewritten` flag when a POSIX-style absolute path is silently remapped to a Windows drive path, so a client no longer has to diff two fields to notice.
- **`sync_to_godot_project`** and **`export_tags_as_sheets`** return single-separator filesystem paths (`app.fs.joinPath` / `normalizePath`); the `res://` path keeps forward slashes.
- **`get_image_data(hex_array)`** now emits 8-digit `#RRGGBBAA` uniformly, so a `get → set` round-trip is byte-identical regardless of opacity.

### Added
- **Stable sprite id**: `list_open_sprites` returns a per-session `id`, and `set_active_sprite` / `copy_between_sprites` accept it (unambiguous when several unsaved sprites share the name "Sprite"). Filenames still work.
- **`get_autotile_rules`**: reads back the metadata stored by `define_autotile_rules` (previously write-only).
- **Selection-aware editing**: `replace_color` and `clear_image` now restrict their effect to the active selection when one exists. (Shape/fill tools — `draw_rect`/`draw_ellipse`/`flood_fill` — still draw unclipped; that remains future work.)

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
