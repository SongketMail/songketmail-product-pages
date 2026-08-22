# Songketmail Docs Repo

This repository (`songketmail/songketmail-product-pages`) is the **Mintlify docs deployment source** for [songketmail.mintlify.app](https://songketmail.mintlify.app).

> ⚠️ **Do not edit files in this repo directly.** This repo is auto-synced from the app repo. Manual changes here will be overwritten on the next sync.

## Where to edit docs

Edit docs in the app repo: [**SongketMail/songketmail**](https://github.com/SongketMail/songketmail) under the `docs-source/` folder.

On every push to `main` that touches `docs-source/`, a GitHub Actions workflow copies the contents into this repo and pushes to `main` here. Mintlify then rebuilds automatically.

```text
SongketMail/songketmail (app repo)
  └── docs-source/                    ← edit here
       ├── *.mdx
       └── docs.json
  └── .github/workflows/sync-docs.yml
  └── scripts/sync_docs.py

     ↓ (on push to main, paths: docs-source/**)

songketmail/songketmail-product-pages (this repo, auto-synced)
  └── *.mdx
  └── docs.json

     ↓ (Mintlify GitHub app watches main)

https://songketmail.mintlify.app
```

## Setup checklist

### ✅ Done

- Mintlify deployment `songketmail-email-services` created
- Git source connected: `songketmail/songketmail-product-pages` on `main`
- Live site: [https://songketmail.mintlify.app](https://songketmail.mintlify.app)

### ⬜ To do (in app repo `SongketMail/songketmail`)

1. **Create `docs-source/` folder** and copy the current contents of this repo into it (all `.mdx` files, `docs.json`, `logo/`, `favicon.svg`, etc.) as the starting point.
2. **Add `scripts/sync_docs.py`:**
   ```python
   import os
   import shutil
   import subprocess
   from pathlib import Path
   SOURCE_DIR = Path("docs-source")
   DOCS_REPO = "songketmail/songketmail-product-pages"
   BRANCH = "main"
   TOKEN = os.environ["DOCS_REPO_TOKEN"]
   def run(cmd, cwd=None):
       subprocess.run(cmd, cwd=cwd, check=True, shell=isinstance(cmd, str))
   def main():
       tmp = Path("/tmp/docs-repo")
       if tmp.exists():
           shutil.rmtree(tmp)
       # Clone docs repo
       url = f"https://x-access-token:{TOKEN}@github.com/{DOCS_REPO}.git"
       run(["git", "clone", "--branch", BRANCH, url, str(tmp)])
       # Wipe old docs content (keep .git)
       for item in tmp.iterdir():
           if item.name == ".git":
               continue
           if item.is_dir():
               shutil.rmtree(item)
           else:
               item.unlink()
       # Copy new content
       shutil.copytree(SOURCE_DIR, tmp, dirs_exist_ok=True)
       # Commit and push
       run(["git", "config", "user.email", "bot@songketmail.com"], cwd=tmp)
       run(["git", "config", "user.name", "Docs Sync Bot"], cwd=tmp)
       run(["git", "add", "-A"], cwd=tmp)
       result = subprocess.run(["git", "diff", "--staged", "--quiet"], cwd=tmp)
       if result.returncode == 0:
           print("No changes")
           return
       run(["git", "commit", "-m", "Sync docs from app repo"], cwd=tmp)
       run(["git", "push", "origin", BRANCH], cwd=tmp)
       print("Synced")
   if __name__ == "__main__":
       main()
   ```
3. **Add `.github/workflows/sync-docs.yml`:**
   ```yaml
   name: Sync Docs
   on:
     push:
       branches: [main]
       paths:
         - "docs-source/**"
   jobs:
     sync:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - uses: actions/setup-python@v5
           with:
             python-version: "3.11"
         - name: Sync docs
           env:
             DOCS_REPO_TOKEN: ${{ secrets.DOCS_REPO_TOKEN }}
           run: python scripts/sync_docs.py
   ```
4. **Create a fine-grained Personal Access Token:**
   - Go to [https://github.com/settings/tokens](https://github.com/settings/tokens) → **Fine-grained tokens** → **Generate new token**
   - Repository access: `songketmail/songketmail-product-pages`
   - Permissions: **Contents: Read and write**
   - Copy the token
5. **Add token as a secret in the app repo:**
   - `SongketMail/songketmail` → **Settings → Secrets and variables → Actions → New repository secret**
   - Name: `DOCS_REPO_TOKEN`
   - Value: paste token
6. **Test the pipeline:**
   - Make a small edit to a file under `docs-source/` in the app repo
   - Commit and push to `main`
   - Watch the workflow run in **Actions** tab
   - Confirm this repo receives the sync commit
   - Confirm [https://songketmail.mintlify.app](https://songketmail.mintlify.app) rebuilds

## Rules

- ❌ Do NOT edit files in this repo (`songketmail-product-pages`) directly
- ❌ Do NOT edit in the Mintlify web editor (changes will be overwritten)
- ✅ Only edit `docs-source/` in `SongketMail/songketmail`
- ✅ Push to `main` → auto-sync → Mintlify rebuild

## Local preview (in app repo)

```bash
cd docs-source
npm i -g mint
mint dev
```

Preview at [http://localhost:3000](http://localhost:3000).

## Resources

- [Mintlify documentation](https://mintlify.com/docs)
- [Live site](https://songketmail.mintlify.app)
- [App repo (edit docs here)](https://github.com/SongketMail/songketmail)