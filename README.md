# Aseprite MCP Pro

AI-powered pixel art creation with Aseprite via the Model Context Protocol.

**129 tools** for complete control over sprites, layers, animations, palettes, and export — with built-in Godot Engine integration.

## What is Aseprite MCP Pro?

Aseprite MCP Pro connects AI assistants (Claude, etc.) directly to your running Aseprite editor. The AI can create sprites, draw pixel art, manage animations, manipulate palettes, and export for game engines — all through natural language.

### Tool Categories

| Category | Tools | Description |
|----------|-------|-------------|
| Sprite | 12 | Create, open, save, resize, crop, screenshot |
| Layer | 10 | Full layer stack control with groups and blend modes |
| Frame | 8 | Animation frame management and duration |
| Cel | 5 | Cel positioning, opacity, linking |
| Drawing | 13 | Pixels, lines, rectangles, ellipses, flood fill, brush strokes |
| Selection | 7 | Rectangle, ellipse, by-color, invert |
| Palette | 10 | Full palette manipulation, auto-generate |
| Tag | 6 | Animation tags with direction and repeat |
| Slice | 5 | Named regions, 9-patch, pivot points |
| Tilemap | 5 | Tilemap layers and tileset management |
| Export | 6 | PNG, sprite sheets, GIF, per-layer, per-tag |
| Godot | 8 | SpriteFrames .tres, AtlasTexture, TileSet generation |
| Analysis | 5 | Color stats, frame diff, animation validation |

## Architecture

```
AI Client (Claude Code, etc.)
    ↕ stdio / MCP Protocol
Node.js MCP Server
    ↕ WebSocket
Aseprite Lua Extension (this repo)
    ↕ Aseprite Scripting API
Aseprite Editor
```

## Installation

### Free Version (Extension Only)

This repository contains the Aseprite Lua extension. To use it, you need the MCP server component.

1. Download or clone this repository
2. Copy the `extension/` folder to your Aseprite extensions directory:
   - **Windows**: `%APPDATA%\Aseprite\extensions\aseprite-mcp-pro\`
   - **macOS**: `~/Library/Application Support/Aseprite/extensions/aseprite-mcp-pro/`
   - **Linux**: `~/.config/aseprite/extensions/aseprite-mcp-pro/`
3. Restart Aseprite

### Full Version (Extension + Server)

Get the complete package with the MCP server at:

**[Buy Me a Coffee](https://buymeacoffee.com/)** (coming soon)

## Godot Integration

Works alongside [Godot MCP Pro](https://godot-mcp.abyo.net/) for a complete AI-driven game art pipeline:

1. Create & animate sprites in Aseprite via AI
2. Auto-export sprite sheets + `.tres` SpriteFrames resources
3. Import directly into your Godot project

## Related Projects

- [Godot MCP Pro](https://godot-mcp.abyo.net/) — AI-powered Godot development (190 tools)
- [Tiled MCP Pro](https://tiled-mcp-pro.abyo.net/) — AI-powered tile map editing (122 tools)

## License

MIT
