# VEKTOR Slipstream: Windows Installer + Auto-Update Plan

**Approach:** bundled `node.exe` + app directory, packaged as a signed installer with **Velopack**, updates served as static files from the existing VPS.

**Status:** plan / draft. Items marked **[VERIFY]** depend on current Velopack docs or tooling and must be confirmed during the spike (Phase 0).

---

## 1. Goals and non-goals

### Goals
- One-click Windows install for customers (no Node, npm or nvm required).
- Silent, automatic updates with delta downloads.
- Customer data (`~/.vektor`) survives every update, with automatic backup before migrations.
- Signed binaries, so there are no SmartScreen warnings.
- Release flow that extends the existing weekly deploy (`pre-deploy-test.js` -> pack -> VPS).
- Stable executable path so Claude Desktop / DXT / MCP configs don't break on update.

### Non-goals
- A true single-file `.exe`. Native addons (`better-sqlite3`, `onnxruntime-node`) cannot be embedded in a packed exe and must live on disk. Packers (Node SEA, `@yao-pkg/pkg`, nexe) would only add fragility.
- macOS/Linux installers (separate plan; Velopack supports them, but out of scope here).
- A GUI. This is a CLI + MCP server. (If a GUI is wanted later, revisit Electron.)

---

## 2. Architecture

```
%LocalAppData%\Vektor\                (managed by Velopack, per-user, no admin)
  Update.exe                          Velopack updater/launcher
  current\                            stable path, always the active version
    vektor.exe                        launcher stub (see 4.2)
    node\node.exe                     bundled Node runtime (pinned)
    app\
      vektor.mjs
      vektor-slipstream-dxt\server\index.js
      *.js (intelligence layers, connectors, wizard, ...)
      node_modules\                   production deps only, native addons unpacked
      models\                         ONNX / embedder models (if bundled)
  packages\                           downloaded full/delta packages

%USERPROFILE%\.vektor\                USER DATA - never inside the install dir
  slipstream-memory.db
  backups\
  logs\
  config.json
```

Key principle: **install dir is disposable, user data is permanent.** Velopack replaces `current\` wholesale on update, so nothing the user owns may live there.

### Update flow

```
app start / daily timer
  -> check feed https://<host>/releases/releases.win.json
  -> newer version? download delta (or full) in background
  -> on next launch (or user-approved restart): Velopack swaps current\
  -> first launch of new version: backup DB -> run migrations -> start
```

---

## 3. Prerequisites

| Item | Notes |
|---|---|
| Windows build machine | The minimaxa PC works. Native modules must be installed on Windows, not cross-built from Linux. |
| .NET SDK | Needed to install the Velopack CLI: `dotnet tool install -g vpk` **[VERIFY]** |
| Code-signing | **Azure Trusted Signing** (cheapest, no hardware token) or an OV/EV certificate. Decide in Phase 0. |
| Pinned Node | Pick one version (e.g. the 24.x LTS-compatible build used in dev) and pin it. Download the Windows x64 `node.exe` zip from nodejs.org and verify the checksum. |
| Icon | `vektor.ico` (multi-size 16-256 px). |
| Feed hosting | Static files over HTTPS. The existing VPS + nginx is sufficient. |
| Test VM | Clean Windows 10/11 VM or Windows Sandbox for install/update testing. |

---

## 4. Build pipeline

### 4.1 Stage the app

Script: `build/stage.ps1`

```powershell
$ErrorActionPreference = 'Stop'
$root   = 'C:\Users\minimaxa\vektor-v2\vektor-slipstream-sdk'
$stage  = "$root\build\stage"
$ver    = (Get-Content "$root\package.json" | ConvertFrom-Json).version

Remove-Item $stage -Recurse -Force -ErrorAction SilentlyContinue
New-Item "$stage\app", "$stage\node" -ItemType Directory | Out-Null

# 1. pre-deploy gate (MANDATORY)
Push-Location $root
node pre-deploy-test.js 2>$null | Select-String 'ALL TESTS PASSED' -Quiet |
  ForEach-Object { if (-not $_) { throw 'pre-deploy-test failed' } }

# 2. app files (allowlist, not blanket copy)
#    vektor.mjs, *.js intelligence layers, DXT server, postinstall logic, package.json, package-lock.json
robocopy $root "$stage\app" vektor.mjs vektor-*.js slipstream-*.js cloak-*.js migrate-*.js package.json package-lock.json
robocopy "$root\vektor-slipstream-dxt" "$stage\app\vektor-slipstream-dxt" /E

# 3. production deps from the lockfile (reproducible)
Push-Location "$stage\app"
npm ci --omit=dev
Pop-Location

# 4. pinned Node runtime
Copy-Item 'C:\build-cache\node-v24.x-win-x64\node.exe' "$stage\node\node.exe"

# 5. launcher
Copy-Item "$root\build\launcher\vektor.exe" "$stage\vektor.exe"
Pop-Location
```

Notes:
- Use an **allowlist** copy so `.bak`, test DBs, `slipstream-memory.db` (dev), `.env` and secrets never ship.
- Keep `onnxruntime-node` pinned to **1.24.1** (the known-good version; 1.26.0 breaks the runtime API). Verify the `overrides` block survives `npm ci`.
- Prune `node_modules` for size: remove `onnxruntime-node` platform folders other than `win32/x64`, plus docs, tests, `.md`, `.map` files.
- Confirm the DXT server's stderr filter and the `mem.store()` fix are in the staged `server/index.js` (v1.6.3 fixes).

### 4.2 Launcher (`vektor.exe`)

Velopack needs a real `.exe` as the main executable. Options:

| Option | Pros | Cons |
|---|---|---|
| **Tiny Go/Rust stub** that spawns `node\node.exe app\vektor.mjs <args>` and forwards stdio/exit code | Small (~1-2 MB), simple, fully controlled | A second toolchain |
| **Node SEA launcher** (a ~5-line script injected into a copy of node.exe that spawns the real app) | Stays in the Node ecosystem | Larger exe (~80 MB duplicate runtime) |
| `.cmd` shim | Zero build step | Not an exe; MCP clients, SmartScreen and shortcuts handle it poorly. Not recommended. |

**Recommendation:** Go stub. Requirements: pass through all CLI args, inherit stdio (MCP uses stdio JSON, so no extra output may ever be written to stdout), propagate exit codes, set `VEKTOR_DB_PATH` explicitly (see 8).

### 4.3 Pack with Velopack

```powershell
vpk pack `
  --packId Vektor `
  --packTitle "VEKTOR Slipstream" `
  --packVersion $ver `
  --packDir "$stage" `
  --mainExe vektor.exe `
  --icon "$root\build\vektor.ico" `
  --outputDir "$root\build\Releases" `
  --channel win        # [VERIFY] flag names against current vpk docs
```

Produces (typical): `Vektor-Setup.exe`, `Vektor-<ver>-full.nupkg`, `Vektor-<ver>-delta.nupkg` (when a previous release is present in the output dir), `releases.win.json` / `RELEASES`.

**Important:** to get delta packages, download the previous release into the output dir first (`vpk download ...` **[VERIFY]**) or keep `Releases\` persistent between builds.

### 4.4 Code signing

- Sign `node.exe`, `vektor.exe`, and let Velopack sign `Setup.exe`/`Update.exe` via its signing options **[VERIFY: `--signParams` / Azure Trusted Signing support]**.
- Never commit certificates or tokens. Store them in the vault (`cloak_passport`) like the existing keys.
- Timestamp every signature so they stay valid after the cert expires.

### 4.5 Publish to VPS

```powershell
scp build\Releases\* ubuntu@159.13.45.182:~/releases-incoming/
ssh ubuntu@159.13.45.182 "bash publish-release.sh"
```

`publish-release.sh` (on VPS):
1. Verify checksums of the uploaded files.
2. Move packages to `/home/ubuntu/tyrell-bot/public/releases/win/`.
3. **Upload the manifest (`releases.win.json`) last**, so clients never see a manifest that references files not yet present.
4. Keep the previous **two** full packages for rollback.
5. Update `VERSION_*` in `/home/ubuntu/vektor-monorepo/server/.env` and restart with HTTPS certs per the HTTPS rule (`pm2 delete` + `pm2 start` with `SSL_KEY`/`SSL_CERT`, `pm2 restart` is not enough).
6. Verify `curl -sk https://localhost:3000/downloads` returns 200 and the feed URL returns 200.

nginx: serve `/releases/win/` as static, long cache on `.nupkg`, **no cache** on the manifest.

---

## 5. Auto-update in the app

### 5.1 Mechanism

Velopack apps update through `Update.exe` in the install root. Two integration options **[VERIFY which is current/stable for Node]**:

1. **Velopack Node.js binding** (npm `velopack`) imported in `vektor.mjs`, using `UpdateManager` to check, download and apply.
2. **Shell out to `Update.exe`** from the launcher or `vektor.mjs` (CLI verbs for check/download/apply).

If the Node binding is not stable, do the update check in the **launcher stub** (Go), since it already runs before Node.

### 5.2 Behavior spec

- Check on startup (non-blocking) and every 24 hours while running.
- Download in the background; never block MCP stdio.
- **Never apply an update mid-session.** Apply on next start, or via an explicit `vektor update` command or user prompt. The MCP server must not be killed under an active Claude Desktop session.
- All update logging goes to `~/.vektor/logs/update.log`, **never stdout/stderr** (stdio contamination breaks MCP; this has bitten us before).
- Respect `VEKTOR_AUTOUPDATE=0` to opt out (enterprise/air-gapped).
- Failure to reach the feed must be silent and non-fatal.

### 5.3 Licensing and channels

- Feed is public static files, so gate **entitlement**, not bytes: the app verifies the licence key against the existing server before applying an update beyond the user's entitled version.
- Use Velopack channels (`win-stable`, `win-beta`) so beta testers can opt in.

---

## 6. First-run and post-update migration

`npm postinstall` no longer runs, so replace it with a startup step in `vektor.mjs`:

```
on start:
  read ~/.vektor/state.json -> lastRunVersion
  if lastRunVersion != currentVersion:
      1. copy slipstream-memory.db -> ~/.vektor/backups/slipstream-memory-<lastRunVersion>-<timestamp>.db
         (use SQLite online backup API / VACUUM INTO, not a raw copy while open)
      2. run SQLite capability checks (formerly postinstall.js)
      3. run migrations (migrate-162.js and successors), idempotent, in a transaction
      4. on failure: restore backup, log, continue on old schema or exit with a clear error
      5. write state.json lastRunVersion = currentVersion
  keep the last N (e.g. 5) backups, prune older
```

Rules:
- Migrations must be **idempotent** and **forward-only**; never assume the user skipped no versions (deltas can jump multiple versions).
- The DB path must be explicit, either `VEKTOR_DB_PATH` or the default `%USERPROFILE%\.vektor\slipstream-memory.db`, to avoid the "ghost DB" problem.

---

## 7. Claude Desktop / MCP / DXT integration

- Stable entry point: `%LocalAppData%\Vektor\current\vektor.exe`. Velopack keeps `current` stable across updates, so MCP configs survive.
- The installer (or first run) writes/updates the MCP entry in `%APPDATA%\Claude\claude_desktop_config.json`:
  ```json
  "vektor": {
    "command": "C:\\Users\\<user>\\AppData\\Local\\Vektor\\current\\vektor.exe",
    "args": ["mcp"],
    "env": { "VEKTOR_DB_PATH": "C:\\Users\\<user>\\.vektor\\slipstream-memory.db" }
  }
  ```
  Back up the config before editing, merge rather than overwrite, and only touch the `vektor` key.
- **Do not hardcode licence keys, tokens or PATs into any shipped file or config template.** The licence key is entered/activated at first run and stored in the user's profile.
- Claude Desktop loads VEKTOR via the DXT extension, not `mcpServers`. Decide whether the installer also registers/refreshes the DXT or whether the DXT is distributed separately; if it is bundled, make sure its server path points at the stable `current\` location.
- Add `current\` to the user PATH (optional) so `vektor` works in any terminal.

---

## 8. Environment and runtime pinning

| Concern | Decision |
|---|---|
| Node version | Pinned; bump deliberately and re-run the full test matrix |
| `onnxruntime-node` | 1.24.1 pinned (dependencies + overrides) |
| `VEKTOR_DB_PATH` | Always set explicitly by the launcher |
| stderr noise | Keep the DXT stderr filter (`[vektor-embedder] cross-encoder ready` must not reach stdio) |
| Native module ABI | Built on Windows against the **same Node version** that ships in the bundle |
| Proxy / offline | Update check fails silently; app works fully offline |

---

## 9. Testing plan

### Pre-release (every build)
- [ ] `node pre-deploy-test.js` prints `ALL TESTS PASSED`.
- [ ] Installed-layout smoke test: `vektor.exe --version`, `store`, `recall`, DB health.
- [ ] MCP stdio test: spawn `vektor.exe mcp`, send `initialize` + `tools/list`, confirm clean JSON with no stray output.
- [ ] Signature check: `Get-AuthenticodeSignature` on all exes.

### Clean-VM matrix
- [ ] Fresh install on clean Windows 10 and 11 (no Node, no admin).
- [ ] Install over an existing dev/nvm setup (both coexist, no PATH conflicts).
- [ ] SmartScreen / Defender: no warnings on signed build; submit new builds to Microsoft for reputation if needed.
- [ ] Uninstall: removes `%LocalAppData%\Vektor`, leaves `~/.vektor` (and says so).

### Update tests
- [ ] N -> N+1 delta update with the app closed.
- [ ] N -> N+3 (skipping versions): migrations still apply cleanly.
- [ ] Update with the DB open / MCP session active: update is deferred, no corruption.
- [ ] Interrupted download / power loss: next launch recovers, old version still runs.
- [ ] Corrupt/missing manifest: app runs normally, logs the failure.
- [ ] Migration failure injection: backup restored, app still usable.
- [ ] Rollback: pull release from the feed, confirm clients stay on a working version.

---

## 10. Release checklist (weekly)

1. [ ] Bump version in `package.json` (and DXT manifest).
2. [ ] `node pre-deploy-test.js 2>$null` -> `ALL TESTS PASSED`.
3. [ ] `build/stage.ps1` (staging).
4. [ ] Sign binaries.
5. [ ] `vpk pack` (with previous release present for deltas).
6. [ ] Run clean-VM install + one update test.
7. [ ] Upload to the VPS; manifest last.
8. [ ] Update `.env` `VERSION_SLIPSTREAM/PRO/STUDIO`; restart `vektor-server` with HTTPS certs.
9. [ ] Verify: `https://localhost:3000/downloads` -> 200; feed URL -> 200; installed test machine picks up the update.
10. [ ] Keep `npm pack` of the tgz in the monorepo `releases/` for the existing npm channel (until it is retired).

---

## 11. Rollback and incident handling

- **Bad release:** remove its entries from the manifest and republish the previous manifest; clients that haven't applied it never see it. Clients that did apply it need a **higher** fixed version, since Velopack won't downgrade by default, so ship a hotfix version rather than trying to withdraw.
- **Broken migration:** the pre-migration backup is the safety net; document the manual restore (`copy backups\...db slipstream-memory.db`).
- **Cert/signing failure:** don't ship unsigned. Hold the release.
- **Feed outage:** clients keep working; the update check fails silently.

---

## 12. Security

- Sign everything; timestamp signatures.
- Serve the feed over HTTPS only. Velopack validates package integrity from the manifest hashes; keep the manifest and packages on the same trusted origin.
- No secrets in the build output. Scan the staged folder for tokens/keys before packing (`grep -rEi "ghp_|token|secret|licen[cs]e" build/stage` and inspect hits).
- **Rotate any credential that has ever been pasted into docs, skill files or configs** (e.g. the GitHub PAT and licence key currently embedded in the dev skill file) before the first public release.
- Keep signing credentials in the vault, never in the repo.

---

## 13. Phased rollout

| Phase | Deliverable | Exit criteria |
|---|---|---|
| **0. Spike** (1-2 days) | Stage + `vpk pack` locally; install on a VM; confirm native modules load; confirm the Node-binding vs `Update.exe` question; pick the signing route | Installed app runs `store`/`recall` and MCP on a clean VM |
| **1. Installer MVP** | Launcher stub, stage script, signed installer, VPS publish script | Customers can install from the downloads page |
| **2. Auto-update** | Background check, deferred apply, delta packages, migration-on-start with backups | N -> N+1 and N -> N+3 tests pass |
| **3. Hardening** | MCP config registration, uninstall behavior, channels, rollback runbook, telemetry-free update logging | Full test matrix green |
| **4. Cutover** | Downloads page points at the installer; npm tgz kept for devs | One full release cycle without manual fixes |

---

## 14. Open questions / decisions needed

1. Code-signing route: Azure Trusted Signing vs OV/EV cert?
2. Launcher: Go stub vs Node SEA?
3. Update integration: Velopack Node binding vs `Update.exe` shell-out **[VERIFY]**.
4. Are the ONNX models bundled (bigger installer, works offline) or downloaded on first run (smaller installer, needs network)?
5. Is the DXT bundled by the installer or distributed separately?
6. Apply updates silently on next start, or prompt the user?
7. Beta channel: yes/no at launch?
8. Retire the npm `.tgz` download channel, or keep it for developers?

---

## 15. References (check current versions before relying on them)

- Velopack docs: <https://docs.velopack.io>
- Node.js Single Executable Applications: <https://nodejs.org/api/single-executable-applications.html>
- Azure Trusted Signing docs (Microsoft Learn)
- SQLite online backup API / `VACUUM INTO`
