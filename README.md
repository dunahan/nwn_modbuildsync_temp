# NWN Module Build Template

A GitHub template repository for building [Neverwinter Nights: Enhanced Edition](https://www.beamdog.com/games/neverwinter-nights-enhanced/) modules with [nasher](https://github.com/squattingmonk/nasher) in GitHub Actions.

On every push to `main`, the workflow:

1. **Builds** `nasher` itself from source (via `nimble`) and packs the module from `src/` (`nasher pack --default`), using nasher's built-in [`nwn_script_comp`](https://github.com/niv/neverwinter.nim) compiler — actively maintained alongside EE, unlike the older standalone `nwnsc`
2. **Releases** the packed `.mod` as a dated GitHub Release
3. **Publishes** the module as an `nwsync` repository via GitHub Pages, so players can install and update it directly from inside the game — no manual `.mod` distribution needed

---

## One-time setup

1. **Enable GitHub Pages**
   Repo Settings → *Pages* → *Build and deployment* → Source: **GitHub Actions**

2. **Set a stable module UUID**
   Repo Settings → *Secrets and variables* → *Actions* → tab *Variables* → add a variable named `MODULE_UUID` with a UUIDv4, e.g.:
   ```
   python3 -c "import uuid; print(uuid.uuid4())"
   ```
   This value identifies the module across all future builds. **Do not change it later** — if it changes, the game will treat the next version as a completely new, unrelated module instead of an update to the existing one.

3. **Fill in `nasher.cfg`**
   Set `[package] name`, `description`, `version`, and the target `.mod` filename under `[target]`.

4. **Fill in `repository.json`**
   This is the catalog entry the in-game module browser reads (`ROOTURL/repository.json`). It is hand-maintained — the CI workflow only fills in build-specific fields (version hash, timestamp, contact/author) before publishing. Edit at least:
   - `name`, `description` (repo-level)
   - `modules[0].name`, `modules[0].description`

5. **Replace the header image (optional)**
   Swap out `assets/header_image.jpg` for your own artwork. The in-game browser expects **1920×600**; anything else gets centered and fit to the window instead of filling it.

---

## Using it in-game

After the first successful workflow run, add the following as a **Custom nwsync URL** under *Single Player → NWSync Repositories*:

```
https://<your-github-username>.github.io/<repo-name>/
```

The game will list your module, and can install or update it directly from there.

---

## File overview

| Path | Purpose |
|---|---|
| `src/` | nasher module source (compiled into the `.mod`) |
| `nasher.cfg` | nasher packaging configuration |
| `repository.json` | Hand-maintained catalog entry for the in-game module browser |
| `assets/header_image.jpg` | Preview image shown in the in-game browser |
| `.github/workflows/create-release.yaml` | CI: build → GitHub Release → nwsync build → GitHub Pages deploy |

---

## How the workflow builds nasher

Rather than using a pre-built third-party wrapper, the `ci_build` job builds everything itself for full version control and transparency:

- Downloads the official NWN dedicated server package (pinned via `NWNSERVER_VERSION` in the workflow's `env:` block) to provide `nwscript.nss` and standard includes via `NWN_ROOT`/`NWN_HOME`, needed by the compiler
- Installs Nim (`jiro4989/setup-nim-action`) and builds `nasher` from source with `nimble install nasher`, which also builds its dependency `neverwinter.nim` — including `nwn_script_comp` (the compiler) and `nwn_nwsync_write` (used later for the nwsync repo)
- The `~/.nimble` folder is cached, so only the first run (or a cache-key bump) pays the full compile time
- `nwn_nwsync_write` is passed to the separate `nwsync_pages` job as a build artifact, so that job doesn't need to install or build anything itself

---

## Notes & limitations

- **Persistent worlds:** the nwsync build uses `--with-module`, which packages the full module for distribution. This is explicitly *not* intended for persistent worlds — use a plain (non-`--with-module`) nwsync setup for those instead.
- **Version history:** GitHub Pages replaces the entire published site on every deploy, so `repository.json` only ever lists the *current* build's version. Fine for small test projects; a real version history would need additional logic to merge in prior entries instead of overwriting them.
- **GitHub Pages limits:** roughly 1 GB published site size and 100 GB/month bandwidth (soft limits). For larger content packs (lots of textures, voice, etc.), swap the final "Deploy to GitHub Pages" step for your own static host (rsync/rclone to a server, S3, ...) — the nwsync build and `repository.json` generation stay the same either way.
- **Private repos:** GitHub Pages only works from a private repo on GitHub Pro (personal) or Team/Enterprise (orgs) — on the Free plan the repo must stay public for Pages (and therefore the in-game nwsync URL) to work at all.
- **Pinned versions to revisit occasionally:** `NWNSERVER_VERSION` in the workflow's `env:` block (bump when you need newer game data), and the major versions of the third-party/GitHub Actions used (`actions/checkout`, `actions/cache`, `actions/upload-artifact`, `actions/download-artifact`, `softprops/action-gh-release`, `actions/upload-pages-artifact`, `actions/deploy-pages`, `jiro4989/setup-nim-action`) — check periodically for new majors.
