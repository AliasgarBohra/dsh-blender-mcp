# dsh-blender-mcp

A configuration-only [DeepSeek Harness](https://deepseek.ai) bundle that puts a **running Blender** into the agent's tool list.

Install it, keep Blender open, and the agent can model, shade, light, frame, render, and *look at* the result — in a loop, in your actual Blender session.

| Tool | What it does |
|---|---|
| `mcp__blender__execute_blender_code` | Run Python with full `bpy` in your live Blender |
| `mcp__blender__get_objects_summary` | Collection tree + objects, type/parent/visibility |
| `mcp__blender__get_object_detail_summary` | Transforms, modifiers, constraints, materials for one object |
| `mcp__blender__get_screenshot_of_area_as_image` | See one Blender area (viewport, shader editor, …) |
| `mcp__blender__get_screenshot_of_window_as_image` | See the whole Blender window |
| `mcp__blender__get_screenshot_of_window_as_json` | Window layout, areas, active object, selection |
| `mcp__blender__render_viewport_to_path` | Render the current scene to a file |
| `mcp__blender__render_thumbnail_to_path` | Fast low-quality preview render |
| `mcp__blender__jump_to_view3d_object_by_name` | Focus the viewport on an object |
| `mcp__blender__get_blendfile_summary_*` | Blend-file stats, missing files, linked libraries, path info |
| `mcp__blender__get_python_api_docs` · `search_api_docs` · `search_manual_docs` | Offline Blender API + manual lookup |

## Requirements

1. **Blender 5.1 or newer**, running.
2. **The official MCP extension** (by Blender Lab) enabled in Blender. Install once:
   *Edit → Preferences → Get Extensions → search "MCP"*, or drag the
   [release zip](https://www.blender.org/lab/mcp-server/) into the viewport.
   It auto-starts a socket server on `127.0.0.1:9876` when Blender opens; check
   *Preferences → Add-ons → MCP* if you want to confirm or disable it.
3. **uv** on `PATH`:

   ```powershell
   # Windows
   winget install --id=astral-sh.uv -e
   ```
   ```bash
   # macOS
   brew install uv
   # Linux
   curl -LsSf https://astral.sh/uv/install.sh | sh
   ```

`uvx` builds the MCP server from source on first use (~20 s, network required)
and caches it; later starts take about a second.

## Install

Once published to npm, it is a package name:

> **Settings → Plugins → Add plugin** → `dsh-blender-mcp` → Install

Without npm — local folder, tarball, or a git repo:

```powershell
plugin_manager install_bundle --target 'C:\path\to\dsh-blender-mcp'         # folder
plugin_manager install_bundle --target '.\dsh-blender-mcp-1.0.0.tgz'        # tarball
plugin_manager install_bundle --target 'https://github.com/<you>/<repo>'    # git
```

Enable it, then restart Blender if it was closed.

## Verify

Ask the agent to call `mcp__blender__get_objects_summary`. A healthy answer names
your scene and lists its objects. If the tools do not appear at all, see
Troubleshooting.

## Configuration

Defaults suit a local Blender. To deviate, edit `cordis.patch.yml` in this package
(or copy its `- insert:` block into your profile's `cordis.patch.yml`):

| Setting | Default | Meaning |
|---|---|---|
| `command` | `uvx` | Use an absolute `uvx.exe` path if a GUI-launched harness can't find it |
| `BLENDER_MCP_HOST` | `localhost` | Where Blender's socket server listens |
| `BLENDER_MCP_PORT` | `9876` | Its port |
| `toolCallTimeoutMs` | `300000` | Per-call timeout; renders are slow |
| `failOnStartupError` | `false` | Keep the harness usable when Blender is closed |

The patch keeps everything portable — no absolute paths, no local clone — so the
same artifact works on any machine.

## Updating upstream

The server is pinned to the upstream release tag `v1.0.3`:

```yaml
- "git+https://projects.blender.org/lab/blender_mcp.git@v1.0.3#subdirectory=mcp"
```

Bump the tag to pick up fixes, or drop `@v1.0.3` to track `main` (faster-moving,
less reproducible). Then bump this bundle's `version` and republish.

## Updating or uninstalling this bundle

```powershell
plugin_manager remove_bundle --target dsh-blender-mcp    # also disabled under Settings → Plugins
plugin_manager install_bundle --target 'dsh-blender-mcp@1.0.1'
```

Removing the bundle leaves Blender and the MCP extension untouched — it only
removes the harness-side wiring, and the `uvx` cache stays warm for a reinstall.

### Two Blender instances

One entry talks to one port. For a second Blender, set that instance's extension
to a different port, then add a second row to `cordis.patch.yml` with
`serverName: blender2`, `BLENDER_MCP_PORT: "9877"`, and a distinct `id`. Tools
arrive as `mcp__blender2__*`.

## Troubleshooting

| Symptom | Cause and fix |
|---|---|
| No `mcp__blender__*` tools at all | Blender not open, or the MCP extension's server stopped. Open Blender → *Preferences → Add-ons → MCP*. |
| `server is disconnected` in the harness | The `uvx` spawn failed. Confirm `uvx --version` works in a terminal; if the harness was launched from a GUI, set `command` to the absolute `uvx.exe` path. |
| `Cannot connect to Blender at localhost:9876…` | The server is up but Blender isn't listening: wrong port, or the extension is disabled. |
| First call hangs ~20 s | `uvx` is building the server. Subsequent calls are instant. Needs network the first time. |
| Tools stop mid-session | Blender was closed or the extension restarted. The client reconnects automatically once Blender is back. |

## Security

`execute_blender_code` runs arbitrary Python inside Blender with your user's
permissions — filesystem, network, everything. This is true of every Blender MCP
integration, including Blender's own. Save your work; treat the port as a shell.

The extension's socket has **no authentication**. Leave it on `localhost`. To use
a remote Blender, enable *Allow Remote Connections*, bind to the machine's LAN
address, and set `BLENDER_MCP_HOST` — only on a trusted network, or over an SSH
tunnel.

## Publishing

Publish-ready: no dependencies, no build step.

```powershell
npm login                     # once
npm pack --dry-run            # confirm exactly the declared files ship
npm publish --access public   # name must be free on npm
```

Bump `version` for every release — npm refuses to overwrite. A tag-triggered
GitHub Actions workflow is included at `.github/workflows/publish.yml`; it checks
that the tag matches `package.json` before publishing, and expects an `NPM_TOKEN`
secret.

**Do not add `engines.dsh` casually.** A range like `^0.2.0` excludes prerelease
harness builds such as `0.2.2-rc.1` under strict semver, which blocks the install
entirely. Add a range only after testing it.

### Distribution channels

| Channel | How users install | Trade-off |
|---|---|---|
| npm registry | package name in the Add-plugin box | best UX, needs an npm account |
| GitHub repo | the repo URL in the Add-plugin box | no npm account, but users must know the URL |
| Custom plugin index | Discover tab, with the experimental `dsh-experimental-plugin-market-bundle` | discoverable, but needs an index JSON and an experimental bundle |

## Credits

The Blender-side extension and `blmcp` server are GPL-3.0-or-later, © Blender
Authors, from [projects.blender.org/lab/blender_mcp](https://projects.blender.org/lab/blender_mcp).
This bundle only wires them into DeepSeek Harness.

## License

MIT — see [LICENSE](LICENSE).
