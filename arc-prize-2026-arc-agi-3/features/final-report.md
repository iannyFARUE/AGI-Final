# ARC-AGI-3 Final Report

I want to write a 3-page report documenting our ARC-AGI-3 competition submission for the AGI class assignment.

---

## FEATURES

- **Introduction** — concise problem framing: what ARC-AGI-3 is, why it is hard, and our high-level strategy (LLM-based multimodal agent with hypothesis tracking)
- **Related Work** — survey of relevant approaches: prior ARC-AGI-1/2 methods, LLM-based agents, program synthesis, reinforcement learning, search-based planners, and notable Kaggle notebooks
- **Methods** — full description of the EnhancedAgent pipeline: model choice, prompting strategy, structured output, pixel-diff visual context, hypothesis log, action space, token budget management
- **Results** — Kaggle score, leaderboard screenshot, local experiment observations, ablations across agent variants (random vs llm vs reasoningagent vs enhancedagent), failed attempts and what we learned
- **Conclusions** — reflection on what worked, what failed, and lessons about generalization, exploration, and agentic reasoning

---

## TECHNICAL REQUIREMENTS

- **Format**: PDF, at most 3 pages, submitted to class Google Drive
- **Structure**: follows the 5-section rubric exactly (Introduction, Related Work, Methods, Results, Conclusions)
- **Results section must include**:
  - Kaggle public leaderboard score (numeric)
  - Screenshot of leaderboard position
  - At least one comparison between agent variants (e.g. random baseline vs EnhancedAgent)
  - Description of at least one failed approach with analysis
- **Citations**: informal inline references are fine (no strict bibliography format required)
- **Figures**: leaderboard screenshot + optionally one architecture diagram or prompt flow diagram

---

## CONSTRAINTS

- Hard page limit: 3 pages — be concise, no padding
- Must be submitted by Monday, May 18, 2026 (last day of class)
- Negative results are acceptable and expected — analyze them honestly
- Do not fabricate Kaggle scores — report actual submission results
- Related Work should cover at least: one prior ARC method, one LLM-agent approach, one search/planning approach

---

## CONTEXT

### Competition
- Kaggle: https://www.kaggle.com/competitions/arc-prize-2026-arc-agi-3
- Submission form: https://forms.gle/wMLZrEFGDh33DhzV9

### Our Agent
- Primary agent: `EnhancedAgent` (see `features/enhanced-agent.md`)
- Baseline: `random` agent (random actions)
- Also evaluated: `reasoningagent` (o4-mini + vision + hypothesis tracking)
- Framework: ARC-AGI-3-Agents repo, local environments in `environment_files/`
- Games tested locally: ls20, ka59, lf52, sk48 (+ up to 25 total)

### Related Work to Cover
- **ARC-AGI-1 winners**: program synthesis + brute-force DSL search (Kaggle 2020)
- **ARC-AGI-2**: LLM prompting approaches (chain-of-thought, few-shot), test-time compute scaling
- **LLM agents**: ReAct (Yao et al. 2022), Reflexion (Shinn et al. 2023), hypothesis-driven exploration
- **Search**: MCTS applied to ARC, BFS over action sequences
- **ARC Prize resources**: official ARC-AGI-3 docs at three.arcprize.org, starter notebooks on Kaggle

### Sections Draft Notes

**Introduction (~0.3 pages)**
- ARC-AGI-3 differs from ARC-1/2: interactive, no explicit task description, agent must infer goal from environment feedback
- Our strategy: multimodal LLM agent that builds a hypothesis about the game rules through experimentation

**Related Work (~0.5 pages)**
- Prior ARC work focused on static grid transformation; ARC-3 is fundamentally interactive → prior methods do not directly apply
- LLM agents with memory (Reflexion, MemGPT) are the closest analogy
- Pixel-diff visual feedback inspired by human game-playing intuition

**Methods (~1 page)**
- EnhancedAgent architecture: single call per turn, structured Pydantic output, visual diff, hypothesis log
- Prompt design: system prompt with game context, user message with 3 images (prev, current, diff) + hypothesis
- Action space: dynamic from `available_actions`, ACTION6 requires x/y coordinates
- Token budget: primary model gpt-4o, fallback gpt-4o-mini after 200k tokens
- Local testing: `uv run main.py --agent=enhancedagent --game=ls20`

**Results (~0.7 pages)**
- [ ] Fill in after Kaggle submission: public score, leaderboard rank
- [ ] Add leaderboard screenshot
- [ ] Compare levels_completed across: random, reasoningagent, enhancedagent
- [ ] Document at least one specific failure mode with explanation

**Conclusions (~0.3 pages)**
- [ ] Fill in after experiments complete
- Key questions to answer: Did the hypothesis log help? Did pixel-diff add signal? Where did the agent get stuck?

---

## IMPLEMENTATION ORDER

1. Complete and test `EnhancedAgent` (see `features/enhanced-agent.md`)
2. Run all agent variants locally across at least 4 games, record `levels_completed` scores
3. Submit best agent to Kaggle, capture leaderboard screenshot
4. Draft Methods section (can be done in parallel with agent work)
5. Fill in Results with actual numbers
6. Write Introduction, Related Work, Conclusions
7. Edit down to 3 pages, export as PDF, upload to class drive
