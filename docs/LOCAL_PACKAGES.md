# Local Package Development Workflow

This project uses two packages that are developed locally alongside the main application:

- `nasirkhan/laravel-cube` — located at `../laravel-starter-packages/laravel-cube`
- `nasirkhan/module-manager` — located at `../laravel-starter-packages/module-manager`

These are referenced via `composer.local.json` as path repositories with symlinks, which allows local changes to be reflected immediately without publishing to Packagist.

## The Problem

When `composer.local.json` is present, Composer resolves these packages as local `path` types and writes that into `composer.lock`. CI environments (GitHub Actions, etc.) cannot resolve those local paths, causing this error:

```
Source path "../laravel-starter-packages/laravel-cube" is not found for package nasirkhan/laravel-cube
```

Additionally, `composer.lock` has the `skip-worktree` git flag set locally, which prevents the path-based lock file from being accidentally committed to the repository.

> **Note:** `skip-worktree` is a local-only git index flag. It is never stored in the repository and does not affect forks or clones. Anyone who clones this repo starts with a clean state.

## The Committed Lock File

The `composer.lock` committed to the repository always resolves `nasirkhan/laravel-cube` and `nasirkhan/module-manager` from **Packagist** (not local paths), so CI can install dependencies without issues.

## How to Update `composer.lock` for the Repository

Follow these steps whenever you need to update `composer.lock` in the repo (e.g., adding or upgrading packages):

### Step 1 — Remove the skip-worktree flag
```bash
git update-index --no-skip-worktree composer.lock
```

### Step 2 — Hide `composer.local.json` temporarily
```bash
# Windows (PowerShell)
Rename-Item composer.local.json composer.local.json.bak

# macOS / Linux
mv composer.local.json composer.local.json.bak
```

### Step 3 — Update the package(s)
```bash
composer update <package-name>
# Example:
composer update fruitcake/laravel-debugbar
```

### Step 4 — Restore `composer.local.json`
```bash
# Windows (PowerShell)
Rename-Item composer.local.json.bak composer.local.json

# macOS / Linux
mv composer.local.json.bak composer.local.json
```

### Step 5 — Commit `composer.lock`
```bash
git add composer.lock
git commit -m "Update composer.lock: <describe what changed>"
```

### Step 6 — Re-apply skip-worktree
```bash
git update-index --skip-worktree composer.lock
```

### Step 7 — Restore local symlinks
```bash
composer update nasirkhan/laravel-cube nasirkhan/module-manager
```

This final step re-links the vendor packages back to your local paths so local development continues to work with live edits.

## Initial Setup for New Clones

When cloning this repository for local development with the local packages:

1. Ensure the sibling directories exist:
   ```
   laravel-starter-packages/
       laravel-cube/
       module-manager/
   ```

2. After `composer install`, run:
   ```bash
   composer update nasirkhan/laravel-cube nasirkhan/module-manager
   ```
   This will symlink the local packages via `composer.local.json`.

3. Apply skip-worktree to avoid accidentally committing path-based lock file changes:
   ```bash
   git update-index --skip-worktree composer.lock
   ```
