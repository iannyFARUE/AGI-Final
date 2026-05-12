# Enhanced ARC-AGI-3 Agent

I want to build an `EnhancedAgent` — a multimodal, hypothesis-driven LLM agent for ARC-AGI-3 that combines the best of the existing `ReasoningAgent` (structured hypothesis tracking) and `MultiModalLLM` (pixel-diff visual analysis) while fixing their core weaknesses.

---

## FEATURES

- **Full action support** — supports all 7 game actions (RESET, ACTION1–4 movement, ACTION5 interact, ACTION6 click with x/y, ACTION7 undo), sourced dynamically from `latest_frame.available_actions` rather than hardcoded
- **Visual frame diffing** — renders the current and previous frame as PNG images, then produces a red-highlight diff image (reusing `image_diff` from `multimodal.py`) so the LLM sees exactly what changed
- **Single LLM call per turn** — combines observation + action selection into one structured tool-call response (unlike `MultiModalLLM` which makes 3 calls), keeping cost and latency low
- **Persistent hypothesis log** — maintains a running `hypothesis` string and `action_log` list across the full episode, injected into every prompt so the model builds on prior findings rather than starting cold each turn
- **Level-reset memory clearing** — clears per-level memory (screen history) on `full_reset` or level transition, but retains cross-level findings in a separate `aggregated_findings` string
- **Token budget guard** — tracks cumulative token usage and falls back to a cheaper model (`gpt-4o-mini`) if the per-episode budget exceeds a configurable threshold (default: 200k tokens)
- **Structured output via tool calling** — response is parsed from a `Pydantic` model (`EnhancedActionResponse`) that captures: `action`, `x`, `y` (for ACTION6), `reasoning`, `hypothesis_update`, `aggregated_findings`

---

## TECHNICAL REQUIREMENTS

- **Base class**: extends `LLM` from `agents/templates/llm_agents.py` (inherits `push_message`, `track_tokens`, `build_tools`, `pretty_print_3d`, `is_done`)
- **Primary model**: `gpt-4o` (vision-capable, tool-calling, cost-effective)
- **Fallback model**: `gpt-4o-mini` when token budget exceeded
- **Image rendering**: reuses `grid_to_image`, `image_to_base64`, `image_diff`, `make_image_block` from `agents/templates/multimodal.py` — no new image code
- **Structured response**: `Pydantic` `EnhancedActionResponse` model with `model_json_schema()` passed as tool parameters
- **Message windowing**: `MESSAGE_LIMIT = 6` (system + 2 user/assistant pairs + tool result) — tight window to control cost; hypothesis log provides long-term memory instead
- **MAX_ACTIONS**: `200` (generous enough for complex puzzles, bounded to avoid runaway cost)
- **Registration**: added to `AVAILABLE_AGENTS` in `agents/__init__.py` as `"enhancedagent"`

---

## CONSTRAINTS

- Must not break existing agents — new file only, minimal changes to `__init__.py`
- Must run offline using the local `environment_files/` (no `ONLINE_ONLY` requirement)
- `OPENAI_API_KEY` must be present in `.env`; no other new env vars required
- ACTION6 (click) coordinates must be clamped to `[0, 63]` grid space (the grid is 64×64; `multimodal.py` uses `_SCALE=2` for display but action coords are always grid-space)
- The pixel-diff image must only be sent when a previous frame exists — skip on the first turn
- Pydantic validation errors on the LLM response should fall back to `GameAction.RESET` with a logged warning, not a crash

---

## CONTEXT

### Existing code to reuse

| Symbol | File | Purpose |
|---|---|---|
| `LLM` | `agents/templates/llm_agents.py:16` | Base class — message loop, token tracking, tool building |
| `grid_to_image` | `agents/templates/multimodal.py:55` | Converts 64×64 int grid → PIL Image |
| `image_to_base64` | `agents/templates/multimodal.py:73` | PIL Image → base64 PNG string |
| `image_diff` | `agents/templates/multimodal.py:92` | Red-highlight diff between two frames |
| `make_image_block` | `agents/templates/multimodal.py:82` | Wraps base64 string as OpenAI image_url block |
| `ReasoningActionResponse` | `agents/templates/reasoning_agent.py:18` | Reference for structured response shape |
| `capture_reasoning_from_response` | `agents/templates/llm_agents.py:453` | Extracts reasoning token counts from response |

### Key framework facts
- `FrameData.frame` is `list[list[list[int]]]` — a list of grids (usually 1, sometimes more for animation)
- `FrameData.available_actions` is `list[GameAction]` — use this to filter which actions to expose as tools
- `FrameData.full_reset` is `bool` — true when the environment fully resets (new episode)
- `FrameData.levels_completed` is `int` — increment signals a level was just completed
- `GameAction.is_complex()` returns `True` for ACTION6 — requires `x`, `y` data via `action.set_data({"x": ..., "y": ...})`
- `GameAction.from_name(str)` converts tool call name → `GameAction` enum value

### File to create
`ARC-AGI-3-Agents/agents/templates/enhanced_agent.py`

### Registration change
`ARC-AGI-3-Agents/agents/__init__.py` — add import and entry:
```python
from .templates.enhanced_agent import EnhancedAgent
AVAILABLE_AGENTS["enhancedagent"] = EnhancedAgent
```

---

## IMPLEMENTATION ORDER

1. **`EnhancedActionResponse` Pydantic model** — define fields: `action` (Literal of all action names), `x` (Optional[int]), `y` (Optional[int]), `reasoning` (str), `hypothesis_update` (str), `aggregated_findings` (str)
2. **`EnhancedAgent` class skeleton** — subclass `LLM`, set class vars (`MODEL`, `MAX_ACTIONS`, `MESSAGE_LIMIT`, `TOKEN_BUDGET`)
3. **`build_tools` override** — dynamically build tools from `available_actions`, include x/y params only for ACTION6
4. **`build_system_prompt`** — static system prompt explaining the agent's role and grid format
5. **`build_user_message`** — assembles image blocks (previous frame, current frame, diff) + text with current hypothesis/action log
6. **`choose_action` override** — full loop: handle `full_reset`, build messages, call LLM, parse `EnhancedActionResponse`, update hypothesis log, return `GameAction`
7. **Registration** in `__init__.py`
8. **Smoke test**: `uv run main.py --agent=enhancedagent --game=ls20`
