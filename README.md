# Microsoft 9 Data Analyst Interview Ans:


---

### 1) Most Popular Client ID
Select the most popular client_id based on a count of the number of users who have at least 50% of their events from the following list: 'video call received', 'video call sent', 'voice call received', 'voice call sent'. Dataset:evens_data.csv
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
Write a query that returns the company (customer_id column) with the highest number of users that use desktop only.  Dataset: mixed_customer_data.csv

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
Write a query that returns a list of the bottom 2 companies by mobile usage. Company is defined in the customer_id column. Mobile usage is defined as the number of events registered on a client_id == 'mobile'. Order the result by the number of events ascending. In the case where there are multiple companies tied for the bottom ranks (rank 1 or 2), return all the companies. Output the customer_id and number of events. Dataset  =  customer_event_data.csv

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
Write a query that returns a number of users who are exclusive to only one client. Output the client_id and number of exclusive 
dataset:user_client_data.csv

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
Write a query that returns the number of unique users per client per month. dataset:unique_user.csv

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
Calculate the share of new and existing users for each month in the table. Output the month, share of new users, and share of existing users as a ratio. New users are defined as users who started using services in the current month (there is no usage history in previous months). Existing users are users who used services in the current month but also used services in any previous month. Assume that the dates are all from the year 2020.

#### Query
```sql
WITH user_month AS (
    SELECT
        user_id,
        SUBSTRING(time_id, 1, 7) AS month
    FROM LoanDB.`events_data-2`
    WHERE SUBSTRING(time_id, 1, 4) = '2020'
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

