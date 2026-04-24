# Archon Module Template

Starter repo for building a new Archon pipeline module.

Use the green **"Use this template"** button at the top of this page to create a new module repo. The new repo inherits everything needed for Archon workflows to run.

## What's included

- `.archon/scripts/` — pipeline helper scripts (compliance gate, checkpoint, lint, etc.) used by PRD Pipeline C and related workflows
- Initial commit on `main` branch (required for Archon worktree creation)

## How to use

1. Click **"Use this template"** → **"Create a new repository"**
2. Name the new repo (e.g. `scraper`, `detection-bot`, `landing-pages`)
3. Create the repo
4. Copy its GitHub URL
5. In Archon UI, paste the URL in the "GitHub URL or local path" box
6. Select the new project in the dropdown
7. Run your workflow

No terminal commands needed. The template handles all the Archon-specific setup.

## What NOT to do

- Don't edit `.archon/scripts/` unless you know what you're doing — these files are shared pipeline infrastructure.
- Don't delete the initial commit. Archon needs at least one commit on `main` to create worktrees.
