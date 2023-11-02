# RFM-Analysis

[Tableau Dashboard](https://public.tableau.com/app/profile/mfernandezcean/viz/Sales_Dashboard_1_16988765103380/Dashboard1)

![Dashboard 1](https://github.com/mfernandezcean/Marketing_Campaign_Results/assets/105746149/d0f9210f-f7b8-40d9-b898-8e09ac629c5e)


## SQL Code: 

All data:
```
Select 
  * 
from 
  sales_data_sample;

```

![1](https://github.com/mfernandezcean/Marketing_Campaign_Results/assets/105746149/4033e832-a2bb-4846-a661-123eac2ac0e0)
---
Distinct Count of key Columns: 
```
SELECT DISTINCT STATUS FROM sales_data_sample;
SELECT DISTINCT YEAR_ID FROM sales_data_sample;
SELECT DISTINCT PRODUCTLINE FROM sales_data_sample;
SELECT DISTINCT COUNTRY FROM sales_data_sample;
SELECT DISTINCT DEALSIZE FROM sales_data_sample;
SELECT DISTINCT TERRITORY FROM sales_data_sample;
```
![2](https://github.com/mfernandezcean/Marketing_Campaign_Results/assets/105746149/4ec81483-bd8a-451a-978f-4951299e1852)
---
Product Line Revenue:
```
SELECT PRODUCTLINE,  sum(sales) AS Revenue 
FROM sales_data_sample
GROUP BY PRODUCTLINE
ORDER BY 2 desc;
```
![3](https://github.com/mfernandezcean/Marketing_Campaign_Results/assets/105746149/c6cfa7f7-c248-40d7-802d-85f6c919ca41)
---
Year Revenue:
```
SELECT YEAR_ID,  sum(sales) AS Revenue 
FROM sales_data_sample
GROUP BY YEAR_ID
ORDER BY 2 desc;
```
![4](https://github.com/mfernandezcean/Marketing_Campaign_Results/assets/105746149/6d17ae03-0d9e-4a7e-947f-c4993c34231c)
---
Deal Size Revenue:
```
SELECT DEALSIZE,  sum(sales) AS Revenue 
FROM sales_data_sample
GROUP BY DEALSIZE
ORDER BY 2 desc;
```
![5](https://github.com/mfernandezcean/Marketing_Campaign_Results/assets/105746149/8d0db57a-955c-4fdc-a5fd-31f2072fdc6d)
---
Revenue by Months:
```
SELECT MONTH_ID,  sum(sales) AS Revenue, count(ORDERNUMBER) AS Frequency
FROM sales_data_sample
--WHERE YEAR_ID = 2005
GROUP BY MONTH_ID
ORDER BY 2 DESC; 
```
![6](https://github.com/mfernandezcean/Marketing_Campaign_Results/assets/105746149/145bf2d1-da4f-42ab-9f80-471d43938cd5)
---
Customers Monetary Value, Recency and Frequency:
```
SELECT 
	CUSTOMERNAME,
	sum(sales) MonetaryValue,
	avg(sales) AvgMonetaryValue,
	count(ORDERNUMBER) Frequency,
	max(ORDERDATE) last_order_date,
	(select max(ORDERDATE) from sales_data_sample) max_order_date,
	DATEDIFF(DD,max(ORDERDATE),(select max(ORDERDATE) from sales_data_sample)) Recency 
FROM sales_data_sample
Group By CUSTOMERNAME;
```
![7](https://github.com/mfernandezcean/Marketing_Campaign_Results/assets/105746149/d84b6682-27cb-4a09-af3c-7ae23298f7a9)
---
Creatin of Table with Recency, Frequency and Monetary Parameters:
```
DROP TABLE IF EXISTS #rfm;
with rfm as 
(
SELECT 
	CUSTOMERNAME,
	sum(sales) MonetaryValue,
	avg(sales) AvgMonetaryValue,
	count(ORDERNUMBER) Frequency,
	max(ORDERDATE) last_order_date,
	(select max(ORDERDATE) from sales_data_sample) max_order_date,
	DATEDIFF(DD,max(ORDERDATE),(select max(ORDERDATE) from sales_data_sample)) Recency 
FROM sales_data_sample
Group By CUSTOMERNAME
),
rfm_calc as

(
	select r.*,
		NTILE(4) OVER (order by Recency) rfm_recency,
		NTILE(4) OVER (order by Frequency) rfm_frequency,
		NTILE(4) OVER (order by MonetaryValue) rfm_monetary
	from rfm r
)
select c.*, rfm_recency + rfm_frequency + rfm_monetary as rfm_cell,
cast(rfm_recency as varchar) + cast(rfm_frequency as varchar) + CAST(rfm_monetary as varchar) rfm_cell_string 
into #rfm
from rfm_calc c
```
![8](https://github.com/mfernandezcean/Marketing_Campaign_Results/assets/105746149/42d62fda-b86f-4c2b-8d29-e0d44e291444)
---
Bulding a Case Statment to classify Customers:
```
select CUSTOMERNAME, rfm_recency, rfm_frequency, rfm_monetary,
	case	
		when rfm_cell_string in (111,112,121,122,123,132,211,212,114,141) then 'lost_customer'
		when rfm_cell_string in (133, 134, 143,244,334,343,344, 144) then 'slipping away, cannt lose'
		when rfm_cell_string in (311,411,331) then 'new customers'
		when rfm_cell_string in (222, 223, 233, 322) then 'potential churners'
		when rfm_cell_string in (323, 333, 321, 422, 332, 432) then 'active'
		when rfm_cell_string in (433, 434, 443, 444) then 'loyal'
	end rfm_segment
from #rfm
```
![9](https://github.com/mfernandezcean/Marketing_Campaign_Results/assets/105746149/cd084338-dad6-4c0a-8a12-6f0737abf73e)


