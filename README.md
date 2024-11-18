# Meta Business Intelligence Analyst Interview Ans:


---

### 1) Most Popular Client ID
Find the most popular `client_id` based on the number of users who have at least 50% of their events as **video/voice calls**.

#### Query
```sql
WITH cte AS (
    SELECT
        Client_ID,
        User_ID,
        COUNT(CASE WHEN Event_Type IN ('video call received', 'video call sent', 'voice call received', 'voice call sent') THEN 1 ELSE NULL END) AS event_cnt,
        COUNT(Event_Type) AS total_event_cnt
    FROM LoanDB.events_data
    GROUP BY Client_ID, User_ID
),
cte2 AS (
    SELECT
        Client_ID,
        User_ID
    FROM cte 
    WHERE event_cnt >= 0.5 * total_event_cnt
)
SELECT
    Client_ID,
    COUNT(User_ID) AS user_cnt
FROM cte2
GROUP BY Client_ID
ORDER BY user_cnt DESC
LIMIT 1;
```

---

### 2) Desktop-Only Users
Find the company (`customer_id`) with the highest number of users that use desktop exclusively.

#### Query
```sql
SELECT
    customer_id AS company
FROM (
    SELECT
        customer_id,
        COUNT(user_type) AS desktop_user_cnt
    FROM LoanDB.mixed_customer_data
    WHERE user_type = 'desktop'
    GROUP BY customer_id
    ORDER BY desktop_user_cnt DESC
    LIMIT 1
) a;
```

---

### 3) Bottom Companies by Mobile Usage
Identify the bottom 2 companies by mobile usage (events where `client_id` = 'mobile'). If there is a tie, return all companies with the same rank.

#### Query
```sql
WITH cte AS (
    SELECT
        customer_id,
        COUNT(event_type) AS numbers_of_events
    FROM LoanDB.customer_event_data
    WHERE event_type = 'mobile'
    GROUP BY customer_id
),
cte2 AS (
    SELECT
        customer_id,
        numbers_of_events,
        DENSE_RANK() OVER (ORDER BY numbers_of_events ASC) AS bottom_rank
    FROM cte
)
SELECT
    customer_id,
    numbers_of_events
FROM cte2
WHERE bottom_rank <= 2;
```

---

### 4) Exclusive Users per Client
Return the number of users who are exclusive to only one client.

#### Query
```sql
SELECT
    client_id,
    COUNT(user_id) AS exclusive_user_count
FROM (
    SELECT
        user_id,
        COUNT(client_id) AS number_of_clients
    FROM LoanDB.user_client_data
    GROUP BY user_id
    HAVING number_of_clients = 1
) a
GROUP BY client_id;
```

---

### 5) Unique Users per Client per Month
Calculate the number of unique users for each `client_id` per month.

#### Query
```sql
SELECT
    client_id,
    month,
    COUNT(DISTINCT user_id) AS unique_user_count
FROM LoanDB.unique_user
GROUP BY client_id, month;
```

---

### 6) Monthly User Share (New vs. Existing)
Calculate the monthly share of **new** and **existing** users in the dataset.

#### Query
```sql
WITH user_month AS (
    SELECT
        user_id,
        SUBSTRING(time_id, 1, 7) AS month
    FROM LoanDB.`events_data-2`
    WHERE SUBSTRING(time_id, 1, 4) = '2024'
    GROUP BY user_id, month
),
first_month AS (
    SELECT
        user_id,
        MIN(month) AS min_month
    FROM user_month
    GROUP BY user_id
),
new_existing_users AS (
    SELECT
        um.month,
        COUNT(CASE WHEN um.month = fm.min_month THEN 1 END) AS new_users,
        COUNT(CASE WHEN um.month > fm.min_month THEN 1 END) AS existing_users
    FROM user_month um
    JOIN first_month fm ON fm.user_id = um.user_id
    GROUP BY um.month
)
SELECT
    month,
    CONCAT(new_users, ':', existing_users) AS new_and_existing_ratio
FROM new_existing_users;
```

