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
| A5 | Soak | 2 | 0.972083/s | 410.98ms | 728.61ms | 0.00% | 100.00% |

## Part B
1. See the completed `## Part A results` table above for all 5 test types (A1 Smoke, A2 Average load, A3 Stress, A4 Spike, and A5 Soak) with their respective users, request rates, p(95) and max latencies, failure rates, and check pass percentages.
2. In Step 0.2, 5 virtual users generated 3.63 req/s (75 requests in 20.7s) rather than 5 req/s because each iteration includes both the network response time (`http_req_duration`, averaging ~312.52ms) and a 1-second think time (`sleep(1)`). Since each iteration takes ~1.36s (> 1s), each VU can only execute ~0.73 requests per second, keeping the total throughput for 5 VUs below 5 req/s.
3. Between A2 (3 VUs, `p(95)=405.35ms`) and A3 (5 VUs, `p(95)=412.14ms`), the 95th percentile response time increased by only ~6.8ms while throughput scaled nearly linearly from 1.30 req/s to 2.10 req/s. In terms of the **Operational Envelope**, 5 concurrent virtual users on Grafana's shared QuickPizza server is still well within the system's normal capacity boundary (operational envelope), so the server handles the additional concurrency without resource contention or queueing delays, and the minor difference is mostly normal network variance (aside from a single transient network timeout of 10.28s out of 381 requests).
4. During the A4 Spike test (sudden jump from 1 VU to 5 VUs in 5s), the request rate spiked while `p(95)` rose moderately to `464.51ms` (with a transient max of `1.62s` on initial connection establishment during the spike) and `0.00%` errors (`100.00%` checks passed). As shown in the data and `reports/a4-spike.html`, the server absorbed the 5-VU spike without failing requests and recovered immediately (< 1 iteration / ~5s) back to baseline ~300–370ms latency as soon as the load ramped back down to 1 VU at the 1m10s mark.
5. Over the 11-minute A5 Soak test (2 VUs sustained for 10m, 642 requests), response times remained flat and stable (`avg=327.08ms`, `p(95)=410.98ms`, `max=728.61ms`, `0.00%` failures), showing no performance degradation over time. In real-world production systems, multi-hour soak tests are designed to uncover gradual resource-exhaustion defects such as **memory leaks** (leading to Out-Of-Memory crashes or heavy GC pauses), **database/HTTP connection pool exhaustion** (unclosed connections/handles), and **disk space depletion** from unrotated logs or temporary files.
6. Averages (`avg`) obscure outliers because a large number of fast requests can mask a small percentage of severely delayed requests. Percentiles like `p(95)` show the latency experienced by the slowest 5% of requests (tail latency), ensuring that SLA thresholds protect the worst-case experience of real users rather than just the arithmetic mean.
7. Two reasons why testing a shared public site (`quickpizza.grafana.com`) over the internet differs from a local lab environment:
   - **Unpredictable internet routing and latency jitter:** Packets traverse multiple public ISP hops and geographically distant links, introducing variable round-trip times and occasional packet loss (such as the single 10.28s outlier in A3) unrelated to server CPU/memory load.
   - **Noisy-neighbor interference:** Because QuickPizza is a shared public demo server used simultaneously by other learners and automated tests worldwide, background traffic from other users fluctuates unpredictably and affects measured response times.
8. Running high-load stress, spike, or breakpoint tests against websites you do not own or have explicit permission to test is unethical, potentially illegal, and operationally harmful because:
   - **Operational impact (Denial of Service):** It can overwhelm server resources and degrade or take down the service for legitimate users.
   - **Financial harm:** It can trigger auto-scaling, bandwidth charges, or third-party API fees that cost the site owner money.
   - **Legal & security consequences:** Unauthorized flood traffic is indistinguishable from a DoS/DDoS attack, violating Terms of Service and computer misuse laws (e.g., CFAA or Computer Crime Acts) and getting your IP/network blocked.
9. **Vague requirement:** "The pizza recommendation feature should be fast and reliable when users are browsing the site."
   **Testable SLA requirement:** "Under a concurrent load of 3 virtual users over 30 seconds, the `POST /api/pizza` endpoint must respond within 1000ms for 95% of requests (`p(95) < 1000ms`), within 1500ms for 99% of requests (`p(99) < 1500ms`), maintain an HTTP error rate below 1% (`rate < 0.01`), and pass over 99% of functional checks (`rate > 0.99`)."
   **K6 `thresholds` code block:**
   ```javascript
   export const options = {
     vus: 3,
     duration: '30s',
     thresholds: {
       http_req_duration: ['p(95)<1000', 'p(99)<1500'],
       http_req_failed: ['rate<0.01'],
       checks: ['rate>0.99'],
     },
   };
   ```
