# Chicago Taxi Market Analysis

![Demo](https://viet-in-tech.github.io/chicago-taxi-demo.gif)
### SQL, EDA & Hypothesis Testing

**Portfolio write-up:** [Does Chicago Weather Tax Your Ride? SQL, EDA & Hypothesis Testing](https://viet-in-tech.github.io/chicago-taxi-eda.html)

**TripleTen Data Science Program · Sprint 7 — SQL, EDA & Hypothesis Testing**

**Completed:** November 3, 2025

---

## Project Overview

This project analyzes Chicago's taxi market using SQL-sourced data and conducts a formal statistical hypothesis test to determine whether bad weather significantly affects ride duration on the Loop-to-O'Hare route.

Two phases:
1. **EDA** — Identify market dominance patterns across 64 taxi companies and dropoff concentration across 94 Chicago neighborhoods
2. **Hypothesis Testing** — Test whether bad weather statistically significantly increases Loop-to-O'Hare Saturday ride duration

---

## Datasets

All data sourced from SQL queries against a Chicago taxi database:

| Dataset | Records | Key Columns |
|---------|---------|-------------|
| `project_sql_result_01.csv` | 64 companies | `company_name`, `trips_amount` |
| `project_sql_result_04.csv` | 94 neighborhoods | `dropoff_location_name`, `average_trips` |
| `project_sql_result_07.csv` | 1,068 rides | `start_ts`, `weather_conditions`, `duration_seconds` |

---

## SQL Queries

Three tables in a relational Chicago taxi database: `trips`, `cabs`, `weather_records`.

### Query 1 — Rides per Taxi Company (Dataset 1)
```sql
SELECT
    cabs.company_name,
    COUNT(trips.trip_id) AS trips_amount
FROM
    cabs
    INNER JOIN trips ON cabs.cab_id = trips.cab_id
WHERE
    CAST(trips.start_ts AS date) BETWEEN '2017-11-15' AND '2017-11-16'
GROUP BY
    cabs.company_name
ORDER BY
    trips_amount DESC
```

### Query 2 — Market Segmentation with CASE
```sql
SELECT
    COUNT(trips.trip_id) AS trips_amount,
    CASE
        WHEN cabs.company_name = 'Flash Cab' THEN 'Flash Cab'
        WHEN cabs.company_name = 'Taxi Affiliation Services' THEN 'Taxi Affiliation Services'
        ELSE 'Other'
    END AS company
FROM
    trips
    INNER JOIN cabs ON cabs.cab_id = trips.cab_id
WHERE
    CAST(trips.start_ts AS date) BETWEEN '2017-11-01' AND '2017-11-07'
GROUP BY
    cabs.company_name = 'Flash Cab',
    cabs.company_name = 'Taxi Affiliation Services'
ORDER BY
    trips_amount DESC
```

### Query 3 — Loop-to-O'Hare Saturday Rides with Weather (Dataset 3)
```sql
SELECT
    trips.start_ts AS start_ts,
    trips.duration_seconds,
    CASE
        WHEN weather_records.description LIKE '%rain%'
          OR weather_records.description LIKE '%storm%' THEN 'Bad'
        ELSE 'Good'
    END AS weather_conditions
FROM
    trips
INNER JOIN
    weather_records ON weather_records.ts = trips.start_ts
WHERE
    EXTRACT(DOW FROM trips.start_ts) = 6   -- Saturday only
    AND trips.pickup_location_id = 50       -- Loop
    AND trips.dropoff_location_id = 63      -- O'Hare
ORDER BY
    trips.trip_id
```

---

## Workflow

### Phase 1 — Exploratory Data Analysis

**Top 10 Taxi Companies (November 15–16, 2017)**

| Rank | Company | Rides |
|------|---------|-------|
| 1 | Flash Cab | 19,558 |
| 2 | Taxi Affiliation Services | 11,422 |
| 3 | Medallion Leasing | 10,367 |
| 4 | Yellow Cab | 9,888 |
| 5 | Taxi Affiliation Service Yellow | 9,299 |

Flash Cab led by ~71% over second place — significant market concentration.

**Top 10 Neighborhoods by Average Dropoffs (November 2017)**

| Rank | Neighborhood | Avg Dropoffs |
|------|-------------|-------------|
| 1 | Loop | 10,727.5 |
| 2 | River North | 9,523.7 |
| 3 | Streeterville | 6,664.7 |
| 4 | West Loop | 5,163.7 |
| 5 | O'Hare | 2,546.9 |

### Phase 2 — Hypothesis Testing

**Hypotheses:**
- H₀: Average Loop-to-O'Hare ride duration is the same in good and bad weather
- H₁: Average duration is longer in bad weather
- α = 0.05

**Descriptive Statistics:**

| Weather | n | Mean | Mean (min) | Std Dev |
|---------|---|------|-----------|---------|
| Good | 882 | 2,013.3s | 33.6 min | 743.6 |
| Bad | 180 | 2,427.2s | 40.5 min | 721.3 |

Difference: **+413.9 seconds (6.9 minutes, +20.6%)** in bad weather.

---

## Final Results

| Metric | Value |
|--------|-------|
| t-statistic | 6.8405 |
| p-value | ≈ 0.000000 |
| Decision | **Reject H₀** |
| Practical Effect | +6.9 min (+20.6%) |

**Conclusion:** Bad weather statistically significantly increases Loop-to-O'Hare Saturday ride duration (p ≪ α = 0.05).

---

## Technologies

- **SQL** — data extraction (JOINs, aggregations, CASE expressions)
- Python 3.8 · pandas · numpy
- matplotlib · seaborn
- scipy.stats (ttest_ind)

---

## Project Status

✅ Approved — TripleTen Data Science Program, Sprint 7
