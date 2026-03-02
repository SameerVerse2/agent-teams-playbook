[English](./README_EN.md) | [中文](./README.md)

# Agent Teams Orchestration Playbook

<div align="center">

![GitHub stars](https://img.shields.io/github/stars/KimYx0207/agent-teams-playbook?style=social)
![GitHub forks](https://img.shields.io/github/forks/KimYx0207/agent-teams-playbook?style=social)
![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Version](https://img.shields.io/badge/Claude_Code-2.1.39-green.svg)

**A Claude Code Skill for generating executable multi-agent (Agent Teams) orchestration strategies**

</div>

---

## Overview

`agent-teams-playbook` is a Claude Code-first Skill for generating executable multi-agent orchestration strategies.

> **Core Concept**: "Swarm" is the generic industry term; Claude Code's official concept is **Agent Teams**. Each teammate is an independent Claude Code instance with its own context window. Agent Teams = "parallel external brains + summarized compression", **not** "single brain expansion".

The core philosophy is "adaptive decision-making" rather than "hardcoded configuration", designed for real-world uncertainty:

- Skill/tool availability changes
- Multi-session or multi-window context forks
- Quality, speed, and cost objective conflicts

## Trigger Methods

**Natural Language Triggers:**
- agent teams, agent swarm, multi-agent, agent collaboration, agent orchestration, parallel agents
- multi-agent collaboration, swarm orchestration, agent team

**Skill Command:**
- `/agent-teams-playbook [task description]`

## Example: "High-End Video Enhancer" Team Mode

Use this prompt when you want the coordinator to behave like a full software + logic team building a premium AI video enhancement product (similar to enterprise-grade enhancement workflows):

```text
Act as a complete product engineering team for a high-end AI video enhancer.
Create an Agent Teams execution plan with these roles:
1) Product Lead (requirements, scope, acceptance criteria)
2) Video ML Researcher (denoise, deblur, super-resolution, temporal consistency)
3) Data Engineer (dataset strategy, synthetic + real data balance, annotation QA)
4) Inference Engineer (tiling, batching, mixed precision, memory/perf optimization)
5) Quality Engineer (objective metrics + human visual QA protocol)

Constraints:
- Use Stage 0 and Stage 1 mandatory flow.
- Prefer existing Skills first; fallback chain must be explicit.
- Output should include: architecture choices, risks, milestones, validation plan, and go/no-go gate.

Goal:
Deliver an MVP plan for "input low-quality video -> output enhanced video" with practical deployment steps.
```

Suggested acceptance criteria for this kind of task:

- Clear MVP boundary (what is in/out for V1)
- Measurable quality targets (e.g., temporal flicker reduction, perceptual score improvement)
- Reproducible validation protocol (datasets, checkpoints, test scripts)
- Deployment path (local GPU, cloud batch, or API inference)


## Implementation Blueprint for ComfyUI Video Enhancer Pipelines

Use this section to convert the "team-mode" plan into concrete workflow changes for production-grade enhancement pipelines.

### 1) Input and frame policy (avoid accidental short clips)

- In `VHS_LoadVideo`, avoid leaving `frame_load_cap` at a tiny test value in production.
- Recommended policy:
  - **Preview mode**: `frame_load_cap=48~120`
  - **Production mode**: `frame_load_cap=0` (full clip) or chunked processing with explicit ranges
- If source FPS is kept (`force_rate=0`), document target output FPS before rendering.

### 2) FPS consistency and motion quality

- Keep ingest and output FPS consistent unless intentionally doing motion stylization.
- If reducing FPS, pair it with `select_every_nth` so temporal sampling is explicit and reproducible.
- Always verify audio sync when output FPS differs from source FPS.

### 3) Temporal color stability (anti-flicker)

- Color transfer from a single reference image can produce frame-to-frame color variation.
- Add a temporal stabilization strategy:
  1. Prefer video-aware grading/matching nodes when available.
  2. Otherwise compute a global grade transform from representative frames and apply consistently.
  3. Start with conservative color-match strength and increase only after flicker checks.

### 4) Determinism and reproducibility

- Keep one **locked baseline profile** for QA runs:
  - fixed seed
  - fixed model versions
  - fixed key parameters (resolution, steps, sampler/perf mode)
- Keep one **exploration profile** for creative tuning:
  - randomized seed or broader parameter sweeps
- Store metadata and include profile identifiers in output filenames.

### 5) Dual-profile operation (fast iteration + final quality)

| Profile | Purpose | Typical Settings |
|--------|---------|------------------|
| Preview | Fast feedback, parameter tuning | lower resolution, shorter frame cap/chunk, lower compute |
| Final | Delivery render | target resolution, full clip/chunks, high-quality settings |

A practical team workflow is: Preview → lock parameters → Final → QA gate.

### 6) Suggested QA gate before export

Check all items before final `VHS_VideoCombine` delivery:

- No major temporal flicker in high-motion segments
- No obvious oversharpen halos or denoise smearing
- FPS/output cadence matches delivery target
- Audio alignment preserved after frame-rate decisions
- Metadata saved for reproducibility (model + seed + key params)

### 7) Agent Teams execution template for this implementation

Use this compact role split when asking `/agent-teams-playbook` to implement the pipeline:

1. **Product Lead**: define V1 scope and acceptance thresholds.
2. **Video ML Engineer**: tune denoise/deblur/upscale and temporal consistency strategy.
3. **Inference Engineer**: optimize VRAM use, chunking, and throughput.
4. **Color/Look Engineer**: design stable color transfer and anti-flicker checks.
5. **QA Engineer**: enforce objective + visual checks and go/no-go criteria.

Expected deliverables:
- parameter tables for Preview and Final profiles
- reproducible run checklist
- risk list + rollback plan


## Installation

### Option 1: CLI Installation (Recommended)

```bash
git clone https://github.com/KimYx0207/agent-teams-playbook.git
cd agent-teams-playbook
chmod +x scripts/install.sh
./scripts/install.sh
```

### Option 2: Manual Installation

```bash
mkdir -p ~/.claude/skills/agent-teams-playbook
cp SKILL.md ~/.claude/skills/agent-teams-playbook/
cp README.md ~/.claude/skills/agent-teams-playbook/
```

### Verify Installation

```bash
# Use Skill command
/agent-teams-playbook my task description

# Or use natural language
Help me build an Agent team to complete this task...
```

## Core Design Principles

1. Goals first, then organization — clarify the task before assembling a team
2. Team size depends on task complexity, parallel Agents recommended <=5
3. Skill fallback chain: local Skill scan → find-skills external search → general-purpose subagent
4. Model assignment: use Task tool's `model` parameter by complexity (opus/sonnet/haiku)
5. Never assume external tools are available — verify before execution
6. Critical milestones must have quality gates and rollback points
7. Cost is a constraint, not a fixed commitment
8. Skill Discovery is purely dynamic — scan available Skills from system-reminder, never hardcode

## Required Skill Dependencies

| Skill | Purpose | Stage |
|-------|---------|-------|
| **planning-with-files** | Manus-style file planning: task_plan.md, findings.md, progress.md | Stage 0 (mandatory) |
| **find-skills** | External skill search and discovery | Stage 1 (Skill fallback chain) |

## 5 Orchestration Scenarios

| # | Scenario | When to Use | Strategy |
|---|----------|------------|----------|
| 1 | Prompt Enhancement | Simple tasks, 1-2 steps | Optimize single agent prompt, no splitting |
| 2 | Direct Skill Reuse | Task solvable by a single Skill | Plan + search, then call matching Skill directly |
| 3 | Plan + Review | Medium/complex tasks (**default**) | Plan → user confirms → parallel execution → review |
| 4 | Lead-Member | Clear team division needed | Leader coordinates, Members execute in parallel |
| 5 | Composite Orchestration | Complex tasks, no fixed pattern | Dynamically combine above scenarios |

## 6-Stage Workflow

```
Stage 0: Planning Setup → Stage 1: Task Analysis + Skill Discovery → Stage 2: Team Assembly → Stage 3: Parallel Execution → Stage 4: Quality Gate → Stage 5: Delivery
```

> **Note**: Stage 0 (planning-with-files) and Stage 1 (Skill search, including find-skills) are **mandatory prerequisites** for all scenarios.

## Collaboration Modes

| Mode | Communication | Use Case | Launch |
|------|--------------|----------|--------|
| Subagent | One-way: child → coordinator | Parallel independent tasks | `Task` tool |
| Agent Team | Bidirectional (SendMessage) | Complex collaborative tasks | `TeamCreate` + `Task(team_name)` |

## Agent → Skill Delegation Patterns

| Pattern | Flow | Best For |
|---------|------|----------|
| Direct Call | Coordinator → `Skill` → result | Single-step Skill tasks |
| Delegated Call | Coordinator → `Task(prompt)` → subagent → `Skill` → report | Parallel Skills, long-running |
| Team Member Call | `TeamCreate` → assign → member → `Skill` → `SendMessage` | Complex coordinated tasks |

## Repository Structure

```text
agent-teams-playbook/
├── SKILL.md    # Runtime loaded (concise, ~170 lines)
└── README.md   # Developer documentation (full details)
```

## Compatibility

- **Primary platform**: Claude Code

### Context Mode (Optional)

Default: no `context: fork`. The 6-stage workflow runs in the main session. Add `context: fork` to SKILL.md frontmatter for isolated execution.

## Non-Goals

This Skill will NOT:
- Force fixed team structures
- Force single Skill dependencies
- Promise fixed speed/cost multipliers
- Claim capabilities beyond Claude Code's actual limits

---

**Version**: V4.5 | **Last Updated**: 2026-02-14 | **Maintainer**: KimYx0207
