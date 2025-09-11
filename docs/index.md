# 30sundays Tech Docs

Welcome to the **30sundays Technical Documentation** 📚  
This site is the central place for documenting all our projects, APIs, workflows, and engineering practices.

---

## 🔹 How to Contribute Documentation

All documentation is written in **Markdown (`.md`)** files inside the `docs/` folder.  
Every file you add here becomes a page in the site automatically when we build with MkDocs.

### Workflow for Developers

1. **Clone the repository**
   ```bash
   git clone https://github.com/<org>/<repo>.git
   cd <repo>
   ```

2. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

3. **Run locally**
   ```bash
   mkdocs serve
   ```
   Open http://127.0.0.1:8000 in your browser to preview docs with live reload.

4. **Add or edit documentation**
   - Write content in Markdown (`.md`) format.
   - Place your files in the correct folder (see structure below).
   - Use code blocks for examples, e.g.:
   ```python
   def example():
       print("Hello, Docs!")
   ```

5. **Commit and push**
   ```bash
   git checkout -b docs/update-auth
   git add docs/api/authentication.md
   git commit -m "docs(api): add authentication docs"
   git push origin docs/update-auth
   ```
   Then open a Pull Request for review.

## 📂 Project Documentation Structure

We follow this folder layout under `docs/`:

```
docs/
│
├── index.md                # Homepage (this page)
│
├── modules/                 
│   ├── lead-creation            
│   ├── mobile-app      
│   └── itnerary-builder  
│
└── assets/                 # Images, diagrams, screenshots
    └── architecture.png
```

## 🛠️ Useful Commands

- `mkdocs serve` – Start the live-reloading docs server.
- `mkdocs build` – Build the static documentation site into `site/`.
- `firebase deploy` – Deploy to Firebase Hosting (if configured).

## ✅ Best Practices

- Keep docs close to the code they describe.
- Use clear file names (`auth.md`, `setup.md`) for easy navigation.
- Use screenshots, diagrams, and examples wherever helpful.
- Always create a Pull Request for changes so others can review.