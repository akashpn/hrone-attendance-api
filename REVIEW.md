### 2. `REVIEW.md`

```markdown

Here is what I found and fixed in the starter code:

1. The /health endpoint was hardcoded to return 200 even if the DB was down. I added a ping command so it actually verifies the mongo connection.
2. Massive race condition in punch_in and create_employee. Doing a find_one followed by insert_one means concurrent requests will bypass the check. I fixed this by adding unique indexes and handling the DuplicateKeyError.
3. list_attendance was loading the entire collection into memory to paginate it. I pushed the skip/limit logic down to MongoDB so it won't crash on 100k docs.
4. Pagination math was wrong in list_employees (skip = page * page_size skips the first 20 records). Changed it to (page - 1).
5. compute_work_hours used python's default round(), which does banker's rounding (round to even). Rule 4 needs round half-up, so I swapped it to use the Decimal module.
6. compute_late_minutes returned 0 if the diff was exactly 10 minutes and a few seconds because of how it floored the math. Fixed the condition to strictly check if seconds > 600.
7. The overnight shift logic was completely broken. It assumed shifts start and end on the same calendar day. I updated the helpers to strictly convert UTC to IST and assign punch-ins to the previous day if they happen before the overnight shift ends.