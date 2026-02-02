# **Deploy MkDocs Material Website to GitHub Pages (Step-by-Step)**
---


This document explains how to deploy an **MkDocs Material** website to **GitHub Pages** using the repository:


---

## 1. Prerequisites

Ensure the following are installed on your system:

```
git --version
mkdocs --version
```

If MkDocs Material is not installed:

```bash
pip install mkdocs-material
```

---

## 2. Project Structure (Expected)

Your project directory should look like this:

```text
Material/
├── docs/
│   ├── index.md
│   ├── assets/
│   │   └── Ayyappaswamy_Resume.pdf
│   └── ...
├── mkdocs.yml
└── .gitignore
```

> Note: MkDocs **only uses files inside the `docs/` directory**.

---

## 3. Navigate to Project Directory

```bash
cd ~/Material
```

---

## 4. Initialize Git Repository

If Git is not initialized yet:

```bash
git init
```

Add the GitHub remote repository:

```bash
git remote add origin https://github.com/hpcteam/hpcteam.github.io.git
```

Verify:

```bash
git remote -v
```

---

## 5. Create `.gitignore` File (Important)

Create a `.gitignore` file to avoid committing generated files:

```bash
cat <<EOF > .gitignore
site/
__pycache__/
*.pyc
.env
EOF
```

---

## 6. Commit Source Files

Add all files:

```bash
git add .
```

Commit:

```bash
git commit -m "Initial MkDocs Material website"
```

Rename branch to `main` and push:

```bash
git branch -M main
git push -u origin main
```

---

## 7. Deploy to GitHub Pages Using MkDocs

MkDocs provides a built-in deployment command.

Run:

```bash
mkdocs gh-deploy
```

This will:

* Build the site
* Create a `gh-pages` branch
* Push static files automatically

---

## 8. Enable GitHub Pages (One-Time Setup)

1. Open GitHub repository: **hpcteam/hpcteam.github.io**
2. Go to **Settings → Pages**
3. Under **Source**:

   * Branch: `gh-pages`
   * Folder: `/ (root)`
4. Click **Save**

Wait 1–2 minutes for deployment.

---

## 9. Access the Live Website

Your site will be available at:

```text
https://hpcteam.github.io
```

---

## 10. Updating the Website (Regular Workflow)

Whenever you modify content:

```bash
git add .
git commit -m "Updated documentation"
git push
mkdocs gh-deploy
```

---

## 11. Common Issues and Fixes

### Site Not Loading

* Wait a few minutes
* Verify GitHub Pages branch is `gh-pages`

### Theme or CSS Missing

```bash
pip install --upgrade mkdocs-material
mkdocs gh-deploy --force
```

### 404 Errors

* Ensure filenames match exactly (case-sensitive)
* Verify paths in `mkdocs.yml`

---

## 12. Summary

* `main` branch → source files
* `gh-pages` branch → deployed website
* `mkdocs gh-deploy` handles everything automatically


