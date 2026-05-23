# GitHub Pages Deployment Design

**Date:** 2026-05-23  
**Project:** rainforest-mind (Astro 4.16.0 static site)  
**Target URL:** https://mirukuz.github.io/rainforest-mind

## Overview

Deploy the Rainforest Mind Astro site to GitHub Pages using GitHub Actions for automatic CI/CD. Every push to `master` triggers a build and deploy.

## Changes Required

### 1. astro.config.mjs — Add site and base

Astro needs to know the deployment URL so it can generate correct asset paths and links:

```js
site: 'https://mirukuz.github.io',
base: '/rainforest-mind',
```

### 2. .github/workflows/deploy.yml — GitHub Actions workflow

Use the official `withastro/action` which handles Node setup, dependency install, build, and upload to GitHub Pages artifact:

- Trigger: push to `master`
- Permissions: `contents: read`, `pages: write`, `id-token: write`
- Concurrency: cancel in-progress deploys on new push
- Steps: checkout → setup pages → install deps → build → upload artifact → deploy

### 3. GitHub repo creation and push

- Create public repo `rainforest-mind` under `mirukuz` using `gh repo create`
- Add remote and push `master` branch
- In GitHub repo Settings → Pages → Source: set to "GitHub Actions"

## Architecture

```
push to master
    ↓
GitHub Actions workflow
    ↓
withastro/action (build → dist/)
    ↓
Upload as GitHub Pages artifact
    ↓
deploy-pages action → live at mirukuz.github.io/rainforest-mind
```

## Verification

After deployment:
- Visit https://mirukuz.github.io/rainforest-mind and confirm the page loads
- Check that CSS/JS assets load correctly (no 404s from wrong base path)
- Confirm GitHub Actions tab shows a green workflow run
