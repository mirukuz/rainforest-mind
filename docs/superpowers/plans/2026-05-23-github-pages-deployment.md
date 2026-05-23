# GitHub Pages Deployment Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Deploy the Rainforest Mind Astro site to GitHub Pages at https://mirukuz.github.io/rainforest-mind with automatic CI/CD on every push to `master`.

**Architecture:** Astro's static build outputs to `dist/`. GitHub Actions picks up that artifact and deploys via the built-in GitHub Pages deployment mechanism. The `base` config ensures all asset paths are prefixed with `/rainforest-mind`.

**Tech Stack:** Astro 4.16.0, GitHub Actions, `withastro/action@v3`, `actions/deploy-pages@v4`

---

### Task 1: Configure Astro for GitHub Pages

**Files:**
- Modify: `astro.config.mjs`

- [ ] **Step 1: Add site and base to astro.config.mjs**

Replace the entire file content with:

```js
import { defineConfig } from 'astro/config';

export default defineConfig({
  site: 'https://mirukuz.github.io',
  base: '/rainforest-mind',
  server: {
    port: 3853,
  },
});
```

- [ ] **Step 2: Verify build works with new base path**

```bash
npm run build
```

Expected: Build completes successfully, `dist/` directory is created. No errors.

- [ ] **Step 3: Spot-check generated output has correct base path**

```bash
grep -r '/rainforest-mind' dist/ | head -5
```

Expected: You see references to `/rainforest-mind/` in the generated HTML/JS files.

- [ ] **Step 4: Commit**

```bash
git add astro.config.mjs
git commit -m "feat: configure Astro site and base for GitHub Pages deployment"
```

---

### Task 2: Create GitHub Actions Workflow

**Files:**
- Create: `.github/workflows/deploy.yml`

- [ ] **Step 1: Create the workflows directory**

```bash
mkdir -p .github/workflows
```

- [ ] **Step 2: Create deploy.yml**

```bash
cat > .github/workflows/deploy.yml << 'EOF'
name: Deploy to GitHub Pages

on:
  push:
    branches: [master]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - name: Setup Pages
        uses: actions/configure-pages@v5

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
EOF
```

- [ ] **Step 3: Validate YAML syntax**

```bash
python3 -c "import yaml; yaml.safe_load(open('.github/workflows/deploy.yml'))" && echo "YAML valid"
```

Expected: prints `YAML valid` with no errors.

- [ ] **Step 4: Commit**

```bash
git add .github/workflows/deploy.yml
git commit -m "feat: add GitHub Actions workflow for GitHub Pages deployment"
```

---

### Task 3: Create GitHub Repo and Push

**Files:** (no code files — git/GitHub operations)

- [ ] **Step 1: Authenticate gh CLI (if not already)**

```bash
gh auth status
```

If not logged in, run:
```bash
gh auth login
```
Choose GitHub.com → HTTPS → Login with a web browser, follow prompts.

- [ ] **Step 2: Create the public GitHub repository**

```bash
gh repo create rainforest-mind --public --source=. --remote=origin --description "Rainforest Mind showcase site"
```

Expected output: `✓ Created repository mirukuz/rainforest-mind on GitHub` and a URL.

- [ ] **Step 3: Push master branch**

```bash
git push -u origin master
```

Expected: Branch pushes successfully, GitHub Actions workflow triggers automatically.

- [ ] **Step 4: Enable GitHub Pages with GitHub Actions source**

```bash
gh api repos/mirukuz/rainforest-mind/pages \
  --method POST \
  --field build_type=workflow \
  --field source='{"branch":"master","path":"/"}' \
  2>/dev/null || \
gh api repos/mirukuz/rainforest-mind/pages \
  --method PUT \
  --field build_type=workflow
```

Expected: Returns JSON with `"build_type": "workflow"` — or a 409 if Pages was already configured (that's fine).

- [ ] **Step 5: Watch the workflow run**

```bash
gh run watch --repo mirukuz/rainforest-mind
```

Expected: Workflow completes with green checkmarks on both `build` and `deploy` jobs.

- [ ] **Step 6: Verify the live site**

Open in browser: https://mirukuz.github.io/rainforest-mind

Expected: Rainforest Mind homepage loads correctly with styles and images intact.

---

## Verification Summary

| Check | Command | Expected |
|-------|---------|----------|
| Build passes | `npm run build` | No errors, `dist/` created |
| Base path in output | `grep -r '/rainforest-mind' dist/ \| head -5` | Hits found |
| Workflow YAML valid | `python3 -c "import yaml; ..."` | `YAML valid` |
| GitHub Actions green | `gh run watch` | Both jobs pass |
| Live site loads | Browser → URL | Page renders correctly |
