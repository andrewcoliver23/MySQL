WITH months AS (
SELECT 
'2017-01-01' AS first_day,
'2017-01-31' AS last_day
UNION
  SELECT 
'2017-02-01' AS first_day,
'2017-02-31' AS last_day
  UNION 
  SELECT 
  '2017-03-01' AS first_day,
'2017-03-31' AS last_day
  FROM subscriptions
),
cross_join AS (
SELECT *
  FROM subscriptions
  CROSS JOIN months 
),

status AS (
SELECT id, first_day AS 'month', 
  CASE 
  WHEN (segment = 87)
  AND (subscription_start < first_day)
  AND ((subscription_end > first_day)
    OR (subscription_end IS NULL)
  )
  THEN 1
  ELSE 0
  END AS is_active_87,
CASE 
  WHEN (segment = 30)
  AND 
    (subscription_start < first_day)
  AND ((subscription_end > first_day)
    OR (subscription_end IS NULL)
  )
  THEN 1
  ELSE 0
  END AS is_active_30,
  CASE 
  WHEN (segment = 87)
       AND (subscription_end BETWEEN first_day AND last_day)
  THEN 1
  ELSE 0 
  END AS not_active_87,
  CASE
  WHEN (segment = 30)
  AND (subscription_end BETWEEN first_day AND last_day)
  THEN 1
  ELSE 0
  END AS not_active_30
  FROM cross_join

),
status_aggregate AS (
SELECT month, SUM(is_active_87) AS Active87, SUM(is_active_30)AS Active30, SUM(not_active_87) AS Canceled87, SUM(not_active_30) AS Canceled30
  FROM status 
  GROUP BY month
  

)

SELECT month, ROUND((100* Canceled87/Active87), 2) AS 'CHURN RATE% SEGMENT 87 ', ROUND((100 * Canceled30/Active30), 2) AS 'CHURN RATE% SEGMENT 30 '
FROM status_aggregate;
