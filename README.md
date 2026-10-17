# Shapr3D MCP Server

An MCP server that lets AI assistants (Claude, etc.) model real CAD parts in
code and exchange them as STEP files with CAD apps such as
[Shapr3D](https://www.shapr3d.com/).

## How it works

Models are built programmatically with
[build123d](https://build123d.readthedocs.io/) on the OpenCascade B-rep
kernel (exact solids, not meshes) and exported as STEP, STL, 3MF and more.
Existing STEP/IGES/STL/BREP/3MF files can be inspected, modified,
clash-checked and converted.

The server does **not** launch, control or screenshot Shapr3D or any other
desktop app. Shapr3D has no public API or scripting interface, and on
Windows it won't even accept STEP files from other programs, so an app
bridge only got in the way. To review a model in Shapr3D, import the
exported STEP file yourself; it comes in as editable solid bodies.

```
AI assistant ──MCP──▶ this server ──build123d/OCCT──▶ model.step ──▶ you import it in Shapr3D
                          ▲                                                    │
                          └──────────── STEP/STL you export from the app ◀─────┘
```

## Tools

| Tool | Purpose |
|---|---|
| `create_model` | Build a model from a build123d script (optionally fed by a shared `params_file`), export STEP/STL/3MF/glTF/GLB/BREP |
| `modify_model` | Import an existing file (e.g. a STEP exported from Shapr3D), edit it in code, re-export |
| `detect_clash` | Pairwise interference check between solids (assembly/keep-out verification) |
| `inspect_model` | Solids, bounding box, volume, face/edge counts of a STEP/IGES/STL/BREP/3MF file |
| `convert_model` | Read STEP / IGES / STL / BREP / 3MF, write STEP / STL / 3MF / glTF / GLB / BREP |
| `render_preview` | Rough shaded isometric PNG preview |
| `list_models` | List files in the models workspace |
| `build123d_guide` | Modeling syntax cheat sheet for the AI |

## Setup

Requires [uv](https://docs.astral.sh/uv/). Shapr3D (or another CAD app) is
optional, for reviewing the exported files.

```sh
uv sync
```

### Claude Code

A project-scoped [.mcp.json](.mcp.json) is included — opening this folder in
Claude Code picks it up automatically. To register it globally:

```sh
claude mcp add shapr3d -s user -- uv run --directory "<path-to-this-folder>" shapr3d-mcp
```

### Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`
(macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "shapr3d": {
      "command": "uv",
      "args": ["run", "--directory", "<path-to-this-folder>", "shapr3d-mcp"]
    }
  }
}
```

## Typical workflow

1. Ask the AI to model something ("design a wall bracket for a 2-inch pipe,
   4 mounting holes for M5 bolts").
2. It writes a build123d script via `create_model`, checks it with
   `inspect_model` and `render_preview`, and tells you where the STEP file is.
3. Import that STEP into Shapr3D (STEP carries mm units) and review or edit
   it there; the bodies are native, editable solids.
4. To AI-edit an existing design: export STEP from Shapr3D, then ask the AI
   to `inspect_model` / `modify_model` it.

Generated files live in `models/` (override with `SHAPR3D_MCP_WORKDIR`).

### Windows notes

The server loads its CAD and plotting libraries at startup (a few seconds)
rather than on first use. On Windows, loading those native libraries
while the MCP stdio transport is reading stdin deadlocks the process.

## Development

```sh
uv run pytest
```

The suite exercises the modeling tools directly (no MCP transport, no CAD
app needed) in a throwaway workspace.

## Limitations

- `.shapr` files are a proprietary, undocumented format — they can't be read
  or written directly. STEP is the interchange format.
- Parametric history doesn't survive the round-trip (a STEP file is the
  final solid, not the feature tree). Shapr3D's direct-modeling tools work
  on imported solids regardless.
- Importing into and exporting from Shapr3D is done by hand; the server
  never touches the app.

## License

[MIT](LICENSE)
