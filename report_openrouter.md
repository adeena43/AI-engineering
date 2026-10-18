## Results (openrouter mode)
Puzzles: [100] (seeds), each exactly 3 moves from solved, optimum = 3, move limit = 20.

| Model | Runs | Success % | Avg latency/move (s) | Median latency (s) | Avg cached (s) | Avg total time (s) | Avg moves | Optimal gap | API calls | Cache hits | Invalid | Cost ($) | Avg confidence |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Jev 1.13 | 1 | 0 | 2.088 | 1.490 | 0.0000 | 35.51 | 20.0 | - | 17 | 3 | 0 | 0.00057 | 0.31 |
| GPT-6 Luna Decisions | 1 | 0 | 5.628 | 5.628 | 0.0000 | 11.26 | 2.0 | - | 2 | 2 | 2 | 0.00000 | 0.17 |
| Cloudflare Clef | 1 | 0 | 3.631 | 3.631 | 0.0000 | 7.26 | 19.0 | - | 2 | 17 | 0 | 0.00019 | 0.07 |
| Claude Opus 5.5 | 1 | 100 | 11.740 | 10.322 | - | 35.22 | 3.0 | 0.0 | 3 | 0 | 0 | 0.03483 | - |

### Data for the five questions
1. Fastest decisions (lowest avg live latency): Jev 1.13 (2.088)
2. Highest solve rate: Claude Opus 5.5 (100.000) %
3. Shortest solutions (lowest optimal gap, 0 = optimal): Claude Opus 5.5 (0.000)
4. Fastest model: Jev 1.13 (2.088) | Best quality: Claude Opus 5.5 (0.000). Compare the two lines to answer.
5. Fast model vs frontier LLM vs hybrid: argue from the table above (latency, success, gap, cost).