# Paradise Island

A cozy, AFK-friendly island ecosystem game for Steam. The player's name + birth year
are hashed into a deterministic seed that generates a unique undiscovered Pacific
island. The island starts as bare rock/sand and grows into a lush ecosystem the
player nurtures: planting limited seeds, watering, clearing tide-borne trash, and
keeping species in balance. Milestones attract birds, then other wildlife, and
eventually a castaway the player helps survive until rescue.

See DESIGN.md for the full game design and simulation architecture.

## Tech stack

- **Unreal Engine 5.8** — custom build at `D:\Unreal-Engine\UE_5.8`, registered under
  engine GUID `{FAE6B49B-473D-7B83-71AF-7CAC81B38D61}` (the `EngineAssociation` in the
  .uproject — do not change it to "5.8").
- **Blueprint-only for now.** The machine has no MSVC C++ toolset installed yet
  (VS 2022 Community is present without the C++ workload). When the workload is
  installed, add a C++ `Source/` module for the ecology simulation and Steamworks.
- **Official UE MCP plugin** (`ModelContextProtocol` + `AllToolsets`, experimental in
  5.8) — `.mcp.json` points at the editor's embedded server `http://127.0.0.1:8000/mcp`.
  The Unreal Editor must be running with this project open for MCP tools to work.
- Enabled plugins: Water (ocean/tides), PCG (vegetation scatter), GeometryScripting
  (procedural island mesh), SunPosition (day/night), plus MCP.

## Conventions

- Content lives under `Content/` with top-level folders: `Maps/`, `Blueprints/`,
  `Materials/`, `Meshes/`, `Audio/`, `UI/`, `Data/`.
- Blueprint assets prefix: `BP_` (actors), `WBP_` (widgets), `M_`/`MI_` (materials),
  `DT_` (data tables), `PCG_` (PCG graphs).
- Binary assets (`.uasset`, `.umap`, textures, audio) go through Git LFS — see
  `.gitattributes`.
- Design pillar: **calm and forgiving**. Nothing the player neglects should punish
  them harshly; AFK time always produces progress or gentle drift, never failure.
- Performance pillar: the game must idle politely (players leave it running). Keep
  tick work minimal; the ecosystem simulation advances on a slow timer, not per-frame.
