# Boston 311 Crew Planner: analysis report

IE 5374 Hackathon. Nishanth Manoharan, Ramgopal Reddy Alumalla, Aryan Patil.

This report covers the key decisions, the assumptions behind them, the sensitivity results, and their impact on cost and on-time performance. Every number here comes from `notebooks/boston311_crew_planner.ipynb`.

## Data and forecast

We used the 2023 and 2024 311 files from Analyze Boston, 307,791 and 306,756 requests. The 2024 counts for the four departments match the brief exactly: Public Works 157,610, Transportation 81,447, Parks 18,736 and Inspectional Services 18,624. Of all 2024 requests, 15.9% have no SLA target date and 42.2% are employee-generated, close to the brief's 16% and 42%. The brief says `case_title` contains HTML codes, but neither file has any, so unescaping changed nothing. A request type can be routed to more than one department and queue: the 170 types appear in 2,208 department and queue pairs. We kept every pair in `request_types.csv`, and the intake assistant routes a type to its most common pair, which covers 95% of requests at the department level.

The 2025 forecast is the average 2023–2024 seasonal shape × the 2024 monthly level × the 2023→2024 growth. That growth is +8.1% for Transportation, +6.8% for Parks and +2.8% for Inspectional Services, but −8.0% for Public Works, so the forecast carries the drop forward: 145,040 requests in 2025 against 157,610 in 2024. We kept the method the same for all four departments rather than adjusting Public Works by hand. Without 2022 data, the backtest predicts 2024 from 2023 alone (the 2023 shape × the 2023 level, with no growth term). Its mean absolute percentage error is 7.6% for Transportation, 13.3% for Inspectional Services, 13.6% for Public Works and 17.7% for Parks. Parks misses the most because its 2023 summer was flat from May to August, while 2024 peaked sharply in June. At January 2025 crew levels, every department would have been over capacity in 7 to 10 months of 2024, which is the gap the staffing plan has to close.

## Staffing plan: cost and on-time impact

_Aryan: the plan against the hold-flat baseline, with total cost, on-time estimate and utilization. Call out Transportation separately, with its on-time estimate and cost against the baseline._

## Sensitivity

_Aryan: demand ×0.8 and ×1.2, the snowstorm and the heat wave, each re-planned and with the base plan kept, and what they do to cost and on-time._

## Equity, and resident versus employee requests

We measured on-time performance with the city's own `on_time` flag for 2024. Requests with no target date are always flagged on time, which matters most for Public Works, where 18.6% have no target: its rate is 89.7% over all requests but 87.3% over those with a target. We report the all-requests rate, which matches the README. After merging duplicate neighborhood names, Transportation has by far the widest gap between neighborhoods: 37% on time in West Roxbury against 62% in South Boston, 25.5 points. The gap is 14.1 points for Inspectional Services, whose lowest rates are in Greater Mattapan (64%) and East Boston (65%), 12.2 for Public Works and 9.0 for Parks. We left Chestnut Hill out of this comparison because it has only 12 requests.

We counted Employee Generated and City Worker App requests as employee requests, and the rest as resident requests. On-time rates for the two groups are within 4 points in every department, but employee requests close faster in Public Works (a median of 0.12 days against 0.53) and Transportation (0.88 against 1.56). Parks is the exception: its employee requests take longer (2.83 days against 1.90). We didn't adjust for the mix of request types, so some of these gaps may come from neighborhoods and requesters filing different kinds of requests rather than from differences in service.

## Intake assistant

_Nishanth: type and department accuracy on the 30-message test set (18 clear, 5 ambiguous, 4 multi, 3 out of scope), with counts and the misrouted messages._

## Limitations

- The staffing model tracks backlog month by month, so it only approximates which requests become overdue, and how late.
- The backtest has no growth term, so it tests the seasonal shape and level, not the full forecast method.
- The forecast carries Public Works' 2023→2024 drop into 2025. If 2023 was an unusual year, it understates 2025 demand.
- On-time rates count requests with no target date as on time, which flatters Public Works.
- _Aryan and Nishanth: add the limitations of the optimizer and the intake assistant._

## How we used AI

_Filled in at BOS-7: the same paragraph and table as the notebook and README._
