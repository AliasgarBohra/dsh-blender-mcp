# Changelog

All notable changes to this bundle. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow [SemVer](https://semver.org/).

## [1.0.0] — 2026-11-07

### Added

- Configuration-only DSH bundle exposing a running Blender as `mcp__blender__*`
  tools: `execute_blender_code`, `get_objects_summary`, `get_object_detail_summary`,
  area/window screenshots, viewport and thumbnail renders, viewport focus helpers,
  blend-file summaries, and offline Blender API/manual search.
- Bridges Blender Lab's official MCP server (`blmcp`) over stdio, pinned to
  upstream `v1.0.3`, resolved by `uvx` from source — no local clone, no manual
  Python setup.
- Configurable `BLENDER_MCP_HOST` / `BLENDER_MCP_PORT` for a remote Blender.
- 300 s per-call timeout for long modeling and render operations.
