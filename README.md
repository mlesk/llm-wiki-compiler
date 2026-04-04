# LLM Wiki Compiler

A Claude Code plugin that compiles scattered markdown knowledge files into a topic-based wiki. Reduce context costs by ~90% and get instant answers from synthesized knowledge.

## Inspiration

This plugin implements the **LLM Knowledge Base** pattern described by [Andrej Karpathy](https://x.com/karpathy/status/2039805659525644595):

> *"Raw data from a given number of sources is collected, then compiled by an LLM into a .md wiki, then operated on by various CLIs by the LLM to do Q&A and to incrementally enhance the wiki, and all of it viewable in Obsidian. You rarely ever write or edit the wiki manually, it's the domain of the LLM."*

The key insight: instead of re-reading hundreds of raw files every session, have the LLM compile them into topic-based articles once, then query the synthesized wiki. Knowledge compounds instead of fragmenting.

## What It Does

You have 100+ markdown files across meetings, strategy docs, session notes, and research. Every Claude session re-reads them. This plugin compiles them into topic-based articles that synthesize everything known about each subject — with backlinks to sources.

**Before:** Read 13+ raw files (~3,200 lines) per session
**After:** Read INDEX + 2 topic articles (~330 lines) per session

## Install

### From GitHub

```bash
# 1. Clone the repo
git clone https://github.com/ussumant/llm-wiki-compiler.git

# 2. Add as a local marketplace
claude plugin marketplace add /path/to/llm-wiki-compiler

# 3. Install the plugin
claude plugin install llm-wiki-compiler

# 4. Restart Claude Code for hooks to register
```

### For a Single Session (no install)

```bash
claude --plugin-dir /path/to/llm-wiki-compiler/plugin
```

## Quick Start

```bash
# 1. Open your project in Claude Code and run:
/wiki-init

# This will:
# - Auto-detect markdown directories in your project
# - Ask you which ones to include
# - Create .wiki-compiler.json config
# - Set up the output directory

# 2. Compile your knowledge base
/wiki-compile

# First run reads all source files and creates topic articles.
# Takes 5-10 minutes depending on source count.
# Subsequent runs are incremental (only changed files).

# 3. Browse in Obsidian
# Open wiki/INDEX.md to see all topics with backlinks

# 4. Query it
/wiki-query what do we know about retention?
```

## How It Works

### Three Commands

| Command | Purpose |
|---------|---------|
| `/wiki-init` | Interactive setup — auto-detects your markdown directories, creates config |
| `/wiki-compile` | Compiles source files into topic articles (incremental — only recompiles changes) |
| `/wiki-query` | Quick Q&A against the compiled wiki |

### Staged Adoption (The Key Feature)

The plugin never modifies your existing CLAUDE.md or AGENTS.md. Instead, it injects context via a SessionStart hook with three modes:

| Mode | What Happens | Your Existing Setup |
|------|-------------|-------------------|
| **staging** (default) | "Wiki available — check it when you need depth" | Completely unchanged |
| **recommended** | "Check wiki articles before raw files" | Unchanged, but Claude prioritizes wiki |
| **primary** | "Wiki is your primary knowledge source" | You can optionally simplify startup reads |

Change mode by editing `.wiki-compiler.json`:
```json
{ "mode": "staging" }  →  { "mode": "recommended" }  →  { "mode": "primary" }
```

### What Gets Compiled

Each topic article contains:
- **Summary** — 2-3 paragraph briefing (standalone understanding)
- **Timeline** — Key events with dates
- **Current State** — What's true right now
- **Key Decisions** — With rationale and source links
- **Experiments & Results** — Status table
- **Gotchas & Known Issues** — Topic-specific traps
- **Open Questions** — Unresolved threads
- **Sources** — Backlinks to every raw file (Obsidian wikilinks)

### Obsidian Compatible

The wiki output is plain markdown with Obsidian-style `[[wikilinks]]`. Open `wiki/INDEX.md` in Obsidian and you'll see the full knowledge base with bidirectional links to source files.

## Configuration

`.wiki-compiler.json` (created by `/wiki-init`):

```json
{
  "version": 1,
  "name": "My Project",
  "sources": [
    { "path": "Knowledge/", "exclude": ["wiki/"] },
    { "path": "docs/meetings/" }
  ],
  "output": "Knowledge/wiki/",
  "mode": "staging",
  "topic_hints": ["retention", "onboarding"],
  "link_style": "obsidian"
}
```

| Field | Description |
|-------|-------------|
| `name` | Display name for the knowledge base |
| `sources` | Directories to scan for .md files |
| `output` | Where compiled wiki lives |
| `mode` | `staging` / `recommended` / `primary` |
| `topic_hints` | Optional seed topics to guide classification |
| `link_style` | `obsidian` (wikilinks) or `markdown` (standard links) |

## Safety Guarantees

- Source files are **never modified** — the compiler only writes to the output directory
- The wiki can be **deleted and regenerated** at any time from source files
- Your **CLAUDE.md and AGENTS.md are never touched** — context injection happens via hooks
- **Rollback anytime** — change mode back to `staging` or delete `.wiki-compiler.json`

## Cost Savings (Real Data)

Tested on a project with 383 markdown files (13.1 MB, ~3.37M tokens):

| | Without Wiki | With Wiki |
|---|---|---|
| Session startup context | 143,269 tokens | 7,752 tokens |
| Per-question research | ~8,000 tokens (10+ files) | ~600 tokens (1 article) |
| **Reduction** | — | **95%** |

| | Tokens | Cost (Opus) |
|---|---|---|
| First compilation | ~880K | ~$13 |
| Daily incremental | ~100K | ~$1.50 |
| Daily savings (5 sessions) | ~678K | ~$10 |
| Monthly savings | ~20M | ~$305 |
| **Break-even** | — | **First session** |

Meeting transcripts compression: **503x** (122,625 lines → 244 lines in one digest article).

## Integrating with AGENTS.md

Once you're comfortable with the wiki, you can optionally add this to your project's AGENTS.md or CLAUDE.md to make sessions wiki-aware:

```markdown
## Knowledge Base

A compiled wiki is available at `{your output path}/`.

**Session startup:** Read `wiki/INDEX.md` for a topic overview, then read
specific topic articles relevant to your current task. Each article synthesizes
10-20+ raw source files into a single briefing.

**When you need depth:** Check the article's Sources section for links to
raw files. Only read raw sources when the wiki article lacks the granular
detail you need.

**Never modify wiki files directly** — they are regenerated by `/wiki-compile`.
```

This is optional and only recommended after you've used the wiki in `staging` or `recommended` mode and trust the compilation quality.

## Advanced

### Incremental Compilation

After the first full compile, `/wiki-compile` only recompiles topics whose source files changed. INDEX.md is always regenerated.

### Force Full Recompile

```
/wiki-compile --full
```

### Compile Single Topic

```
/wiki-compile --topic retention
```

### Scheduled Compilation

Use Claude Code's `/schedule` to set up daily automatic compilation.

## License

MIT
