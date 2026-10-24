# DATA MART SALES PERFORMANCE ANALYSIS

 ## Table Of Contents
  - [Project Overview](#project-overview)
  - [Data Source](#data-source)
  - [Tool](#tool)
  - [Data-Cleaning/Preparation](#data-cleaningpreparation)
  - [Data-Exploration](#data-exploration)
  - [Data-Analysis](#data-analysis)
  - [Results/Findings](#resultsfindings)
  - [Recommendations](#recommendations)
  - [Limitations](#limitations)

## PROJECT OVERVIEW

This data analysis project aims to help Danny analyse his sales performance after introducing a new sustainable methods of packaging. By analysing the various aspects of the dataset, we seek to understand the impact of the changes introduced in June of 2020, which platform, region, segment and customer types most affected and how best to minimise the impact on sales in the future.

### DATA SOURCE 
 Data Mart weekly_Sales: The dataset used was provided by Danny and it contains detailed information about each sale made by the company 

### TOOL
  - SQL - Data cleaning and analysis

### DATA CLEANING/PREPARATION
The data cleaning process includes
- Converting week_date from text to date datatype
- Creating new columns such as week_number, month_number, calendar_year from the week_date
- Creating new columns such as age_band, demographic from the segment
- Replacing all null values and blank cells with Unknown
- Creating an average transaction column from both sales and transaction columns

### DATA EXPLORATION 
The EDA involved exploring the dataset to answer key questions such as
- What is the total sales for each region for each month?
- What is the total count of transactions for each platform?
- What is the percentage of sales for Retail vs Shopify for each month?
- Which age_band and demographic values contributes the most to Retail Sales?
- What is the percentage of sales by demographic for each year in the dataset?

### DATA ANALYSIS 
This was done to understand the impact of the sustainable packaging change on sales performance before and after it came into effect. This includes answering questions like:
- What the total sales is four weeks before and after the introduction? What the growth/reduction rate in actual value and percentage of sales is?
```SQL
WITH total_4weeks_sale as (
                            Select
                                  Case
                                        When week_date > = dateadd(week, -4, '2020-06-15') and weekd_date < '2020-06-15'
                                        Then cast(sales as bigint) end) as before_sales,

                                 Case
                                     When week_date >= '2020-06-15' and week_date < dateadd(week, 4, '2020-06-15')
                                     Then cast(sales as bigint) end) as after_sales
                              From clean_weekly_sales_view
)
Select before_sales, after_sales,
before_sales - after_sales as sales_diff,
Cast(after_sales - before_sales) * 100.0/before_sales as decimal(10,2)) as pct_growth
from total_4weeks_sale;
```
- What the total sales is 12 weeks before and after the introduction of the sustainable packaging methods?
```SQL
With weeks_12 as (
					 select
							SUM(CASE
									 When week_date >= dateadd(week, -12, '2020-06-15') and week_date < '2020-06-15'
									 Then cast(sales as bigint) end) as before_12_sales,
					 
							 SUM(CASE
									 WHEN week_date >= '2020-06-15' and week_date < DATEADD(week, 12, '2020-06-15')
									 Then cast(sales as bigint) end) as after_12_sales

					from clean_weekly_sales_view
)
Select before_12_sales, after_12_sales,
after_12_sales-before_12_sales as sales_diff,
cast((after_12_sales-before_12_sales)*100.0/before_12_sales as decimal(10,2)) as pct_growth
from weeks_12;
```
- How does this before and after 12 weeks sales compare in previous years (2018 and 2019) ?
```SQL
WITH sales_metric AS (
 SELECT 
  calendar_year,
  SUM(CASE
   WHEN week_number BETWEEN 12 AND 23 
   THEN CAST(sales AS BIGINT) END) AS Before_12,
  SUM(CASE 
   WHEN week_number BETWEEN 24 AND 35
   THEN CAST(sales AS BIGINT) END) AS after_12
 FROM data_mart.clean_weekly_sales_view
 WHERE calendar_year IN (2018,2019,2020)
  AND week_number BETWEEN 12 AND 35
 GROUP BY calendar_year
)
SELECT 
  calendar_year,
  Before_12, 
  after_12,
  after_12 - Before_12 AS sales_diff,
  CAST((after_12 - Before_12)*100.0 / Before_12 AS DECIMAL(10,2)) AS pct_growth
FROM sales_metric
ORDER BY calendar_year;
```
- To understand which area of the business had the highest negative impact in sales metrics that year (2020) for the 12 weeks before and after period.
```SQL
With sales_metric_impact as (
						select 
								Calendar_year,
								region,
								platform, 
								age_band,
								demographic,
								customer_type,
								sum(CASE
									WHEN week_number between 12 and 23 
									Then cast(sales as bigint) end) as Before_12,

						Sum(CASE 
							WHEN week_number between 24 and 35
							Then cast(sales as bigint) end) as after_12
					From clean_weekly_sales_view
					WHERE calendar_year = 2020
						and week_number between 12 and 35
					Group by calendar_year,
							 region,
							 platform, 
							 age_band,
							 demographic,
							 customer_type
)
Select TOP 10 Calendar_year,region,
	  platform, age_band,
	demographic,customer_type,
Before_12, after_12,
after_12-before_12 as sales_diff,
cast((after_12-before_12)*100.0/before_12 as decimal(10,2)) as pct_growth
from sales_metric_impact
Order by pct_growth, sales_diff asc;
```

### RESULTS/FINDINGS
The analysis results showed that:

1. There was a decline in both percentage and dollar value four weeks after the packaging was introduced and when extended to twelve weeks period the decline was pronounced. This shows the impact deepened over time 

2. When the twelve week period was compared across previous years(2018, 2019 and 2020), it showed growth was already slowing from 10.54% to 8.30% before packaging. In 2020, it slowed further to 6.54%. This drop is seasonal and the change is extra.

3. When the 12 weeks period was broken down across business areas, South-America region, Shopify platform, Retirees (age-band), Families (demographic) and New customer_type felt the highest impact negatively.

### RECOMMENDATIONS
1. Based on the 4 and 12-week before and after analysis, the decline was larger over 12 weeks (-2.15%) than 4 weeks (-1.18%), and growth in the same period slowed to 6.54% in 2020 compared to 10.54% in 2018 and 8.30% in 2019. To minimise future impact, I recommend communicating the change to customers at least 4 weeks before rollout and adopting a phased regional rollout, starting with less impacted regions such as Oceania.
   
2. The analysis shows Shopify customers were the most impacted, accounting for 6 of the top 10 declines by percentage. To address this, I recommend A/B testing the sustainability badge wording on the Shopify store, adding a clear FAQ explaining why packaging changed, and considering free shipping to offset any perceived drop in value.
   
3. The highest negative impact was concentrated in South America, particularly among Retirees and Families customer segments who are New customers. I recommend reviewing the South America supply chain for any shelf-life or freight issues linked to the new packaging, and creating targeted communication for Families and Retirees explaining the benefits of sustainable packaging, with a dedicated onboarding journey for New customers to improve retention.

### LIMITATIONS
1. Too many unknown customers, this doesn't give us the full idea of our age_band and demographic. It'd be better to make age and demographic fields required during check-out

2. We only have sales and no profit, this means we only know sales went down but don't know if the sustainable package cost more. Profit could be down more than sales. Our data only shows one side of the money

3. Our data is weekly and not daily, we can't tell if customers visits less often or same time and bought less each time. We need to know if customers are visiting less or spending less per visit.
