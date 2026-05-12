# ARC-AGI-3 — AGI Final Assignment

## Overview

This project is a submission for the [ARC-Prize 2026 ARC-AGI-3 Kaggle Competition](https://www.kaggle.com/competitions/arc-prize-2026-arc-agi-3).

ARC-AGI-3 is an interactive reasoning benchmark where agents must explore unfamiliar environments, infer goals, build internal models of the environment, and plan actions — all without being given explicit task instructions.

---

## Repository Structure

```
AGI-Final/
└── arc-prize-2026-arc-agi-3/
    ├── features/                        # Feature plans and design docs
    │   └── enhanced-agent.md            # Plan for the EnhancedAgent
    ├── ARC-AGI-3-Agents/                # Main agent framework (cloned from ARC Prize)
    │   ├── main.py                      # Entry point — CLI to run agents
    │   ├── agents/
    │   │   ├── agent.py                 # Abstract Agent base class
    │   │   ├── swarm.py                 # Runs multiple agents across all games
    │   │   ├── recorder.py              # Records/replays agent sessions
    │   │   └── templates/
    │   │       ├── random_agent.py      # Baseline random action agent
    │   │       ├── llm_agents.py        # LLM base classes (LLM, ReasoningLLM, GuidedLLM)
    │   │       ├── reasoning_agent.py   # Hypothesis-tracking visual agent (o4-mini)
    │   │       ├── multimodal.py        # Pixel-diff multimodal agent (gpt-4o)
    │   │       ├── smolagents.py        # HuggingFace smolagents integration
    │   │       └── langgraph_*/         # LangGraph-based agent variants
    │   ├── tests/                       # Unit tests
    │   └── recordings/                  # Saved agent session recordings
    ├── environment_files/               # 25 local game environments (offline)
    └── arc_agi_3_wheels/               # Offline Python wheels for Kaggle submission
```

---

## Setup

### Prerequisites

- [uv](https://docs.astral.sh/uv/getting-started/installation/) package manager
- Python 3.12
- OpenAI API key
- ARC-AGI-3 API key (from [three.arcprize.org](https://three.arcprize.org/))

### Install

```bash
cd arc-prize-2026-arc-agi-3/ARC-AGI-3-Agents
cp .env.example .env
# Edit .env and add your ARC_API_KEY and OPENAI_API_KEY
uv sync
```

### Run

```bash
# Baseline random agent on ls20 game
uv run main.py --agent=random --game=ls20

# Reasoning agent (o4-mini + vision)
uv run main.py --agent=reasoningagent --game=ls20

# Multimodal agent (gpt-4o, pixel-diff)
uv run main.py --agent=multimodalllm --game=ls20

# Run all games (swarm mode)
uv run main.py --agent=reasoningagent
```

---

## Available Agents

| Agent | Model | Strategy |
|---|---|---|
| `random` | — | Random actions (baseline) |
| `llm` | gpt-4o-mini | Text-only, function calling |
| `fastllm` | gpt-4o-mini | Text-only, no observation step |
| `reasoningllm` | o4-mini | Text-only + reasoning tokens |
| `reasoningagent` | o4-mini | Vision + hypothesis tracking |
| `multimodalllm` | gpt-4o | Vision + pixel-diff + self-updating memory |
| `guidedllm` | o3 | Text-only with game-specific rules injected |
| `langgraphthinking` | — | LangGraph with extended thinking |

---

## Agent Design: EnhancedAgent (Planned)

See [features/enhanced-agent.md](arc-prize-2026-arc-agi-3/features/enhanced-agent.md) for the full design plan.

The `EnhancedAgent` combines:
- Single LLM call per turn (cheap + fast)
- Visual frame diffing (shows the model exactly what changed)
- Persistent hypothesis log across the full episode
- Full 7-action support from `available_actions`
- Token budget guard with automatic model fallback

---

## Competition

- **Platform**: Kaggle — [ARC-Prize 2026 ARC-AGI-3](https://www.kaggle.com/competitions/arc-prize-2026-arc-agi-3)
- **Submission form**: https://forms.gle/wMLZrEFGDh33DhzV9
- **Due**: Monday, May 18, 2026

---

## Report Structure

The assignment requires a 3-page report (submitted to class drive) covering:

1. **Introduction** — problem description and overall strategy
2. **Related Work** — prior ARC-AGI methods, LLM agents, search, program synthesis, RL
3. **Methods** — agent architecture, prompting strategy, algorithms, engineering decisions
4. **Results** — Kaggle score, leaderboard screenshot, local experiments, ablations
5. **Conclusions** — what worked, what didn't, lessons learned about generalization and agentic reasoning
