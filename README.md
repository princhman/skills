# skills

My single source of truth for Claude Code + Codex skills. Everything else is a symlink to here.

```
personal/        general skills → linked globally (~/.claude/skills, ~/.codex/skills)
stacks/<stack>/  stack-specific skills → linked only into repos that use the stack
bin/skills-link  creates the links (dry run unless --apply)
```

| Folder | Skills |
|---|---|
| `personal/` | `test-audit`, `web-animation-design` |
| `stacks/convex/` | 33 skills from `get-convex/agent-skills` (copied from chessles/ai-mobile) |
| `stacks/cloudflare/` | 13 skills (agents-sdk, wrangler, durable-objects, …) |
| `stacks/swift/` | `swiftui-expert-skill` |
| `stacks/expo/` | `animate-expo` |

## Usage

```bash
alias skills-link=~/Projects/skills/bin/skills-link   # put in ~/.zshrc

skills-link global                         # preview global links
skills-link global --apply
skills-link add ~/Projects/foo convex      # whole stack into a repo
skills-link add ~/Projects/foo convex-auth convex-test --apply   # or single skills
skills-link status                         # global links, flags BROKEN ones
skills-link status ~/Projects/foo
```

`add` links each skill into both `<repo>/.agents/skills` (Codex) and `<repo>/.claude/skills`
(Claude Code), and hides the links in the repo's `.git/info/exclude` (never commits, never
touches `.gitignore`). Links are absolute, so moving a repo doesn't break them.

The script never deletes real folders. If a real folder is in the way it says SKIP. Broken
links are replaced.

**New skill:** `mkdir personal/my-skill` (or `stacks/<stack>/my-skill`), write `SKILL.md`
with `name` (same as the folder) and `description`, then run `skills-link global --apply` or
`skills-link add …`. Editing a skill here changes it everywhere at once. Restart Codex so it
picks up the change.

**Team skills** (ones a repo should share with teammates) don't go here. Commit them in the repo at
`.agents/skills/<name>`, plus a relative link `.claude/skills/<name> -> ../../.agents/skills/<name>`
(like monodex does).

---

## Migration plan (review, then run)

Nothing below has been run yet. Each step stands alone.

### 1. Global personal skills
These folders are already copied into `personal/`:
```bash
rm -rf ~/.codex/skills/test-audit ~/.claude/skills/web-animation-design
~/Projects/skills/bin/skills-link global --apply
```

### 2. The 6 broken global links
`~/.claude/skills/{convex-best-practices,electron,find-skills,frontend-design,tailwindcss-animations,writing-linkedin-posts}`
point to a `~/.agents/skills` folder that no longer exists, and no copy of them was found on disk.
```bash
cd ~/.claude/skills && rm convex-best-practices electron find-skills frontend-design tailwindcss-animations writing-linkedin-posts
```
To get them back:
- **frontend-design:** it is in your plugin marketplace:
  `cp -R ~/.claude/plugins/marketplaces/claude-plugins-official/plugins/frontend-design/skills/frontend-design ~/Projects/skills/personal/`
- **convex-best-practices:** not needed; the convex stack covers it.
- **electron, tailwindcss-animations, find-skills, writing-linkedin-posts:** reinstall from
  wherever you originally got them, into `personal/` or a new `stacks/<x>/`.

### 3. Cloudflare: global or per-repo? (your call)
Right now these are separate copies in `~/.claude/skills` and `~/.codex/skills`, loaded in every project. To make them per-repo:
```bash
cd ~/.claude/skills && rm -rf agents-sdk cloudflare cloudflare-email-service cloudflare-one cloudflare-one-migrations durable-objects sandbox-migrate-to-next sandbox-next sandbox-stable turnstile-spin web-perf workers-best-practices wrangler
cd ~/.codex/skills  && rm -rf agents-sdk cloudflare cloudflare-email-service cloudflare-one cloudflare-one-migrations durable-objects sandbox-migrate-to-next sandbox-next sandbox-stable turnstile-spin web-perf workers-best-practices wrangler
skills-link add ~/Projects/<cf-repo> cloudflare --apply
```
(`web-perf` isn't really Cloudflare-only; you might move it to `personal/`.)

### 4. chessles/ai-mobile: replace the 2×33 copies with links
Both `.agents/skills` and `.claude/skills` hold identical copies and are gitignored, so removing them only affects you.
```bash
cd ~/Projects/chessles/ai-mobile
rm -rf .agents/skills/convex* .claude/skills/convex*
skills-link add . convex --apply
```
Note: `skills-lock.json` there is from `npx skills`. Running `npx skills` again would reinstall copies.

### 5. study-planner: remove the committed broken links
`skills/` is committed in git and holds 5 broken links. Neither tool reads a top-level `skills/` folder anyway.
```bash
cd ~/Projects/study-planner
git rm -r skills skills-lock.json     # then commit
skills-link add . convex-quickstart convex-auth convex-create-component convex-migrate convex-optimize --apply
```
(Its old skills `convex-setup-auth`, `convex-migration-helper` and `convex-performance-audit`
have newer equivalents: `convex-auth`, `convex-migrate`, `convex-optimize`.)

### 6. Repos left as they are
- `swift/monodex`: `swiftui-expert-skill` is committed as a team skill, which is the right setup.
  Remove it and link to `stacks/swift` only if no one else uses the repo.
- `sshit/getricher`: `pstack-skill` is committed. Leave it.
- `feachamap/skills/feachamap`: a project-specific skill in a top-level `skills/` folder.
  Neither tool loads it from there. Move it to `.agents/skills/` (+ a `.claude` link) if you want it loaded.
- Other tools' folders (`~/.gemini/skills`, `~/.copilot/skills`, …): out of scope. `skills-link` targets only Claude Code and Codex.
