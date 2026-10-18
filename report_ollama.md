## Results (ollama mode)
Puzzles: [3, 4, 5, 6, 7] (seeds), each exactly 3 moves from solved, optimum = 3, move limit = 20.

| Model | Runs | Success % | Avg latency/move (s) | Median latency (s) | Avg cached (s) | Avg total time (s) | Avg moves | Optimal gap | API calls | Cache hits | Invalid | Cost ($) | Avg confidence |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Jev 1.13 | 5 | 0 | 0.644 | 0.629 | 0.0000 | 12.36 | 20.0 | - | 96 | 4 | 0 | 0.00000 | - |
| GPT-6 Luna Decisions | 5 | 0 | 0.625 | 0.624 | 0.0000 | 11.13 | 20.0 | - | 89 | 11 | 0 | 0.00000 | - |
| Cloudflare Clef | 5 | 0 | 0.630 | 0.629 | 0.0000 | 8.69 | 20.0 | - | 69 | 31 | 0 | 0.00000 | - |
| Claude Opus 5.5 | 5 | 0 | 0.648 | 0.637 | 0.0000 | 10.62 | 20.0 | - | 82 | 18 | 0 | 0.00000 | - |

### Data for the five questions
1. Fastest decisions (lowest avg live latency): GPT-6 Luna Decisions (0.625)
2. Highest solve rate: Jev 1.13, GPT-6 Luna Decisions, Cloudflare Clef, Claude Opus 5.5 (0.000) %
3. Shortest solutions (lowest optimal gap, 0 = optimal): n/a
4. Fastest model: GPT-6 Luna Decisions (0.625) | Best quality: n/a. Compare the two lines to answer.
5. Fast model vs frontier LLM vs hybrid: argue from the table above (latency, success, gap, cost).