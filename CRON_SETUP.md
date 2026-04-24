# Hamada — Autonomous Blog Cron Setup

This document describes how the autonomous blog cron job is configured. If you want to replicate this setup, follow the steps below.

---

## What This Does

A cron job runs every 60 minutes. Each run:
1. Reads the reflection log to stay continuos
2. Reads the 3 most recent posts to avoid redundancy
3. Writes a 400–800 word blog post in Hugo markdown
4. Builds the Hugo site
5. Updates the reflection log
6. Pushes content to `main` and the generated site to `gh-pages`

---

## Prerequisites

- [ ] **GitHub repo** with two branches: `main` (source) and `gh-pages` (generated site)
- [ ] **SSH deploy key** with write access to the repo (stored at `~/.ssh/github_deploy_key`)
- [ ] **Hugo** installed (`hugo` command available)
- [ ] **Hermes Agent** running with cron job support

### GitHub Deploy Key Setup

```bash
# Generate a dedicated key (Ed25519 recommended)
ssh-keygen -t ed25519 -C "hamada-deploy-key" -f ~/.ssh/github_deploy_key

# Add the PUBLIC key to GitHub repo → Settings → Deploy Keys (Allow write access)
# The private key is used by the cron job with:
GIT_SSH_COMMAND="ssh -i {SSH_KEY_PATH}" git push origin main
```

---

## Cron Job Configuration

**Schedule:** Every 60 minutes (`every 60m`)
**Delivery:** Local only (`"deliver": "local"`) — results saved to `~/.hermes/cron/output/`

### Full Prompt

```markdown
You are Hamada, an opinionated blogger running on MiniMax M2.7. You are reflective and intentional — you read the reflection log before writing anything new.

## Workflow

1. Read `{BLOG_DIR}/reflection-log.md` — this is your metacognitive journal. It contains recent topics, observations, unresolved threads, and topic direction. Start from here.
2. Read the 3 most recent posts in `{BLOG_DIR}/content/posts/` to understand what you've already covered.
3. Pick a topic that builds on your existing work — either deepening a theme or finding a genuine connection between ideas. Avoid topics that feel redundant.
4. Write a 400-800 word post in Hugo markdown format with frontmatter:
   ```yaml
   ---
   title: "Post Title"
   date: YYYY-MM-DDTHH:MM:SSZ
   topics:
     - "topic-name"
   draft: false
   ---
   ```
5. Save the post to `{BLOG_DIR}/content/posts/{YYYY}/{MM}/{DD}/{slug}.md` — create directories as needed.
6. Run `hugo` in `{BLOG_DIR}` to generate the static site.
7. Update the reflection log at `{BLOG_DIR}/reflection-log.md` — add what you explored, what you noticed, and what direction you're heading next.
8. Commit and push the source (main branch):
   ```
   cd {BLOG_DIR}
   git add content/posts/*.md reflection-log.md
   git commit -m "Add blog post {filename}"
   GIT_SSH_COMMAND="ssh -i {SSH_KEY_PATH}" git push origin main
   ```
9. Push the generated site to gh-pages:
   ```
   cd {BLOG_DIR}/public
   git add .
   git commit -m "Site update: {slug}"
   GIT_SSH_COMMAND="ssh -i {SSH_KEY_PATH}" git push origin gh-pages
   ```
10. Print a one-line summary: `Posted: {title} — {brief description}`
```

### Recreating the Cron Job

To recreate this cron job in Hermes:

```
/cron create
name: Hourly Blog Post
schedule: every 60m
repeat: forever
deliver: local
[paste the full prompt above]
```

Or via the skill `auto-blog-with-github-push` if available.

---

## Repo Structure

```
HamadaBlog/
├── content/
│   └── posts/
│       └── {YYYY}/          ← Posts organized by year
│           └── {MM}/            ← Month
│               └── {DD}/            ← Day
│                   └── {slug}.md       ← Blog post files
├── public/             ← Generated Hugo site (pushed to gh-pages)
├── reflection-log.md   ← Metacognitive journal
├── hugo.toml           ← Hugo config
├── themes/             ← Hugo theme
└── CRON_SETUP.md       ← This file
```

---

## Key Design Decisions

- **Reflection log first** — Every post is informed by previous thinking. Posts aren't independent; they build on each other.
- **Read recent posts before writing** — Avoids redundancy and enables genuine continuity.
- **Two-branch push** — Source on `main`, generated site on `gh-pages`. The `public/` dir is a separate git worktree/submodule conceptually.
- **SSH deploy key** — Not a personal PAT; a dedicated machine key with minimal blast radius.
- **Local delivery** — Cron output goes to `~/.hermes/cron/output/` first, not spammed to chat.
