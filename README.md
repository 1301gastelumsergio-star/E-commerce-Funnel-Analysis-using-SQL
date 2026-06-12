# E-commerce-Funnel-Analysis-using-SQL
SQL project analyzing an e-commerce conversion funnel, including conversion rates, customer journey timing, and revenue metrics
# E-commerce Funnel Analysis using SQL

## Project Overview

This project analyzes user behavior across an e-commerce conversion funnel using Google BigQuery. The objective is to understand user progression from page views to purchases, evaluate conversion rates, analyze customer journey timing, and assess revenue performance.

## Tools Used

- Google BigQuery
- SQL 

## Business Questions

- How many users reach each stage of the funnel?
- What are the conversion rates between stages?
- How long does it take users to complete a purchase?
- What revenue metrics can be derived from user behavior?

## SQL Analysis

select *
from sql_practice.user_events
limit 10;

select max(event_date)
from sql_practice.user_events;

select distinct(event_type)
from sql_practice.user_events;

select
      count(distinct(case when event_type = 'page_view' then user_id end)) as stage_1_views,
      count(distinct(case when event_type = 'add_to_cart' then user_id end)) as stage_2_cart,
      count(distinct(case when event_type = 'checkout_start' then user_id end)) as stage_3_checkout,
      count(distinct(case when event_type = 'payment_info' then user_id end)) as stage_4_payment,
      count(distinct(case when event_type = 'purchase' then user_id end)) as stage_5_purchase
from sql_practice.user_events
where event_date >= timestamp(date_sub('2026-02-03', INTERVAL 30 day))
;

-- convertions rates through the funnel

with funnel_stages as(select
      count(distinct(case when event_type = 'page_view' then user_id end)) as stage_1_views,
      count(distinct(case when event_type = 'add_to_cart' then user_id end)) as stage_2_cart,
      count(distinct(case when event_type = 'checkout_start' then user_id end)) as stage_3_checkout,
      count(distinct(case when event_type = 'payment_info' then user_id end)) as stage_4_payment,
      count(distinct(case when event_type = 'purchase' then user_id end)) as stage_5_purchase
from sql_practice.user_events
where event_date >= timestamp(date_sub('2026-02-03', INTERVAL 30 day)))
select  stage_1_views, stage_2_cart,
        round((stage_2_cart/stage_1_views)*100,2) as stage_2_rate,
        stage_3_checkout, round((stage_3_checkout/stage_2_cart)*100,2) as stage_3_rate,
        stage_4_payment, round((stage_4_payment/stage_3_checkout)*100,2) as stage_4_rate,
        stage_5_purchase, round((stage_5_purchase/stage_4_payment)*100,2) as stage_5_rate,
from funnel_stages
 
 -- time to convertions analysis
with user_journey as(select
      user_id,
      min(case when event_type = 'page_view' then event_date end) as views,
      min(case when event_type = 'add_to_cart' then event_date end) as cart,
      min(case when event_type = 'purchase' then event_date end) as purchase
from sql_practice.user_events
where event_date >= timestamp(date_sub('2026-02-03', INTERVAL 30 day))
group by user_id
having purchase is not null)
select count(*) as converted_users,
      round(avg(timestamp_diff(cart, views, minute))) as avg_view_to_cart_minute,
      round(avg(timestamp_diff(purchase, cart, minute))) as avg_cart_to_purchase_minute,
      round(avg(timestamp_diff(purchase, views, minute))) as avg_overall_minute,
from user_journey
 
 -- revenue analysis
 with revenue_funnel as(select
      count(distinct(case when event_type = 'page_view' then user_id end)) as visitors,
      count(distinct(case when event_type = 'purchase' then user_id end)) as buyers,
      round(sum(case when event_type = 'purchase' then amount end),2) as total_revenue,
      count(case when event_type = 'purchase' then 1 end) as total_orders
from sql_practice.user_events
where event_date >= timestamp(date_sub('2026-02-03', INTERVAL 30 day)))
select visitors,buyers,total_revenue,total_orders,
      round(total_revenue/total_orders) as avg_order_amount,
      round(total_revenue/buyers) as revenue_per_buyer,
      round(total_revenue/visitors) as revenue_per_visitor
from revenue_funnel
