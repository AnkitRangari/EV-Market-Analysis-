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