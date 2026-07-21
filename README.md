# Thestrays — Novel-Writing Agent Pipeline

Four Claude Code subagents that write a novel one chapter at a time, each stage
feeding the next. Every agent reads your world rules from `world/` first and
treats them as hard constraints.

```
  world/                         you upload canon here
     │
     ▼
  ┌─────────┐   plot spec    ┌────────────┐   scene cards   ┌───────────────┐   brief    ┌────────┐   prose
  │  plot   │ ─────────────▶ │ scene-card │ ──────────────▶ │ writing-brief │ ─────────▶ │ prose  │ ───────▶ chapter
  └─────────┘   (JSON)       └────────────┘   (JSON)        └───────────────┘   (MD)     └────────┘   (MD)
```

## The four agents

| # | Agent | Input | Output | Key fields |
|---|-------|-------|--------|-----------|
| 1 | **plot** | premise + `world/` | `manuscript/plot/episode-NN.json` | episode role, key scenes (each with a turn), opening & closing hook, setups/payoffs |
| 2 | **scene-card** | plot spec | `manuscript/scene-cards/episode-NN.json` | POV, start state, end state, value shift, must-include, **forbidden items** |
| 3 | **writing-brief** | scene cards | `manuscript/briefs/episode-NN-ch-NN.md` | POV/tense lock, density limits, required-elements checklist, consolidated forbidden list |
| 4 | **prose** | writing brief | `manuscript/chapters/episode-NN-ch-NN.md` | the finished chapter, **5,000–8,000 characters**, with a compliance report |

The `forbidden_items` rail is the spine of the pipeline: it originates in the
scene cards, is consolidated in the brief, and is enforced absolutely by the
prose agent — so future reveals, off-POV knowledge, and world-rule breaks can't
leak into the manuscript.

## How to run it

1. Upload your world/canon files into `world/` (see `world/README.md`).
2. Run the stages in order. Each agent writes a file the next one reads, so you
   can inspect and tweak between stages:

   ```
   > Use the plot agent to outline episode 1 from this premise: <premise>
   > Use the scene-card agent on episode 1
   > Use the writing-brief agent for episode 1, chapter 1
   > Use the prose agent to write episode 1, chapter 1
   ```

   (In Claude Code, subagents in `.claude/agents/` are invoked by name; the main
   assistant will delegate to them, or you can ask for one explicitly.)

3. Repeat per episode/chapter. The agents read prior outputs in `manuscript/`
   for continuity, so later chapters stay consistent with earlier ones.

## Layout

```
.claude/agents/    plot.md · scene-card.md · writing-brief.md · prose.md
world/             your uploaded canon (agents read all of it)
manuscript/
  plot/            episode plot specs (JSON)
  scene-cards/     per-episode scene cards (JSON)
  briefs/          per-chapter writing briefs (Markdown)
  chapters/        finished prose (Markdown)
```

The agent definitions are the source of truth for each stage's exact
input/output contract — open them in `.claude/agents/` to see the full schemas.
