# Neverwinter Nights Module Building and Publishing via nwsync template
This is a mod building and publishing template for Neverwinter Nights that uses nasher, nasher4gh and nwsync via GitHub pages (can be changed later).

To ensure ${{ vars.MODULE_UUID }} works, configure it once:
- Generate a UUIDv4 locally: uuidgen (Linux/Mac) or an online generator
- In the repository: Settings → Secrets and variables → Actions → Variables tab → New repository variable → Name MODULE_UUID, value the generated UUID

This keeps the module identity stable across all releases, and players who have already installed the module via nwsync will receive an update instead of a duplicate with a new version.

This should be a UUIDv4, e.g. generated using python3 -c \"import uuid; print(uuid.uuid4())\" – not uuidgen without -r, as this sometimes returns v1, or via online generators.

There's already a UUIDv4 as a variable available (98446a6a-84df-42b6-948d-37cbfbe89394), therefore it is necessary to override it with another one.

Following an example of the repository.json, which the workflow will create:

```
          repo = {
              "format_version": 1,
              "name": pkg.get("name", "module"),
              "description": pkg.get("description", ""),
              "contact": f"https://github.com/{os.environ['REPO_OWNER']}",
              "header_image": "",
              "modules": [
                  {
                      "uuid": os.environ["MODULE_UUID"],
                      "name": pkg.get("name", "module"),
                      "author": os.environ["REPO_OWNER"],
                      "description": pkg.get("description", ""),
                      "header_image": "",
                      "screenshots": [],
                      "versions": [
                          {
                              "identifier": manifest_hash,
                              "version": pkg.get("version", "0.1"),
                              "created_at": int(time.time()),
                          }
                      ],
                      "multiplayer": False,
                  }
              ],
          }
```
