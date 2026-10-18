# Rubik's Cube Race: fast decision models vs a frontier LLM

**Student:** Adina  |  **Roll No:** 23k-0008  |  **Course:** AI Engineering, Assignment 02

## Demo recording

**[Watch the screen recording: demo1.mp4 and demo2.mp4](demo1.mp4)**
Direct link (after pushing): `https://github.com/adeena43/AI-engineering/blob/main/demo1.mp4`

The recording shows the real OpenRouter run (seed 100): models playing (in progress), the finished state (Opus SOLVED, the other three NOT SOLVED), with my name and roll number visible on screen throughout. The statistics are also in the tables below.

> In the recording, Jev shows `0:00.00` because in that session its moves were replayed from the persistent cache (the live calls were made in an earlier session of the same run; see "How the real run was executed").

---

## What was built

An AI-powered Rubik's Cube game and benchmark. Every model gets the **same** starting cube, the **same** state/action contract and the same move definitions, and picks one move per turn until the cube is solved or the move limit is reached.

- **Cube engine** (`cube.py`): 54-facelet state, 18 moves (U, R, F, D, L, B with `'` and `2`), solved check. Facelet permutations are generated from 3D geometry. `python test_cube.py` verifies it (e.g. `(R U R' U')` x 6 returns to solved).
- **Exact optimum by BFS** (`cube.py`): breadth-first search from the solved cube gives the exact shortest distance. Every start state is exactly **3 moves** from solved (BFS-verified), so the optimal solution length is 3.
- **Model adapters** (`llm.py`): Decisions API (`POST /api/alpha/decisions`) for Jev, GPT-6 Luna Decisions and Cloudflare Clef, with one typed `choice` question over the 18 moves (same wording for all three); chat completions with constrained JSON output for Claude Opus 5.5. Responses are validated; malformed or invalid output is counted, never silently converted into a move.
- **Backend and game UI** (`main.py`, `static/index.html`): FastAPI backend, HTML/JS frontend with the unfolded cube, current move, move history, result badge and the statistics table. Name and roll number are pinned on screen.
- **Budget and cache safeguards** (rules from the instructor's email): 20 moves max per model; Claude max 5 real requests; 65-request total cap enforced in the backend; every real request counted *before* it is sent (failures included) and persisted in `cache.json` (never reset); every request checks the persistent cache first; live latency and cached latency are tracked separately; every real request is logged to `requests.log`; no retries (same policy for all models); loops are bounded; a model stops after 2 errors in a row.
- **Modes**: `mock` (no API, for debugging), `ollama` (local, free), `openrouter` (real, limited).

## Models

| Model | Category | OpenRouter ID | Interface |
|---|---|---|---|
| Jev 1.13 | System-1 decision model | `typesafe/jev-1.13` | Decisions API |
| GPT-6 Luna Decisions | System-1 decision model | `openai/gpt-6-luna-decisions` | Decisions API |
| Cloudflare Clef | System-1 decision model | `cloudflare/clef` | Decisions API |
| Claude Opus 5.5 | Frontier LLM | `anthropic/claude-opus-5.5` | Chat completions |

## Results: OpenRouter (real run, 1 puzzle, seed 100)

Per the instructor's budget rules only one real OpenRouter experiment was run. Start state: exactly 3 moves from solved; optimum = 3; move limit = 20 (Claude: 5 requests).

| Model | Solved | Success % | Avg latency / move (s) | Median (s) | Total time (s) | Total moves | Optimal gap | Live calls | Cache hits | Invalid | Cost ($) | Avg confidence |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Jev 1.13 | 0/1 | 0 | 2.088 | 1.490 | 35.51 | 20 (limit) | n/a | 17 | 3 | 0 | 0.00057 | 0.31 |
| GPT-6 Luna Decisions | 0/1 | 0 | 5.628 (*) | 5.628 | 11.26 | 2 | n/a | 2 | 2 | 2 | 0.00000 | 0.17 |
| Cloudflare Clef | 0/1 | 0 | 3.631 (*) | 3.631 | 7.26 (*) | 19 | n/a | 2 | 17 | 0 | 0.00019 | 0.07 |
| Claude Opus 5.5 | 1/1 | 100 | 11.740 | 10.322 | 35.22 | **3** | **0** | 3 | 0 | 0 | 0.03483 | n/a |

Optimal gap = model moves minus exact shortest moves (3). Defined only for solved runs. Source: `results.json`, produced by `python report.py openrouter` (`report_openrouter.md`).

(*) **Read these numbers with care** (details under "Honest notes"):
- Luna's 5.6 s comes from its two *refused* calls only.
- Clef's latency and total time come from only 2 live calls, because 17 of its 19 moves were replayed from the cache of an earlier session of the same run.

### Real OpenRouter requests used (`cache.json` / `requests.log`)

Total **49 of 65**: Jev 17, GPT-6 Luna 8, Clef 20, Opus 4 (limit 5). Recorded cost of the finished runs is about **$0.036**; the actual figure is slightly higher because of one failed Opus call.

## Results: Ollama (local baseline, 5 puzzles, seeds 3 to 7)

**Important:** in Ollama mode all four slots ran the **same** small model, `gemma3:1b`. This was a pipeline and baseline test (free, no API limits), not a comparison of the four models. Models were run one at a time so they did not slow each other down.

| Slot (all `gemma3:1b`) | Runs | Success % | Avg latency / move (s) | Median (s) | Avg total time (s) | Avg moves | Local requests | Cache hits | Invalid |
|---|---|---|---|---|---|---|---|---|---|
| Jev slot | 5 | 0 | 0.644 | 0.629 | 12.36 | 20.0 | 96 | 4 | 0 |
| Luna slot | 5 | 0 | 0.625 | 0.624 | 11.13 | 20.0 | 89 | 11 | 0 |
| Clef slot | 5 | 0 | 0.630 | 0.629 | 8.69 | 20.0 | 69 | 31 | 0 |
| Opus slot | 5 | 0 | 0.648 | 0.637 | 10.62 | 20.0 | 82 | 18 | 0 |

(`report_ollama.md`.) Results show a 1B-parameter model returns valid JSON every time (0 invalid) but cannot solve a 3-move cube. The small latency differences are noise: same model.

Mock-mode runs in `results.json` were only used to debug the app and are not part of any result.

## Answers to the five questions (based on the single real puzzle, so tentative)

**1. Which model is fastest in making decisions?**
Jev: 2.09 s average live latency per move (median 1.49 s) over 17 real calls, the only decision model with a solid number of live calls. Clef (3.6 s from 2 calls; about 2.3 s on the dashboard during the first session, not persisted) and Luna (5.6 s from two refused calls) cannot be ranked reliably against it. Claude Opus is clearly the slowest: 11.7 s per move.

**2. Which model achieved the highest solve rate?**
Claude Opus 5.5: 1 of 1 (100%). The three decision models solved 0 of 1. With one puzzle this is evidence, not a statistic.

**3. Which model found the shortest solutions, and how close to the optimum?**
Only Opus solved the cube: in **3 moves, the exact optimum (gap 0)**. For the others the gap is undefined because they never solved it. Jev and Clef ended up oscillating between turns such as `U U'`, which shows they pick a plausible move each turn but do not plan ahead.

**4. Did the fastest model also achieve the best solution quality?**
No. Jev was the fastest per move but did not solve the cube. Opus was about 5x slower per move but was the only one that solved it, and its total wall-clock time (35.2 s) was about the same as the time Jev spent (35.5 s) making 20 moves without solving. Cost differs sharply: Opus about $0.035 vs about $0.0008 for all three decision models together.

**5. Fastest decision model, frontier LLM, or a hybrid in a real application?**
A **hybrid**, with the evidence stated honestly. For a multi-step planning task like this, only the frontier LLM worked, so the decision model should not be trusted alone. But decision models are about 5x faster per move and much cheaper, and they expose a confidence score. Their average confidence here was low (0.31 Jev, 0.17 Luna, 0.07 Clef) on a task they failed, which suggests a routing design: use the fast model first and escalate to the frontier LLM when confidence is low or progress stalls. Decision models are better suited to fast, high-volume single-step choices (classification, routing). One puzzle is too small to prove the hybrid, so this is a design recommendation to test, not a conclusion.

## Honest notes and limitations

- **One real puzzle only.** The instructor's API budget (65 requests, Claude 5) allowed a single real experiment. Results show behaviour on one cube, not a statistically valid ranking. The Ollama runs (5 puzzles) used a single small model.
- **GPT-6 Luna refused.** After two valid moves, Luna returned HTTP 502 `OpenAI refused to answer question "next_move"` in three separate sessions at the same state (see `requests.log`). Refusal is a documented outcome of OpenAI's Decisions API, surfaced by OpenRouter as a 502. Retrying did not help, and changing the wording would have broken the "same question for all models" rule, so Luna is recorded as not solved. Its 8 calls were 2 answered and 6 refused.
- **Clef hit the 20-call limit.** One of its calls was lost when I stopped an earlier session with Ctrl+C before the answer was saved, so it made 19 moves. The cube was cycling between `U'` and `D'` turns, so one more move would not have solved it.
- **Opus lost one request to a bug of mine.** The first Opus call used `max_tokens=300`; the model spent all of them reasoning and returned `finish_reason: length` with empty content. I raised the limit to 4000, stopped bad responses from being cached, and Opus then solved the cube in 3 calls (4 of its 5 allowed requests used).
- **How the real run was executed.** The run was done in several sessions of the same puzzle (seed 100) using the persistent cache; replayed moves are counted as cache hits, their latency is tracked separately, and they are not counted as new API calls. This is why some totals (Clef, Jev) understate wall-clock time and why in the recording some models finish instantly.
- Model names in the assignment screenshots (for example "GPT-6 Sol", "Laya") are mock-ups; the IDs in the table above are the ones used.

## Setup

```bash
python -m venv venv
venv\Scripts\activate            # Windows PowerShell: .\venv\Scripts\Activate.ps1
pip install -r requirements.txt
copy .env.example .env           # then fill in your own values
uvicorn main:app
```

Open http://127.0.0.1:8000, choose a mode (Mock, Ollama, OpenRouter) and a seed, and press **Run benchmark**. The OpenRouter mode asks for confirmation because it makes real, limited requests.

`.env` (never committed):

```
OPENROUTER_API_KEY=your_key_here
STUDENT_NAME=Your Name
STUDENT_ROLL=Your Roll Number
OLLAMA_URL=http://localhost:11434
OLLAMA_JEV=gemma3:1b
OLLAMA_LUNA=gemma3:1b
OLLAMA_CLEF=gemma3:1b
OLLAMA_OPUS=gemma3:1b
```

Useful commands:

```bash
python test_cube.py              # engine tests + BFS counts (1, 18, 243, 3240)
python dry_check.py              # prints the request bodies without sending anything
python report.py openrouter      # statistics table (also: ollama, mock)
```

## Repository layout

| Path | Purpose |
|---|---|
| `cube.py` | Cube engine, 18 moves, BFS optimum, scramble generator |
| `llm.py` | Model adapters, persistent cache, request budget, logging |
| `main.py` | FastAPI backend and game runner |
| `static/index.html` | Game interface and statistics dashboard |
| `report.py` | Builds the statistics tables from `results.json` |
| `test_cube.py`, `dry_check.py` | Engine tests and request dry-run |
| `cache.json`, `results.json`, `requests.log` | Persistent cache and request counters, per-run results, log of every real request |
| `report_openrouter.md`, `report_ollama.md` | Generated statistics |
| `demo1.mp4` | Screen recording |
| `.env.example`, `requirements.txt` | Setup |
