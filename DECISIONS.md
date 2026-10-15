# DECISIONS.md  (about 150-200 words in total - plain language, your own words)

1. **Indexes Created:** 
   Created a compound unique index on `attendance_logs` for `(emp_code, date)` to enforce uniqueness and make daily lookups instantaneous. Also indexed `employees.emp_code` and `attendance_logs.date` to support fast range filters and aggregation pipelines.

2. **Concurrent Duplicate Punch-Ins:**
   When two identical punch-in requests arrive simultaneously, the unique compound index `(emp_code, date)` on MongoDB prevents duplicate insertion at the database level by throwing a duplicate key error, which our app catches to return a clean `409 Conflict`.

3. **Leaderboard Ties:**
   Ties are handled using MongoDB window functions (`$denseRank`), ensuring tied employees share the exact same rank and subsequent ranks skip numbers appropriately (e.g., 1, 2, 2, 4). Results are deterministically ordered by late count descending and then `emp_code` ascending.

4. **No-Log Employees in Department Summary:**
   The department summary aggregation uses a `$lookup` combined with a `$match` condition checking `joined_on` dates, ensuring employees with zero attendance logs are still correctly counted toward departmental headcount (Rule R9).

5. **Scaling for 100x Data:**
   For 10,000,000+ records, we would introduce compound indexes covering date ranges and department facets, shard collections by `department` or `emp_code`, and implement server-side read/cache layers for heavy analytics.