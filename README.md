# K6 Performance Testing Lab

This repository contains the test scripts, HTML dashboard reports, and analysis results for the K6 performance testing lab against the Grafana QuickPizza application (`https://quickpizza.grafana.com`).

## Prerequisites

- [Grafana k6](https://grafana.com/docs/k6/latest/set-up/install-k6/) installed on your machine
- Git

## Project Structure

### Part 0: K6 Basics
- `01-first-test.js` - Basic HTTP GET request to the QuickPizza home page with a status 200 check and 1-second sleep.
- `04-pizza-api.js` - Authenticated POST request to `/api/pizza` verifying vegetarian pizza recommendations.
- `05-no-token.js` - Negative test verifying that unauthenticated POST requests to `/api/pizza` return HTTP 401 (`expectedStatuses(401)`).
- `06-thresholds.js` - Load test with configurable `P95` latency threshold, error rate threshold, and check rate threshold.
- `07-requirement.js` - SLA verification script checking `p(95)<1000`, `p(99)<1500`, `http_req_failed<0.01`, and `checks>0.99` with 3 VUs over 30s.
- `explore.js` - Helper script that queries `/api/ingredients/topping` and `/api/tools` to inspect exact API naming and casing.
- `08-bug-hunt.js` - Functional load test verifying that `excludedIngredients` (`Pepperoni`) and `excludedTools` (`Knife`) are properly excluded across random pizza generations.

### Part A: User Journey and Load Test Types
- `journey.js` - Shared user journey module grouping requests for `home` (`GET /`), `quotes` (`GET /api/quotes`), and `recommendation` (`POST /api/pizza`) with realistic think times.
- `a1-smoke.js` - Smoke test (1 VU for 30s) to verify the user journey works under minimal load.
- `a2-average.js` - Average-load test (ramps to 3 VUs over 30s, holds for 2m, ramps down over 30s).
- `a3-stress.js` - Stress test (ramps to 5 VUs over 30s, holds for 2m, ramps down over 30s).
- `a4-spike.js` - Spike test (1 VU baseline, sudden spike to 5 VUs in 5s, holds for 30s, drops back to 1 VU for recovery).
- `a5-soak.js` - Soak test (sustains 2 VUs for 10m to observe long-term stability).

### Reports and Results
- `RESULTS.md` - Recorded metrics from Part 0, the comparison table for Part A, and answers to the Part B reflection questions.
- `reports/a2-average.html` - Exported k6 web dashboard report for the Average-Load Test.
- `reports/a3-stress.html` - Exported k6 web dashboard report for the Stress Test.
- `reports/a4-spike.html` - Exported k6 web dashboard report for the Spike Test.
- `reports/a5-soak.html` - Exported k6 web dashboard report for the Soak Test.

## How to Run

Note: Do not run more than 5 virtual users against `https://quickpizza.grafana.com`.

### Run Part 0 Scripts
```bash
k6 run 01-first-test.js
k6 run --vus 5 --duration 20s 01-first-test.js
k6 run --vus 5 --iterations 20 01-first-test.js
k6 run 04-pizza-api.js
k6 run 05-no-token.js
k6 run 06-thresholds.js
k6 run -e P95=300 06-thresholds.js
k6 run 07-requirement.js
k6 run explore.js
k6 run --vus 5 --duration 20s 08-bug-hunt.js
```

### Run Part A Load Tests and Export HTML Reports
```bash
k6 run a1-smoke.js
k6 run --out web-dashboard=export=reports/a2-average.html a2-average.js
k6 run --out web-dashboard=export=reports/a3-stress.html a3-stress.js
k6 run --out web-dashboard=export=reports/a4-spike.html a4-spike.js
k6 run --out web-dashboard=export=reports/a5-soak.html a5-soak.js
```
