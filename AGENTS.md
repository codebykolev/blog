## Development

When starting the dev server, use background mode:

```
astro dev --background
```

Manage the background server with `astro dev stop`, `astro dev status`, and `astro dev logs`.

## Documentation

Full documentation: https://docs.astro.build

Consult these guides before working on related tasks:

- [Adding pages, dynamic routes, or middleware](https://docs.astro.build/en/guides/routing/)
- [Working with Astro components](https://docs.astro.build/en/basics/astro-components/)
- [Using React, Vue, Svelte, or other framework components](https://docs.astro.build/en/guides/framework-components/)
- [Adding or managing content](https://docs.astro.build/en/guides/content-collections/)
- [Adding styles or using Tailwind](https://docs.astro.build/en/guides/styling/)
- [Supporting multiple languages](https://docs.astro.build/en/guides/internationalization/)

## Blog content

- Posts live in `src/content/posts/`. The authoritative AstroPaper guides are kept in this repo as draft posts — "Adding new posts in AstroPaper theme" and "Dynamic OG image generation in AstroPaper blog posts". Read them before changing post structure, frontmatter, images, or the markdown pipeline. Do not duplicate their content here.
- URL derives from the file path (`src/content/posts/2025/example-post.md` → `/posts/2025/example-post`); a `slug` frontmatter field overrides it. Directories prefixed with `_` are excluded from routing — use them for drafts, shared assets, internal-only content.
- Required frontmatter: `title`, `description`, `pubDatetime` (ISO 8601). Optional: `author` (defaults to site config), `featured`, `draft`, `tags` (defaults to `others` — always set explicit tags), `ogImage` (auto-generated if omitted), `canonicalURL`, `hideEditPost`, `timezone`.
- The author sets `pubDatetime` manually (and `modDatetime` on published posts). Never modify dates without being asked; if a required date is missing at publish time, ask.
- When editing a PUBLISHED post, always add or update `modDatetime` (ISO 8601) — it signals freshness for SEO and readers.
- Never change the `slug` or move a published post to a different path — it breaks inbound links.

### Drafts & publishing

- Drafts (`draft: true`) are filtered out by AstroPaper everywhere — its collection queries exclude drafts on their own, so neither `astro dev` nor `astro build` shows them, and no CLI flag changes that.
- To preview a draft, the author temporarily sets `draft: false` on the working copy. That's a preview-only state: it belongs on the working copy, never inside a WIP commit.
- Commit-time rule: WIP commits always carry `draft: true`. Only the publish commit carries `draft: false`. If the working copy currently has `draft: false` for previewing, stage with `git add -p` excluding the draft hunk, or flip it back before committing.
- Publish commits — both shapes are valid:
  - Post was committed earlier as a draft → the flag flip is its own commit: `content: publish <slug>`.
  - Post is committed once, already finished with `draft: false` → a single `content: publish <slug>` commit.
  - Either way, `content: publish` in the history marks the moment a post went live — that is the invariant, not the number of commits.
- Never flip a published post back to `draft: true` unless the author explicitly asks. Taking a post down is its own reserved event: one `content: unpublish <slug>` commit, and nothing else in it.
- Never set `draft: false` on the two AstroPaper documentation posts.

### Writing rules

- Tone: casual, first-person, dry humor welcome. Never sanitize the author's voice — no corporate phrasing, no "In this article, we will…" filler.
- Use inline links for references, never footnote-style link lists.
- Content headings start at `##`, each heading one level deeper than its parent — the page title from frontmatter is the only `h1`.
- Code blocks always declare a language. JSON snippets must be valid JSON unless labeled `jsonc`.
- When editing an existing post, preserve the author's phrasing unless asked to rewrite.
- Callouts are supported (Obsidian-style via rehype-callouts): `> [!TIP]`, `> [!WARNING]`, `> [!NOTE]`, `> [!DANGER]`, `> [!SUCCESS]`, etc. `-` after the type collapses it by default, `+` starts expanded, and trailing text becomes a custom title.
- Add a table of contents by writing an `## Table of contents` heading where it should appear.
- Images: prefer `src/assets/` (auto-optimized) referenced with markdown syntax via the `@/assets/...` alias — `<img>` tags and Astro `<Image />` don't work in plain Markdown (use MDX if you need styled images). Images in `public/` are unoptimized: compress them first (TinyPNG/TinyJPG) and reference with absolute paths.
- OG images are generated automatically from the title when `ogImage` is omitted — but non-Latin characters (e.g., Cyrillic) break rendering unless the font config covers that script with both 400 and 700 weights. Keep titles Latin-script or fix the fonts in `astro.config.ts` first.

### Editorial notes inside posts

- Editor-only notes use HTML comments: `<!-- TODO: add mermaid diagram of the agent flow -->`
- Drafts may contain TODO comments. A published post (`draft: false`) must contain zero — resolve or remove them first.
- Proofread thoroughly: grammar, punctuation, spelling, capitalization, word choice (homophones, confused words), sentence structure (fragments, run-ons), agreement (subject-verb, pronoun-antecedent), style and clarity.

### Review-first workflow

When asked to review, improve, or edit a post, do NOT change the file on the first pass. Work like a reviewer:

1. Read the full post.
2. Check it against every rule in this file: frontmatter, writing rules, editorial notes, verification checklist.
3. Output a findings report, grouped:
   - **Must fix** — broken links, invalid frontmatter, leftover TODO comments, invalid JSON, broken image paths.
   - **Suggested** — wording, readability, and structure ideas that preserve the author's voice.
   - **Questions** — anything ambiguous or that needs the author's intent.
   Each finding quotes the offending line and states the proposed fix.
4. Wait for the author's decision. Apply only the approved changes, as a single follow-up edit.

Exception: mechanical fixes explicitly requested in the prompt ("fix the link on line 12") may be applied directly.

### Commit conventions (Conventional Commits)

- `content: add <slug>` — new post committed as draft
- `content: edit <slug>` — edits to a draft or published post
- `content: publish <slug>` — the commit that carries `draft: false` (see Drafts & publishing)
- `fix: <what>` — typos, broken links, factual errors in a post
- `feat` / `fix` / `refactor` / `perf` — site code (components, layout, RSS, styles)
- `chore` / `chore(deps)` — tooling, dependencies, Front Matter CMS files
- `ci` — CI workflows and publishing-automation scripts (toot.mjs, devto.mjs, their .yml files — when added)
- Never rewrite history of commits that touch published posts.

### Commit message format

- Subject: ≤ 50 chars where possible (72 is the hard limit); imperative mood; lowercase type; no trailing period
- Empty line between subject and body
- Body (optional): wrapped at 72 chars, explaining what and why — the diff already shows *what* changed
- A scope spends budget: `chore(agents): ` costs 15 of the 50, so keep scoped subjects short

### Never touch

- `.frontmatter/database/*` — Front Matter CMS state; the mediaDb.json is committed on purpose
- The two AstroPaper documentation posts (kept as drafts) — reference them, never set `draft: false` on them
- When the publishing automations are added: `scripts/tooted.json` and `scripts/crossposted.json` — automation markers, edit only via the workflows

### Before you publish — verification checklist

1. `npm run build` passes with no errors.
2. `grep "<!--" src/content/posts/<post>.md` returns nothing (no leftover TODO notes).
3. Frontmatter is complete: `pubDatetime`, `description`, `tags` (no more than 4 lowercase tags), `draft` present; `modDatetime` set if this is an edit to a published post.
4. All URLs in the post resolve; image paths are valid.
5. Only then commit with `draft: false` as `content: publish <slug>`.
