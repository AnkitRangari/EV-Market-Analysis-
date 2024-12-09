1) Top 3 and bottom 3 makers for the fiscal years 2023 and 2024 in terms of the number of 2-wheelers sold.

```bash
WITH RankedSales AS (
    SELECT d.fiscal_year, m.maker, 
        SUM(m.electric_vehicles_sold) AS total_electric_vehicles_sold,
        ROW_NUMBER() OVER (PARTITION BY d.fiscal_year ORDER BY SUM(m.electric_vehicles_sold) DESC) AS rank_desc,
        ROW_NUMBER() OVER (PARTITION BY d.fiscal_year ORDER BY SUM(m.electric_vehicles_sold) ASC) AS rank_asc
    FROM electric_vehicle_sales_by_makers m
    JOIN dim_date d
    On m.date = d.date
    WHERE d.fiscal_year IN (2023, 2024) AND m.vehicle_category = '2-Wheelers'
    GROUP BY d.fiscal_year, m.maker
)
SELECT fiscal_year, maker, total_electric_vehicles_sold,
    CASE WHEN rank_desc <= 3 THEN 'Top 3'
        WHEN rank_asc <= 3 THEN 'Bottom 3'
    END AS rank_type
FROM RankedSales
WHERE rank_desc <= 3 OR rank_asc <= 3
ORDER BY fiscal_year, rank_type, total_electric_vehicles_sold DESC;
```

2) Identify the top 5 states with the highest penetration rate in 2-wheeler and 4-wheeler EV sales in FY 2024.

```bash
SELECT * 
FROM ( 
SELECT state, vehicle_category,
SUM(electric_vehicles_sold)/sum(total_vehicles_sold)*100 AS penetration_rate
FROM electric_vehicle_sales_by_state AS c
JOIN dim_date AS a
on c.date=a.date 
WHERE a.fiscal_year=2024 and vehicle_category="2-wheelers"
GROUP BY state, vehicle_category
ORDER BY penetration_rate DESC LIMIT 5
) as Two 
UNION ALL 
SELECT * 
FROM ( 
SELECT state, vehicle_category,
SUM(electric_vehicles_sold)/sum(total_vehicles_sold)*100 AS penetration_rate
FROM electric_vehicle_sales_by_state AS c
JOIN dim_date AS a
on c.date=a.date 
WHERE a.fiscal_year=2024 and vehicle_category="4-wheelers"
GROUP BY state, vehicle_category
ORDER BY penetration_rate DESC limit 5
) AS four;
```
