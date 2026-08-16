# SDE-2 Prep

Personal Obsidian vault for SDE-2 interview preparation — concurrency, Java, Spring Boot, system design (HLD/LLD), DSA, production debugging, and per-company interview logs.

## Structure

- `Concurrency/` — threads, race conditions, locks, atomics, virtual threads, etc.
- `DSA/` — data structures & algorithms practice and notes
- `HLD/` — high-level system design
- `LLD/` — low-level design
- `Java/` — core Java concepts
- `Spring Boot/` — framework-specific notes
- `Production Debugging/` — real-world debugging scenarios and playbooks
- `System Design/` — general system design notes
- `Company Interview Logs/` — per-company interview prep and post-interview notes

## ⚠️ Note on GitHub formatting

This vault is written for **Obsidian**, not for GitHub's markdown renderer. Callouts (`> [!warning]`, `> [!tip]`, etc.), wikilinks (`[[note name]]`), and embedded diagrams (`![[diagram.svg]]`) will **not render correctly on GitHub** — callouts show up as plain blockquotes, wikilinks show as literal double-bracketed text, and embeds won't display at all. To view this vault as intended, open it in Obsidian locally (see below).

## Using this vault in Obsidian

1. **Install Obsidian** — download from [obsidian.md](https://obsidian.md) for your OS (Windows/macOS/Linux) and install it.
2. **Clone this repo:**
   ```bash
   git clone https://github.com/abhishekk7r/obsidian-sde-2-prep.git
   ```
3. **Open it as a vault in Obsidian:**
   - Launch Obsidian
   - Click **"Open folder as vault"** on the startup screen (or **File → Open Vault** if Obsidian is already running)
   - Select the cloned `obsidian-sde-2-prep` folder
4. Obsidian will detect the existing `.obsidian` config folder in the repo and load the vault with its notes, links, and settings intact — callouts, wikilinks, and embedded diagrams will render correctly.
5. To keep local changes in sync with GitHub, commit and push from the vault folder as with any git repo:
   ```bash
   git add .
   git commit -m "update notes"
   git push
   ```
