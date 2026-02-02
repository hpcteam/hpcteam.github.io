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


Perfect 👍
Below is a **clean, copy-paste ready Markdown document** you can save as, for example:

```bash
docs/Documents/How_To_Add_New_Page_Auto_Deploy.md
```

# How to Create or Update Pages in MkDocs (Auto Deployment via GitHub Actions)

This document explains **how to create new pages, edit existing pages, and how changes are automatically deployed** to the live website using **GitHub Actions CI/CD**.

Repository:
- https://github.com/hpcteam/hpcteam.github.io  
Live site:
- https://hpcteam.github.io

---

## 1. How Auto Deployment Works (Simple Flow)

Whenever you push changes to the `main` branch, the following happens automatically:

```

Edit Markdown File
↓
Git Commit
↓
Git Push
↓
GitHub Actions CI/CD
↓
MkDocs Build (--strict)
↓
Website Updated Automatically

```

⚠️ You **do not need to run** `mkdocs gh-deploy` manually.

---

## 2. Creating a New Page (Step-by-Step)

### Example: Creating a New Page called `Slurm Basics`

---

### Step 1: Create a New Markdown File

In VS Code:

1. Open the project folder
2. Navigate to:
```

docs/Documents/

```
3. Create a new file:
```

Slurm.md

````

Add content:

```markdown
# Slurm Basics

Slurm is a workload manager used in HPC clusters.

## Common Commands
```bash
sinfo
squeue
sbatch
````

````

Save the file.

---

### Step 2: Add the Page to Navigation (MANDATORY)

Open `mkdocs.yml`.

Under the `Documents:` section, add:

```yaml
- Slurm Basics: Documents/Slurm.md
````

⚠️ Important:
Because CI uses `mkdocs build --strict`, **every page must be listed in `nav`**.

---

### Step 3: Test Locally (Optional but Recommended)

```bash
mkdocs serve
```

Open in browser:

```
http://127.0.0.1:8000
```

Verify:

* Page loads correctly
* Navigation entry appears

---

### Step 4: Commit Changes

Using terminal:

```bash
git add docs/Documents/Slurm.md mkdocs.yml
git commit -m "Add Slurm Basics documentation"
```

OR using VS Code Source Control:

* Commit with message

---

### Step 5: Push Changes

```bash
git push
```

---

### Step 6: Automatic Deployment

After push:

* GitHub Actions runs automatically
* MkDocs site is rebuilt
* Website updates in ~30–60 seconds

Visit:

```
https://hpcteam.github.io
```

---

## 3. Editing an Existing Page

Example: Editing `Linux_commands.md`

Steps:

1. Open file in VS Code
2. Modify content
3. Save
4. Commit
5. Push

Deployment happens automatically.

---

## 4. What Will Break the Deployment (IMPORTANT)

Because `--strict` mode is enabled, the build will **FAIL** if:

* A Markdown file exists but is **not listed in `nav`**
* A file is listed in `nav` but **does not exist**
* Filename case does not match (`Linux.md` ≠ `linux.md`)
* Temporary or test files are left in `docs/`

---

## 5. Best Practices (Recommended)

* Always add new pages to `mkdocs.yml`
* Avoid `test.md` files in production
* Use clear, descriptive filenames
* Run `mkdocs serve` before pushing if possible
* Keep navigation clean and structured

---

## 6. Daily Workflow (Quick Reference)

```bash
# Edit or add markdown files
git add .
git commit -m "Update documentation"
git push
```

The website updates automatically.

---

## 7. Summary

* GitHub Actions handles deployment
* Every push to `main` triggers a rebuild
* No manual deployment steps required
* Strict mode ensures clean, error-free documentation

This workflow follows **industry-standard DevOps documentation practices**.

```

---


