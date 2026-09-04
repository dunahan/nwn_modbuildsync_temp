# Neverwinter Nights Module Building and Publishing via nwsync template
This is a mod building and publishing template for Neverwinter Nights that uses nasher, nasher4gh and nwsync via GitHub pages (can be changed later).

To ensure ${{ vars.MODULE_UUID }} works, configure it once:
- Generate a UUIDv4 locally: uuidgen (Linux/Mac) or an online generator
- In the repository: Settings → Secrets and variables → Actions → Variables tab → New repository variable → Name MODULE_UUID, value the generated UUID

This keeps the module identity stable across all releases, and players who have already installed the module via nwsync will receive an update instead of a duplicate with a new version.

This should be a UUIDv4, e.g. generated using python3 -c \"import uuid; print(uuid.uuid4())\" – not uuidgen without -r, as this sometimes returns v1, or via online generators.
