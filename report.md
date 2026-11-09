# Boston 311 Crew Planner: analysis report

IE 5374 Hackathon. Nishanth Manoharan, Ramgopal Reddy Alumalla, Aryan Patil.

This report covers the key decisions, the assumptions behind them, the sensitivity results, and their impact on cost and on-time performance. Every number here comes from `notebooks/boston311_crew_planner.ipynb`.

## Data and forecast

We used the 2023 and 2024 311 files from Analyze Boston, 307,791 and 306,756 requests. The 2024 counts for the four departments match the brief exactly: Public Works 157,610, Transportation 81,447, Parks 18,736 and Inspectional Services 18,624. Of all 2024 requests, 15.9% have no SLA target date and 42.2% are employee-generated, close to the brief's 16% and 42%. The brief says `case_title` contains HTML codes, but neither file has any, so unescaping changed nothing. A request type can be routed to more than one department and queue: the 170 types appear in 2,208 department and queue pairs. We kept every pair in `request_types.csv`, and the intake assistant routes a type to its most common pair, which covers 95% of requests at the department level.

The 2025 forecast is the average 2023–2024 seasonal shape × the 2024 monthly level × the 2023→2024 growth. That growth is +8.1% for Transportation, +6.8% for Parks and +2.8% for Inspectional Services, but −8.0% for Public Works, so the forecast carries the drop forward: 145,040 requests in 2025 against 157,610 in 2024. We kept the method the same for all four departments rather than adjusting Public Works by hand. Without 2022 data, the backtest predicts 2024 from 2023 alone (the 2023 shape × the 2023 level, with no growth term). Its mean absolute percentage error is 7.6% for Transportation, 13.3% for Inspectional Services, 13.6% for Public Works and 17.7% for Parks. Parks misses the most because its 2023 summer was flat from May to August, while 2024 peaked sharply in June. At January 2025 crew levels, every department would have been over capacity in 7 to 10 months of 2024, which is the gap the staffing plan has to close.

## Staffing plan: cost and on-time impact

Each department gets its own MILP over the 12 months of 2025, on the Section 3 forecast. We compare it with a hold-flat baseline that keeps January's crews all year. Penalties are $20 for each newly overdue request plus $100 for each one still open a month later. The service rule caps backlog at 10% of the month's demand from February on. The plan costs $14.77M against $22.53M for the baseline, and its on-time estimate, weighted by demand, is 98.0% against 56.7%. Most of the $7.76M saving is avoided penalties: the plan spends $2.74M more on crews and cuts penalties by $10.50M. The saving comes mainly from Transportation ($6.00M) and Parks ($1.99M, which uses only seasonal crews, up to 12 in June and July). Public Works is the exception. Its plan costs $0.32M more than holding 50 crews ($7.22M against $6.90M), because it hires 10 crews for the August peak to meet the 10% rule, which lifts its on-time estimate from 86.4% to 98.1%. Overall utilization is 91.6% under the plan, against 97.2% for the overloaded baseline.

Transportation, which the brief singles out for on-time improvement, gains the most. Eight hires between January and August take it from 18 to 26 crews. Its 2025 cost falls from $8.79M to $2.79M, almost all of it the baseline's $6.81M of penalties, and its on-time estimate rises from 19.7% to 98.0%.

## Sensitivity

We changed the forecast four ways and ran each scenario two ways: re-planned, and with the base plan kept, where unmet demand shows up as backlog penalties. Under a one-month shock, re-planning buys on-time service, not savings. In the snowstorm (Public Works +35% in January and February), re-planning costs $15.35M against $15.10M with the base plan kept, because it pays for hires to meet the 10% rule. In return, it holds Public Works' on-time estimate at 95.3% instead of 87.5%. The heat wave costs about the same either way ($14.94M against $14.97M), but re-planning keeps Parks at 99.1% instead of 69.9%. With demand ×0.8, re-planning saves $2.3M ($12.37M against $14.66M) by hiring less. With demand ×1.2, it saves $4.6M ($17.67M against $22.25M) and holds the on-time estimate at 96–97%, where the kept plan falls to 21–44%.

## Equity, and resident versus employee requests

We measured on-time performance with the city's own `on_time` flag for 2024. Requests with no target date are always flagged on time, which matters most for Public Works, where 18.6% have no target: its rate is 89.7% over all requests but 87.3% over those with a target. We report the all-requests rate, which matches the README. After merging duplicate neighborhood names, Transportation has by far the widest gap between neighborhoods: 37% on time in West Roxbury against 62% in South Boston, 25.5 points. The gap is 14.1 points for Inspectional Services, whose lowest rates are in Greater Mattapan (64%) and East Boston (65%), 12.2 for Public Works and 9.0 for Parks. We left Chestnut Hill out of this comparison because it has only 12 requests.

We counted Employee Generated and City Worker App requests as employee requests, and the rest as resident requests. On-time rates for the two groups are within 4 points in every department, but employee requests close faster in Public Works (a median of 0.12 days against 0.53) and Transportation (0.88 against 1.56). Parks is the exception: its employee requests take longer (2.83 days against 1.90). We didn't adjust for the mix of request types, so some of these gaps may come from neighborhoods and requesters filing different kinds of requests rather than from differences in service.

## Intake assistant

The assistant routes a resident's message to one of the 170 request types used in 2024. It retrieves the 5 closest types, each embedded as its name and category with `bge-small-en-v1.5`, then makes one strict structured-output call to Groq's gpt-oss. That call can only choose one of those 5 or `OUT_OF_SCOPE`, and it can ask a follow-up question. The department and queue come from `request_types.csv`, never from the model. On Ram's 30 test messages (18 clear, 5 ambiguous, 4 multi, 3 out of scope), it chose the right type for 24 (80%) and the right department for 29 (97%). All six type misses were retrieval misses: the right type wasn't among the 5 retrieved. In five of them, the model still chose a type in the right department. For example, dumped tires became Construction Debris instead of Illegal Dumping, and a clogged storm drain became Roadway Flooding instead of Catchbasin. The only department miss was "something is wrong with the road on my street", sent to Transportation instead of Public Works. All three non-311 questions went to `OUT_OF_SCOPE`. The assistant suggests a routing; it doesn't replace 311 staff.

## Limitations

- The staffing model tracks backlog month by month, so it only approximates which requests become overdue, and how late. It charges the $100 late penalty for every month a request stays over a month old, not just once, so the baseline cost is a range: $20.08M to $22.53M.
- The backtest has no growth term, so it tests the seasonal shape and level, not the full forecast method.
- The forecast carries Public Works' 2023→2024 drop into 2025. If 2023 was an unusual year, it understates 2025 demand.
- On-time rates count requests with no target date as on time, which flatters Public Works.
- The intake assistant sees only each type's name and category, so retrieval misses types whose names don't match how residents describe the problem; that caused all six type misses. On vague messages it usually picks a likely type rather than asking a follow-up, and whether it asks varies between runs, even at temperature 0.
- The test set is 30 hand-labeled messages, so one message moves accuracy by 3.3 points.
- The planner agent is told to quote only numbers its tools return, and the tools return the comparisons it needs. An LLM can still misstate or compute a number, so its answers should be checked against the CSVs.
- _Aryan: add the limitations of the optimizer._

## How we used AI

_Filled in at BOS-7: the same paragraph and table as the notebook and README._
