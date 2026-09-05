# NWN Module Build Template

A GitHub template repository for building [Neverwinter Nights: Enhanced Edition](https://www.beamdog.com/games/neverwinter-nights-enhanced/) modules with [nasher](https://github.com/squattingmonk/nasher) and [nasher4gh](https://github.com/Ardesco/nasher4gh) in GitHub Actions.

On every push to `main`, the workflow:

1. **Builds** the module from `src/` (`nasher pack --default -usenwnscriptcomp`)
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
   Swap out `assets/header_image.jpg` for your own artwork. Any image works — it's referenced purely by relative path.

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

## Notes & limitations

- **Persistent worlds:** the nwsync build uses `--with-module`, which packages the full module for distribution. This is explicitly *not* intended for persistent worlds — use a plain (non-`--with-module`) nwsync setup for those instead.
- **Version history:** GitHub Pages replaces the entire published site on every deploy, so `repository.json` only ever lists the *current* build's version. Fine for small test projects; a real version history would need additional logic to merge in prior entries instead of overwriting them.
- **GitHub Pages limits:** roughly 1 GB published site size and 100 GB/month bandwidth (soft limits). For larger content packs (lots of textures, voice, etc.), swap the final "Deploy to GitHub Pages" step for your own static host (rsync/rclone to a server, S3, ...) — the nwsync build and `repository.json` generation stay the same either way.
- **nwsync tooling:** the workflow downloads the Linux release of [`neverwinter.nim`](https://github.com/niv/neverwinter.nim) matching `*linux*`. If that stops matching after an upstream release naming change, check the [releases page](https://github.com/niv/neverwinter.nim/releases) and adjust the `--pattern` in the workflow.
