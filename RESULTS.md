# K6 lab results

## Part 0
- Step 0.1, http_req_duration: avg=321.34ms min=321.34ms med=321.34ms max=321.34ms p(90)=321.34ms p(95)=321.34ms
- Step 0.2, requests, rate and p(95), and why the rate stays below 5: 75 requests, 3.626283/s, p(95)=348ms. The rate stays below 5 req/s because each iteration takes more than 1 second (http_req_duration + 1s sleep, averaging 1.36s per iteration), so each of the 5 virtual users completes fewer than 1 request per second.
- Step 0.6, exit code of the normal run and of the run with P95=300: Normal run exit code = 0 (p(95)=437.61ms < 1500ms); P95=300 run exit code = 99 (p(95)=484.63ms >= 300ms, threshold crossed).
- Step 0.7, rules that passed, p(95), p(99), exit code after your change: All 4 rules passed (`checks: rate>0.99`, `http_req_duration: p(95)<1000`, `http_req_duration: p(99)<1500`, `http_req_failed: rate<0.01`), p(95)=418.24ms, p(99)=439.18ms (normal exit code = 0); exit code after tightening p(95)<300 = 99.
- Step 0.8, what was wrong, and how the checks showed it: The API is case-sensitive for `excludedIngredients` and `excludedTools` (`'pepperoni'`/`'knife'` vs `'Pepperoni'`/`'Knife'`). Because the API generates random pizzas, a single manual request might randomly avoid pepperoni and a knife by chance, whereas running dozens of requests under load (72 iterations) reliably caught the bug (`no pepperoni` failed 6/72 times, `no knife needed` failed 27/72 times). After fixing the casing to `'Pepperoni'` and `'Knife'`, 100% (216/216) of checks passed.

## Part A results
| Step | Test type | Users | Req/s | p(95) | Max | Failed | Checks |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| A1 | Smoke | 1 | 0.531633/s | 375.13ms | 406.22ms | 0.00% | 100.00% |
| A2 | Average load | 3 | 1.299116/s | 405.35ms | 489.81ms | 0.00% | 100.00% |
| A3 | Stress | 5 | 2.096201/s | 412.14ms | 10.28s | 0.26% | 99.60% |
| A4 | Spike | 5 | 1.005002/s | 464.51ms | 1.62s | 0.00% | 100.00% |
| A5 | Soak | 2 | | | | | |

## Part B
1. 
2. 
3. 
4. 
5. 
6. 
7. 
8. 
9.
