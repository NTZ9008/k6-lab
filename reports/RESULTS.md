# K6 lab results

## Part 0
- Step 0.1, http_req_duration: avg=321.34ms min=321.34ms med=321.34ms max=321.34ms p(90)=321.34ms p(95)=321.34ms
- Step 0.2, requests, rate and p(95), and why the rate stays below 5: 75 requests, 3.626283/s, p(95)=348ms. The rate stays below 5 req/s because each iteration takes more than 1 second (http_req_duration + 1s sleep, averaging 1.36s per iteration), so each of the 5 virtual users completes fewer than 1 request per second.
- Step 0.6, exit code of the normal run and of the run with P95=300: 
- Step 0.7, rules that passed, p(95), p(99), exit code after your change: 
- Step 0.8, what was wrong, and how the checks showed it: 

## Part A results
| Step | Test type | Users | Req/s | p(95) | Max | Failed | Checks |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| A1 | Smoke | 1 | | | | | |
| A2 | Average load | 3 | | | | | |
| A3 | Stress | 5 | | | | | |
| A4 | Spike | 5 | | | | | |
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
