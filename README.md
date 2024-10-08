FMCG - Dashboard

### Dashboard Link : https://app.powerbi.com/view?r=eyJrIjoiNTgzOTQ3YjUtMDVjZi00ZTk5LWJiNzctM2I5OTI5ODdkNzM5IiwidCI6IjZhNWY5M2Q2LTJjNzItNGM3Ni05ZGM2LWI4MTVhYmMyYTU5ZCJ9

## Problem Statement 
Problem: Analyzing the Effectiveness of AtliQ Mart Promotions
AtliQ Mart launched a promotional campaign across its 50 stores during the Diwali 2023 and Sankranti 2024 festive seasons, focusing on their AtliQ branded products. To optimize future promotional strategies, the sales director seeks to understand the performance of these promotions.

## Objective:
Assess Promotional Effectiveness: how different promotions impacted overall sales and customer engagement.
Identify High-Performing Promotions: Highlighting promotions that boost sales and customer engagement.
Understand Underperforming Promotions: Identify which promotions did not achieve the desired outcomes and understand the potential reasons behind them. 
Provide Insights for Future Planning: Provide insights and recommendations based on the analysis to optimize future promotional efforts and enhance overall sales performance.


## Dataset:
![datasets](https://github.com/user-attachments/assets/6c0d9346-a38c-434a-83de-a5e692190bb7)


## Data Analysis using DAX Measures:

## A Measure was created to find the Total Revenue generated after the promotion

Revenue_After_Promo = SUM(fact_events[quantity_sold(after_promo)])* SUM(fact_events[base_price])

## A Measure was created to find the Total Revenue generated before the promotion

Revenue_Before_Promo = SUM(fact_events[quantity_sold(before_promo)])* SUM(fact_events[base_price])

## A Measure is created to find the Total Revenue generated (before and after the promotion)

Total_Revenue = [Rev_After_Promo]+[Rev_Before_Promo]

## A Measure is created to find the no of Total units sold all togother

Tot_Units_Sold = SUM(fact_events[quantity_sold(before_promo)])+ SUM(fact_events[quantity_sold(after_promo)])

## A Measure is created to find out the increased amount revenue after promotion and the increased percentage
Increased_Revenue = [Rev_After_Promo]-[Rev_Before_Promo]

Increased_Revenue% = DIVIDE([Rev_After_Promo]-[Rev_Before_Promo], [Rev_Before_Promo],0)*100

## A Measure is created to find out the increased units sold after promotion and the increased percentage
Increased_Sold_Units = SUM(fact_events[quantity_sold(after_promo)]) - SUM(fact_events[quantity_sold(before_promo)])

Increased_sold_unit% = VAR PREV = SUM(fact_events[quantity_sold(before_promo)])
RETURN DIVIDE([ISU], PREV,0)

## A Measure is created to find out the promotional revenue contribution ratio over the total Revenue 
Promotion_Penetration_Rate = DIVIDE([Rev_After_Promo], [Total_Rev], 0)


## A Measure is created to find out what is the growth in average item per transaction
Growth_in_Avg_Item_per_Transaction = 
 var Avg_Quantity_Sold_After_Promo = DIVIDE(SUM(fact_events[quantity_sold(after_promo)]), COUNT(fact_events[event_id]))
 var Avg_Quantity_Sold_Before_Promo = DIVIDE(SUM(fact_events[quantity_sold(before_promo)]),COUNT(fact_events[event_id]))
RETURN
DIVIDE(Avg_Quantity_Sold_After_Promo - Avg_Quantity_Sold_Before_Promo, Avg_Quantity_Sold_Before_Promo, 0)

## Three page report is created on Power BI Desktop & it was then published to Power BI Service to find the Store Performance Insight, Promotion Type Insight and Product and category Insight

Following inferences can be drawn from the dashboard;
### [1] Total sum of Revenue generated was 533633M
### [2] Total Revenue generated before the promotion was 173083M
### [3] Total Revenue generated After the promotion was 360550M
### [4] Increased amount Revenue Post promotion was 187467M
### [5] Increased revenue Percentage was 108%
### [6] Out of the Total Revenue, promotion penetration rate was 67.5%
### [7] Growth in Average item per Transaction was 108.31%
### [8] Total Number of Units sold were 645K
### [9] The Number of units sold before promotion was 209k
### [10] The Number of units sold After promotion was 435k
### [11] The Total incresaed units sold were 226k

### Ad-hoc Business Analysis using SQL 

### Provide a list of products with a base price greater than 500 and that are featured in promo type of 'BOGOF' (Buy One Get One Free). 

WITH cte AS 
(SELECT e.event_id, e.store_id, e.campaign_id, e.product_code, e.base_price, e.promo_type,            e.quantity_sold_before_promo, e.quantity_sold_after_promo, 
p.product_name, p.category   
FROM fmcg.fact_events AS e    
JOIN fmcg.dim_products AS p    
ON e.product_code = p.product_code)
SELECT distinct product_name , base_price, promo_type 
FROM cte 
WHERE base_price > 500 and
 promo_type = "BOGOF";

 ###  Generate a report that provides an overview of the number of stores in each city. 

select distinct(city), count(store_id) AS Num_of_Store 
FROM fmcg.dim_stores 
group by city 
order by Num_of_Store Desc;

 ###  Generate a report that displays each campaign along with the total revenue generated before and after the campaign. 

 WITH CTE AS 
(select distinct(C.campaign_name), SUM(F.quantity_sold_before_promo) as Qty_Before_promo, 
SUM(F.quantity_sold_after_promo) as qty_after_promo,
SUM(base_price) as base_price
FROM fmcg.dim_campaigns AS C
JOIN fmcg.fact_events AS F 
ON C.campaign_id = F.campaign_id 
GROUP BY C.campaign_name)  
SELECT campaign_name, round((Qty_Before_promo*base_price)/1000000,0) AS Rev_before_promo, round((qty_after_promo*base_price)/1000000,0) AS Rev_After_promo from CTE;

 ###  Produce a report that calculates the Incremental Sold Quantity (ISU%) for each category during the Diwali campaign. Additionally, provide rankings for the categories based on their ISU%. 

 WITH CTE AS 
(SELECT F.event_id, F.store_id, F.campaign_id, F.product_code,            F.quantity_sold_before_promo,      F.quantity_sold_after_promo, C.campaign_name,            P.product_name, P.category     
FROM fmcg.fact_events AS F    
JOIN fmcg.dim_campaigns AS C       
ON C.campaign_id = F.campaign_id    
JOIN fmcg.dim_products AS P        
ON P.product_code = F.product_code    
WHERE C.campaign_name = 'Diwali’) 
SELECT category,  MAX(IR_PERC) AS Max_ISU_PERC,       
RANK() OVER (ORDER BY MAX(IR_PERC) DESC) AS Ranking FROM 
(SELECT product_name, category,         ((quantity_sold_after_promo - quantity_sold_before_promo) / quantity_sold_before_promo) * 100 AS IR_PERC,            campaign_name    FROM CTE) AS SUB_QUERY
GROUP BY category
ORDER BY Max_ISU_PERC DESC;

 ### Create a report featuring the Top 5 products, ranked by Incremental Revenue Percentage (IR%), across all campaigns. 

WITH CTE AS  
(SELECT F.product_code,   P.product_name, P.category, SUM(F.quantity_sold_before_promo * F.base_price) AS Rev_before_promo,            
SUM(F.quantity_sold_after_promo * F.base_price) AS Rev_after_promo    
FROM fmcg.fact_events F   
JOIN fmcg.dim_products P        
ON F.product_code = P.product_code    
GROUP BY F.product_code, P.product_name, P.category)
SELECT product_name, category,       ROUND(((Rev_after_promo - Rev_before_promo) / Rev_before_promo) * 100, 2) AS IR_Perc,     
RANK() OVER (ORDER BY ((Rev_after_promo - Rev_before_promo) / Rev_before_promo) DESC) AS Ranks
FROM CTE
ORDER BY IR_Perc DESC
LIMIT 5;


Dashboard_Image 
![Store_Performance_Insight](https://github.com/user-attachments/assets/21e7c02f-7f37-4ee6-8d00-106cd5aa0dfe)
![Promotion_Type_Insights](https://github.com/user-attachments/assets/e7e6ec44-d06e-4499-8c35-9d2cb76d5555)
![Product_Category_Insights](https://github.com/user-attachments/assets/92566f26-74b0-45b5-a096-607baca7a3a0)


Executive Summary

BOGOF (Buy One Get One Free) has generated the highest incremental revenue across multiple product categories. customers are highly responsive to this offer. This promotion should be heavily marketed for high-selling items.

The 500 Cashback promotion also resulted in significant revenue growth, particularly for high-value product combos

Promotions like 25% OFF were ineffective. Customers may not find these discounts substantial enough to drive action. We can reduce smaller discount offers like 25% OFF and focus on higher percentage discounts or different. For smaller promotions, consider other value-adds like loyalty points or bundling with high-performing items to stimulate demand without excessive discounting, which can also maintain margins without over-discounting.

Gathering feedback on how customers perceive promotions could further clarify why some promotions work better than others.

Ensure that promotions are targeted to the appropriate audience for each product and category.

Apply the successful elements of the Sankranti campaign to future promotions, especially for high-traffic sales periods like Diwali, to achieve higher revenue gains.
