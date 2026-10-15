# REVIEW.md

## Defects Found & Fixed

1. **Missing Employee Validation in Punch-In**
   - **Where:** `app/main.py` in `punch_in()`
   - **What is wrong:** If an invalid `emp_code` is submitted, `db.employees.find_one()` returns `None`, and trying to access `emp["shift_start"]` immediately raises an unhandled `KeyError` resulting in a 500 error instead of a 404.
   - **Fix:** Added an explicit check `if not emp: raise HTTPException(404, "Employee not found")`.

2. **Incorrect Pagination Skip Formula**
   - **Where:** `app/main.py` in `list_employees()`
   - **What is wrong:** `skip = page * page_size` was used. For page 1 with page_size 20, it skipped the first 20 records (`20 * 1 = 20`).
   - **Fix:** Corrected offset formula to `skip = (page - 1) * page_size`.

3. **In-Memory Slicing & Naive Sorting in Attendance List**
   - **Where:** `app/main.py` in `list_attendance()`
   - **What is wrong:** The starter fetched all documents into memory via `list(db.attendance_logs.find(q))`, sorted them in Python, and sliced them manually. This fails to scale to 100,000 records.
   - **Fix:** Pushed sorting, `.skip()`, and `.limit()` directly into the MongoDB query cursor.

4. **Timezone Naivety & Timestamp Truncation (Rule R1)**
   - **Where:** `app/main.py` helper functions
   - **What is wrong:** Datetimes created via `datetime.fromtimestamp()` or `datetime.now()` were naive and lacked proper IST timezone binding and microsecond truncation.
   - **Fix:** Enforced `tz=IST` conversion and truncated microsecond fields across calculations.