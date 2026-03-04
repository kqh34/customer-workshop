````skill
---
name: changelog-updater
description: Update CHANGELOG.md whenever the user commits or is about to commit changes. Use this skill when the user says "commit", "update the changelog", "write a changelog entry", "what should I add to the changelog", or mentions finishing a feature, fix, or refactoring. Proactively trigger this skill when code changes have been staged or a commit message is being composed. Always use this skill rather than manually writing changelog entries.
---

# Changelog Updater

This skill automates maintaining a `CHANGELOG.md` at the repository root following the [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format and [Semantic Versioning](https://semver.org/).

## When to use

- User says "commit", "I'm done with this feature", "update the changelog", or similar
- User has staged changes and wants to commit
- User asks to summarize what changed in a PR or branch
- User asks to add a changelog entry for something they just built

## Workflow

### Step 1: Understand what changed

Run the following to gather context (prefer staged changes if available, fall back to recent commit):

```bash
# Staged changes (pre-commit)
git diff --cached --stat
git diff --cached

# OR if changes are already committed
git show --stat HEAD
git show HEAD
```

Also check the commit message if the user provided one — it contains intent that may not be obvious from the diff.

### Step 2: Read the current CHANGELOG.md

Read `CHANGELOG.md` from the repository root. If it does not exist yet, create it from scratch using the template below.

Look for:
- The `[Unreleased]` section (entries go here before a release)
- The most recent version tag to understand the current version baseline
- Existing entries to avoid duplicates

### Step 3: Categorize the changes

Map diff content to Keep a Changelog categories:

| Category | When to use |
|---|---|
| `Added` | New feature, endpoint, component, file, or capability |
| `Changed` | Modified behavior, updated dependency, renamed/moved code |
| `Deprecated` | Something still works but is flagged for removal |
| `Removed` | Deleted feature, route, component, or file |
| `Fixed` | Bug fix, corrected logic, error handling improvement |
| `Security` | Vulnerability patch, auth fix, input validation |

Use your judgment — one diff may produce entries in multiple categories. Omit categories with no changes.

### Step 4: Write the entries

Each entry should be:
- A single sentence starting with a past-tense verb (e.g., "Added", "Fixed", "Removed")
- Specific enough to be useful ("Added supplier status filter endpoint" > "Updated suppliers")
- Free of implementation details that don't matter to users ("Fixed off-by-one in SQL query" is fine; "changed `i < n` to `i <= n` in loop" is not)

**Example entries:**
```
### Added
- Added `GET /api/suppliers?status=active` endpoint for filtering suppliers by status

### Fixed  
- Fixed 500 error when creating an order with a non-existent product ID

### Changed
- Updated React `Products` component to use server-side pagination
```

### Step 5: Update CHANGELOG.md

Insert the new entries under `## [Unreleased]`. If that section has no existing subsections, add them. If it already has entries in the same category, append to that list — do not create a duplicate header.

Keep entries in reverse-chronological order within each category (newest first within a single editing session is fine — don't re-sort existing entries).

### Step 6: Confirm with the user

Briefly summarize what you added to the changelog and offer to adjust the entries or the commit message.

---

## CHANGELOG.md Template (for new files)

If `CHANGELOG.md` doesn't exist, create it with this structure:

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

### Changed

### Fixed

## [0.1.0] - YYYY-MM-DD

### Added
- Initial release
```

Replace `YYYY-MM-DD` with today's date.

---

## Tips for this codebase

This is a TypeScript monorepo (`api/` + `frontend/`). When categorizing changes:

- **`api/src/routes/`** changes → likely `Added` or `Changed` (new/modified endpoints)
- **`api/src/repositories/`** changes → likely `Changed` or `Fixed` (data access layer)
- **`database/migrations/`** changes → likely `Added` or `Changed` (schema changes)
- **`frontend/src/components/`** changes → likely `Added` or `Changed` (UI)
- **`package.json`** dependency bumps → `Changed` (if minor/patch) or `Security` (if CVE-related)
- Test-only changes (`.test.ts`) → typically omit from changelog unless a test infrastructure change

When a migration file is added, mention the schema change (e.g., "Added `status` and `status_updated_at` columns to `suppliers` table") rather than just referencing the migration file name.
````
