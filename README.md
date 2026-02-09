# Toy-Store-MySQL-Data-Analysis

Project Overview
This project focuses on analyzing the eCommerce database of Kraven Toy Store, an online retailer that recently launched its first product.
As an eCommerce Database Analyst, the goal is to collaborate with the CEO, Head of Marketing, and Website Manager to optimize marketing channels,
measure website performance, and assess the impact of product launches.

# Entity Relationship Diagram(ERD)
![ERD](https://github.com/user-attachments/assets/795f72a8-2116-4473-b8f3-8127d7c1e0f4)

# Objectives
1) Traffic Source Analysis: Evaluate and optimize marketing channels by analyzing traffic sources using UTM parameters to identify which channels are most effective in driving website sessions.
2) Conversion Rate Optimization: Measure and enhance conversion rates, particularly for "gsearch" traffic, to ensure they meet target thresholds, thereby informing bidding strategies.
3) Device Performance Assessment: Compare conversion rates and user engagement across different devices (desktop vs. mobile) to tailor marketing efforts and optimize bids based on performance.
4) User Engagement and Website Performance Monitoring: Analyze user engagement metrics such as bounce rates and session volumes on key landing pages to improve website design and content, ultimately enhancing user experience and conversion rates.

## Business Problems and Solutions
# Analyzing Website Traffic Sources and Optimizing the Bids

# 1. Site traffic breakdown
Objective: Cindy Sharp (CEO) requested a breakdown of traffic sources by UTM source, campaign, and referring domain.
Query:
SELECT
 utm_source,
  utm_campaign,
  http_referer,
  COUNT(website_sessions) AS sessions
FROM
  website_sessions 
WHERE
  created_at < '2012-04-12'
GROUP BY
  utm_source,
  utm_campaign,
  http_referer
ORDER BY sessions DESC;


Query Result:
<img width="486" height="212" alt="results_query1" src="https://github.com/user-attachments/assets/3108c261-741c-4786-a269-6d1fe79b86c9" />




