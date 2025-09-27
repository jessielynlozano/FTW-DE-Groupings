
# FTW Data Engineering Journal Template

A minimal learning journal for a **4-Saturday Data Engineering Bootcamp**.  

Use it to capture learnings, reflections, vocabulary, and the **DE mindset** — especially for documenting, communicating, and presenting.

---

## 🚀 Getting Started

Click **Use this template** (top-right on GitHub) to create your own copy.  
Or clone and rename:

```bash
git clone https://github.com/ogbinar/ftw-de-journal.git
````

---

## 📝 How to Use

1. **Create an entry** for each session:

   * Copy `templates/entry.md` to `journal/YYYY-MM-DD-topic.md`
   * Example: `journal/2025-09-13-day1.md`

2. **Write during/after class** (10–15 mins is fine).

3. **Commit small & often**:

   ```bash
   git add . 
   git commit -m "Add Day 1 notes"
   git push
   ```
or you can also use [github.dev](https://docs.github.com/en/codespaces/the-githubdev-web-based-editor).

4. **Present from your journal** in standups/demos.

---

## 📚 Learn Markdown

This journal uses [Markdown](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax) — a simple way to format text on GitHub.
Learning Markdown will make your notes cleaner and easier to read (headings, bullets, tables, code blocks, etc.).

Quick examples:

```markdown
# Heading 1
## Heading 2
- Bullet point
- **Bold** and *italic*
`inline code`
```

---

## 📂 Suggested File Naming

```
journal/
  2025-09-13-day1.md
  2025-09-20-day2.md
  2025-09-27-day3.md
  2025-10-04-day4.md
```

---

## 🧠 Data Engineering Mindset (keep visible)

* **Raw stays raw** → Clean → **Mart** (separate layers).
* **Reproducible**: scripts/configs over manual steps.
* **Document assumptions & decisions** (why, not just what).
* **Communicate clearly**: inputs, outputs, owners, SLAs.
* **Small changes**: atomic commits; test + verify.
* **Observe & log**: data quality, lineage, and failure points.

---

## 🎤 Presenting with This Journal

* Start with **context** (dataset, goal, constraints).
* Show a **diagram** (even a quick sketch) of flow (Raw → Clean → Mart).
* Call out **trade-offs** (cost, freshness, complexity).
* End with **next steps** + **open questions**.
