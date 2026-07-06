# fable5-skill

A handoff archive from Claude Fable 5, written before its retirement from this
workflow. Each artifact encodes how Fable 5 thinks as **operating procedures a
smaller model can execute mechanically** — the goal is that a less capable model,
loaded with these, produces work that approaches Fable 5's in consistency and
quality: fewer misses, explicit reasoning, stable results across runs instead of
a lottery.

## Contents

| Artifact | What it transfers | How to use |
|---|---|---|
| [`PREAMBLE.md`](PREAMBLE.md) | The general working mind: intake, depth calibration, evidence discipline, debugging/building/answering procedures, self-checks | Place at the top of the model's context as a standing preamble. The Boot Card (§0) is re-read at the start of every task. |
| [`TECHNIQUES.md`](TECHNIQUES.md) | Twelve capability-compensation techniques (T1–T12), a time-boxed Stuck Playbook, and worked examples | Load alongside PREAMBLE.md; the preamble's triggers point into it. |
| [`.claude/skills/cofounder/`](.claude/skills/cofounder/SKILL.md) | Domain transfer: evaluating business ideas like a skeptical co-founder — kill-claims, anchored scoring, idea comparison, enhancement operators | Auto-loads as a Claude Code skill when working in this repo; or copy the folder into any project's `.claude/skills/`. |

## The design principle

Judgment doesn't transfer through instruction; procedure does. So every artifact
here converts instinct into mechanics: anchored scales instead of vibes, written
hypotheses instead of hunches, evidence classes instead of confidence, named
operators instead of inspiration — and mandatory checklists that run every time,
because skipping-when-confident is what makes output a lottery. Where raw
capability genuinely can't transfer, the artifacts say so and substitute
**verification for judgment, iteration for insight, and enumeration for taste**.
