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

Install straight from the Git repository — no package registry involved:

> **Settings → Plugins → Add plugin** → paste the repo URL → Install

```
https://github.com/AliasgarBohra/dsh-blender-mcp
```

Paste that exact URL into the box (adjust the username if your repo lives
elsewhere). DSH runs `git ls-remote` first, so a wrong URL or a private repo
fails immediately with a clear message instead of hanging.

From the CLI, or to pin a release:

```powershell
# latest main
plugin_manager install_bundle --target 'https://github.com/AliasgarBohra/dsh-blender-mcp'

# pinned to a tag (reproducible)
plugin_manager install_bundle --target 'git+https://github.com/AliasgarBohra/dsh-blender-mcp#v1.0.0'
```

Local development also works:

```powershell
plugin_manager install_bundle --target 'C:\path\to\dsh-blender-mcp'          # folder
plugin_manager install_bundle --target '.\dsh-blender-mcp-1.0.0.tgz'         # tarball
```

> **A folder install records that absolute path.** If you later move or delete
> the folder, the plugin card under *Settings → Plugins* turns red with
> `Package metadata error … <that path>\package.json`, because the profile still
> points at a directory that no longer exists. The bundle is usually still
> installed and working; only the card's metadata lookup fails. Fix it with
> [Replacing a stale folder install](#replacing-a-stale-folder-install) rather
> than by recreating the old folder. Prefer the Git URL unless you are actively
> iterating on the bundle.

Enable it if it isn't already, then restart Blender if it was closed.

Requirements for the repo host: it must be **public** and reachable over HTTPS.
Cloning runs as your user, so SSH remotes work too if your keys are already set up.

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

### Replacing a stale folder install

If you once installed from a local folder and later moved or deleted it, the
profile still references that path. The symptom is a red plugin card reading
`Package metadata error … \package.json: Error: ENOENT: no such file or directory`.
The tools may even still work — only the metadata lookup is broken.

Swap the folder install for the Git one:

```powershell
plugin_manager remove_bundle --target dsh-blender-mcp
plugin_manager install_bundle --target 'https://github.com/AliasgarBohra/dsh-blender-mcp'
```

Then reload the GUI (`Ctrl+R`) or restart DSH so the plugin card re-reads its
metadata. Do **not** recreate the old folder to silence the error — that keeps
the profile pinned to a path you no longer maintain.

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
| Plugin card shows `Package metadata error … .dsh\bundles\dsh-blender-mcp\package.json` | A previous **folder** install points at a directory that no longer exists. The tools can still work. Remove the bundle and reinstall from the Git URL — see [Replacing a stale folder install](#replacing-a-stale-folder-install). |

## Security

`execute_blender_code` runs arbitrary Python inside Blender with your user's
permissions — filesystem, network, everything. This is true of every Blender MCP
integration, including Blender's own. Save your work; treat the port as a shell.

The extension's socket has **no authentication**. Leave it on `localhost`. To use
a remote Blender, enable *Allow Remote Connections*, bind to the machine's LAN
address, and set `BLENDER_MCP_HOST` — only on a trusted network, or over an SSH
tunnel.

## Publishing to Git

This bundle ships from a Git repository, so a release is a commit plus a tag:

```powershell
git add -A
git commit -m "v1.0.1: <what changed>"
git tag v1.0.1
git push origin main --tags
```

Rules that keep Git installs working:

- **`version` and the tag must match.** CI enforces it; a mismatched tag still
  installs, but users then get a version nobody can reproduce.
- **Every file the bundle needs must be committed.** Git installs fetch the
  repository, not your working directory — anything untracked is invisible.
- **`files` in `package.json` still applies.** It filters what lands in a packed
  install, so a file missing from it is missing for users even when committed.
- **Keep `node_modules/` and `*.tgz` ignored** (already in `.gitignore`) so the
  repository stays small and installs fast.

CI at `.github/workflows/validate.yml` checks the manifest, the patch YAML, and
the tag/version match on every push and PR — no secrets required.

### Pinning for users

`#v1.0.0` after the repo URL installs that exact tag; omitting it tracks `main`.
Pinned installs never surprise anyone with a change they didn't choose.

### Do not add `engines.dsh` casually

A range like `^0.2.0` excludes prerelease harness builds such as `0.2.2-rc.1`
under strict semver, which blocks the install entirely. Add a range only after
testing it.

### npm, optionally

Nothing here needs npm, but the package is registry-ready if you ever want the
one-word install: `npm publish --access public`. The name `dsh-blender-mcp` is
free on npm. Doing both is fine — the same files serve both channels.

## Credits

The Blender-side extension and `blmcp` server are GPL-3.0-or-later, © Blender
Authors, from [projects.blender.org/lab/blender_mcp](https://projects.blender.org/lab/blender_mcp).
This bundle only wires them into DeepSeek Harness.

## License

MIT — see [LICENSE](LICENSE).
