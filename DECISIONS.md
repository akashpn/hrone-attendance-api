# Decisions

1. Indexes: I added a unique index on emp_code for the employees collection, and a unique compound index on {emp_code: 1, date: -1} for attendance_logs. I also indexed the date field. This makes the range queries for analytics fast and stops duplicate punch-ins at the db level.

2. Concurrent punch-ins: The compound unique index handles this atomically. The first request inserts the doc. The second one hits a DuplicateKeyError at the storage level, which the code catches and turns into a 409 Conflict. 

3. Leaderboard ties: I used $rank inside a $setWindowFields stage. Ties get the same rank number (e.g. 1, 2, 2, 4) and are then sorted by emp_code. The limit is applied after, so if there's a tie at the cutoff, it just drops the alphabetically later employee.

4. Department headcount: The pipeline starts on the employees collection and does a $lookup into attendance_logs. Since every employee is included, we get the real headcount using $addToSet. For the averages, I used $ifNull to make sure employees with no logs add 0 to the log count so the math doesn't skew.

5. Scaling to 100x data: Doing $lookup and $densify on the fly will eventually get too slow. I'd move to pre-aggregating the daily/monthly stats using MongoDB change streams and store them in a separate collection.