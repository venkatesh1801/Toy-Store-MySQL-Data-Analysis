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
```sql
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
ORDER BY
    sessions DESC;
```
<img width="486" height="212" alt="results_query1" src="https://github.com/user-attachments/assets/3108c261-741c-4786-a269-6d1fe79b86c9" />

# 2. Gsearch Conversion Rate Analysis
Objective: Tom Parmesan (Marketing Director) wanted to evaluate the conversion rate of "gsearch" traffic, expecting it to be at least 4%.
Query:
```sql
SELECT
   COUNT(DISTINCT ws.website_session_id) AS sessions,
   COUNT(o.order_id) AS orders,
   ROUND(
       COUNT(o.order_id) * 100.0 / COUNT(DISTINCT ws.website_session_id), 
       2
   ) AS sessions_to_order_conversion_rate
FROM
   website_sessions ws
LEFT JOIN
   orders o
ON 
   o.website_session_id = ws.website_session_id
WHERE
   ws.created_at < '2012-04-14'
```


<img width="364" height="50" alt="results_query2" src="https://github.com/user-attachments/assets/aecaf395-6403-4d7d-b205-9890dcf8bd51" />
# Findings: 
The conversion rate was found to be 2.88%, below the target threshold of 4%, prompting a decision to reduce bids on this source.

#  Gsearch volume trends
Objective: Following the bid reduction on "gsearch nonbrand," the team needed to assess if this change affected session counts.
```sql
SELECT
   MIN(DATE(created_at)) AS week_start_date,
   COUNT(DISTINCT website_session_id) AS sessions
FROM
   website_sessions
WHERE
   created_at < '2012-05-12'
   AND utm_campaign = 'nonbrand'
   AND utm_source = 'gsearch'
GROUP BY
   WEEK(created_at);
```

<img width="205" height="200" alt="image" src="https://github.com/user-attachments/assets/e286e6e9-7af9-4e96-a5f1-fd82de226de1" />
Findings: The analysis confirmed that "gsearch nonbrand" is sensitive to bid changes.

# 4. Gsearch device level performance
Objective: Tom sought insights into conversion rates segmented by device type to optimize bidding strategies
```sql
 SELECT
   ws.device_type,
   COUNT(DISTINCT ws.website_session_id) AS sessions,
   COUNT(o.order_id) AS orders,
   ROUND(
       COUNT(o.order_id) * 100.0 / COUNT(DISTINCT ws.website_session_id),
       2
   ) AS sessions_to_order_conversion_rate
FROM
   website_sessions ws
LEFT JOIN
   orders o
ON 
   o.website_session_id = ws.website_session_id
WHERE
   ws.created_at < '2012-04-14'
   AND ws.utm_source = 'gsearch'
   AND ws.utm_campaign = 'nonbrand'
GROUP BY
   ws.device_type;

```
<img width="472" height="70" alt="image" src="https://github.com/user-attachments/assets/620066bb-47fa-4f47-8228-4794128975bf" />
Findings: Desktop users exhibited a higher conversion rate of 4.14% compared to mobile users at 0.92%.

# 5. Gsearch device level trends
Objective: To monitor the performance of desktop versus mobile sessions over several weeks.
```sql
SELECT
   MIN(DATE(created_at)) AS week_start_date,
   COUNT(DISTINCT CASE WHEN device_type = 'desktop' THEN website_session_id ELSE NULL END) AS desktop_sessions,
   COUNT(DISTINCT CASE WHEN device_type = 'mobile' THEN website_session_id ELSE NULL END) AS mobile_sessions
FROM
   website_sessions
WHERE
   created_at BETWEEN '2012-04-15' AND '2012-06-09'
   AND utm_campaign = 'nonbrand'
   AND utm_source = 'gsearch'
GROUP BY
   WEEK(created_at);
```
<img width="336" height="204" alt="image" src="https://github.com/user-attachments/assets/0a4a3927-70a1-4adb-855c-8c524a678dd4" />

Findings: Desktop sessions consistently outperformed mobile sessions.
## Analyzing Website Traffic Sources and Optimizing the Bids
# 6. Top Website Pages by Session Volume
Objective
```sql
SELECT
   MIN(DATE(created_at)) AS week_start_date,
   COUNT(DISTINCT CASE WHEN device_type = 'desktop' THEN website_session_id ELSE NULL END) AS desktop_sessions,
   COUNT(DISTINCT CASE WHEN device_type = 'mobile' THEN website_session_id ELSE NULL END) AS mobile_sessions
FROM
   website_sessions
WHERE
   created_at BETWEEN '2012-04-15' AND '2012-06-09'
   AND utm_campaign = 'nonbrand'
   AND utm_source = 'gsearch'
GROUP BY
   WEEK(created_at);
```
<img width="380" height="182" alt="image" src="https://github.com/user-attachments/assets/0754cc4b-f641-4ac2-a219-be4bfba23013" />

Findings: Looks like the homepage, the products page, and the Mr. Fuzzy page get the bulk of our traffic.

# 7. Top Entry Pages
Objective: Morgan Rockwell (Website Manager) wants us to pull all entry pages and rank them on entry volume.
```sql
-- Create a temporary table to find the first pageview per session
CREATE TEMPORARY TABLE first_pv_per_session AS
SELECT 
   website_session_id,
   MIN(website_pageview_id) AS landing_pageview_id
FROM
   website_pageviews
WHERE
   created_at < '2012-06-12'
GROUP BY
   website_session_id;

-- Query to count sessions hitting each landing page
SELECT 
   wp.pageview_url AS landing_page,
   COUNT(fps.landing_pageview_id) AS sessions_hitting_landing_page
FROM
   first_pv_per_session fps
LEFT JOIN
   website_pageviews wp
ON 
   wp.website_pageview_id = fps.landing_pageview_id
GROUP BY
   wp.pageview_url;

```
<img width="300" height="52" alt="image" src="https://github.com/user-attachments/assets/b2bb2c6b-5315-4649-9e7a-3e3ffdf3bb14" />

Findings: Looks like our traffic all comes in through the homepage right now!

# 8. Bounce Rate Analysis
Objective: Morgan wanted to measure how many users left after viewing only one page.
```sql
-- Step 1: Finding the minimum website pageview ID for each session that we care about
CREATE TEMPORARY TABLE first_pageviews AS
SELECT 
   website_session_id,
   MIN(website_pageview_id) AS landing_pageview_id
FROM
   website_pageviews
WHERE
   created_at < '2012-06-14'
GROUP BY
   website_session_id;

-- Step 2: Finding the landing page for each website session ID
CREATE TEMPORARY TABLE session_with_landing_page_demo AS
SELECT 
   fp.website_session_id,
   wp.pageview_url AS landing_page
FROM 
   first_pageviews fp
LEFT JOIN 
   website_pageviews wp
ON 
   wp.website_pageview_id = fp.landing_pageview_id;

-- Step 3: Finding the bounced sessions
CREATE TEMPORARY TABLE bounced_sessions_only AS
SELECT 
   swlp.website_session_id,
   swlp.landing_page,
   COUNT(wp.website_pageview_id) AS nos_of_sessions_per_website_session_id
FROM 
   session_with_landing_page_demo swlp
LEFT JOIN 
   website_pageviews wp
ON 
   wp.website_session_id = swlp.website_session_id
WHERE 
   swlp.landing_page = '/home'
GROUP BY
   swlp.website_session_id,
   swlp.landing_page
HAVING 
   COUNT(wp.website_pageview_id) = 1;

-- Step 4: Calculating the bounced session rates
SELECT
   COUNT(swlp.website_session_id) AS sessions,
   COUNT(bs.website_session_id) AS bounced_sessions,
   ROUND(
       COUNT(bs.website_session_id) * 100.0 / COUNT(swlp.website_session_id), 
       2
   ) AS bounced_rate
FROM
   session_with_landing_page_demo swlp
LEFT JOIN 
   bounced_sessions_only bs
ON 
   bs.website_session_id = swlp.website_session_id;

```
<img width="310" height="49" alt="image" src="https://github.com/user-attachments/assets/035cfcd2-7cf8-4726-a0e2-4dc8dd1e9b29" />

Findings: The overall bounce rate was found to be approximately 59.18%, raising concerns about user engagement.

# 9. Help Analyzing Lander Page Test
Background: Based on the bounce rate analysis, Morgan ran a new custom landing page (/lander 1) in a 50/50 test against the homepage (/home) for our gsearch nonbrand traffic. Morgan Wants to know the bounce rates for the two groups.

```sql
-- Step 0: Finding the day new homepage '/lander-1' was introduced
SELECT 
    MIN(created_at) AS first_created_at,
    MIN(website_pageview_id) AS first_pageview_id
FROM 
    website_pageviews
WHERE 
    pageview_url = '/lander-1';

-- Step 1: Find the first pageview id for sessions with specific conditions
CREATE TEMPORARY TABLE first_test_pageviews AS
SELECT 
    wp.website_session_id,
    MIN(wp.website_pageview_id) AS min_pageview_id
FROM
    website_pageviews wp
INNER JOIN 
    website_sessions ws
    ON ws.website_session_id = wp.website_session_id
WHERE 
    ws.created_at < '2012-07-28'
    AND wp.website_pageview_id > 23504
    AND ws.utm_source = 'gsearch'
    AND ws.utm_campaign = 'nonbrand'
GROUP BY 
    wp.website_session_id;

-- Step 2: Finding the landing page for each session ID
CREATE TEMPORARY TABLE nonbrand_test_sessions_W_landing_page AS
SELECT 
    ftp.website_session_id,
    wp.pageview_url AS landing_page,
    COUNT(wp.website_pageview_id) AS pageview_count
FROM 
    first_test_pageviews ftp
LEFT JOIN 
    website_pageviews wp
    ON wp.website_pageview_id = ftp.min_pageview_id
WHERE 
    wp.pageview_url IN ('/home', '/lander-1')
GROUP BY 
    ftp.website_session_id, wp.pageview_url;

-- Step 3: Finding the bounced session IDs
CREATE TEMPORARY TABLE nonbrand_bounced_sessions_only AS
SELECT
    nt.landing_page,
    nt.website_session_id,
    COUNT(wp.website_pageview_id) AS pageview_count
FROM 
    nonbrand_test_sessions_W_landing_page nt
LEFT JOIN 
    website_pageviews wp
    ON wp.website_session_id = nt.website_session_id
GROUP BY 
    nt.website_session_id, nt.landing_page
HAVING 
    COUNT(wp.website_pageview_id) = 1;

-- Step 4: Calculating the bounce rate
SELECT
    nt.landing_page,
    COUNT(DISTINCT nt.website_session_id) AS sessions,
    COUNT(DISTINCT bss.website_session_id) AS bounced_sessions,
    ROUND(
        COUNT(DISTINCT bss.website_session_id) * 100.0 / COUNT(DISTINCT nt.website_session_id), 
        2
    ) AS bounce_rate
FROM 
    nonbrand_test_sessions_W_landing_page nt
LEFT JOIN 
    nonbrand_bounced_sessions_only bss
    ON bss.website_session_id = nt.website_session_id
GROUP BY 
    nt.landing_page;

```
<img width="411" height="77" alt="image" src="https://github.com/user-attachments/assets/ada738d5-1b8d-457e-b338-31f39854d7e2" />
Findings: It looks like the custom lander has a lower bounce rate with 53,22%, success!

# 10. Landing Page Trend Analysis
Background: Based on the landing page test, Morgan routed all the traffic to the new lander page and wants to confirm all the traffic rate is routed to the new lander page. She wants us to pull our overall paid search bounce rate trended weekly starting from june 1st 2012.
```sql
-- Step 1: Finding the first website_pageview_id for each website_session_id
CREATE TEMPORARY TABLE sessions_w_min_pageview_id_and_view_count AS
SELECT
    wp.website_session_id,
    MIN(wp.website_pageview_id) AS min_pageview_id,
    COUNT(wp.website_pageview_id) AS count_pageviews
FROM
    website_pageviews wp
LEFT JOIN
    website_sessions ws
    ON ws.website_session_id = wp.website_session_id
WHERE 
    ws.created_at BETWEEN '2012-06-01' AND '2012-08-31'
    AND ws.utm_source = 'gsearch'
    AND ws.utm_campaign = 'nonbrand'
GROUP BY 
    wp.website_session_id;

-- Step 2: Identifying the landing page for each session id
CREATE TEMPORARY TABLE sessions_w_count_lander_and_created_at AS
SELECT
    swmp.website_session_id,
    swmp.min_pageview_id,
    swmp.count_pageviews,
    wp.pageview_url AS landing_page,
    wp.created_at
FROM
    sessions_w_min_pageview_id_and_view_count swmp
LEFT JOIN
    website_pageviews wp
    ON wp.website_session_id = swmp.website_session_id
WHERE 
    wp.pageview_url IN ('/home', '/lander-1');

-- Step 3: Calculating the bounce rates and total sessions for each homepage between '2012-06-01' and '2012-08-31'
SELECT
    MIN(DATE(wp.created_at)) AS week_start_date,
    ROUND(COUNT(DISTINCT CASE WHEN swmp.count_pageviews = 1 THEN swmp.website_session_id ELSE NULL END) * 1.0 /
          COUNT(DISTINCT swmp.website_session_id), 2) AS bounce_rate,
    COUNT(DISTINCT CASE WHEN swmp.landing_page = '/home' THEN swmp.website_session_id ELSE NULL END) AS home_sessions,
    COUNT(DISTINCT CASE WHEN swmp.landing_page = '/lander-1' THEN swmp.website_session_id ELSE NULL END) AS lander_sessions
FROM
    sessions_w_count_lander_and_created_at swmp
GROUP BY
    WEEK(wp.created_at);

```
<img width="461" height="335" alt="image" src="https://github.com/user-attachments/assets/45db8407-0e3a-4d35-adc2-1ca8dfb13f4c" />
Findings: The sessions have been completed to the new lander page(/lander) and overall bounce rate has also decreased.

# 11. Click Rates Across Key Pages
Background: Morgan wants to Understand click-through rates for various pages was essential for optimizing user navigation.
```sql
-- Step 1: Select all the pageviews with relevant sessions
CREATE TEMPORARY TABLE session_level_made_it_flag AS
SELECT
      ws.website_session_id,
      MAX(CASE WHEN wp.pageview_url = '/products' THEN 1 ELSE 0 END) AS products_made_it,
      MAX(CASE WHEN wp.pageview_url = '/the-original-mr-fuzzy' THEN 1 ELSE 0 END) AS mrfuzzy_made_it,
      MAX(CASE WHEN wp.pageview_url = '/cart' THEN 1 ELSE 0 END) AS cart_made_it,      
      MAX(CASE WHEN wp.pageview_url = '/shipping' THEN 1 ELSE 0 END) AS shipping_made_it,
      MAX(CASE WHEN wp.pageview_url = '/billing' THEN 1 ELSE 0 END) AS billing_made_it,
      MAX(CASE WHEN wp.pageview_url = '/thank-you-for-your-order' THEN 1 ELSE 0 END) AS thankyou_made_it
FROM
    website_sessions ws
LEFT JOIN
    website_pageviews wp
    ON ws.website_session_id = wp.website_session_id
WHERE 
    ws.utm_source = 'gsearch'
    AND ws.utm_campaign = 'nonbrand'
    AND ws.created_at BETWEEN '2012-08-05' AND '2012-09-05'
GROUP BY 
    ws.website_session_id;

-- Step 2: Calculating total visits to all the pages
CREATE TEMPORARY TABLE total_sessions_for_each_page2 AS
SELECT 
    COUNT(website_session_id) AS total_session,
    SUM(products_made_it) AS to_products,
    SUM(mrfuzzy_made_it) AS to_mrfuzzy,
    SUM(cart_made_it) AS to_cart,
    SUM(shipping_made_it) AS to_shipping,
    SUM(billing_made_it) AS to_billing,
    SUM(thankyou_made_it) AS to_thankyou
FROM 
    session_level_made_it_flag;

-- Step 3: Finding the conversion rates for each page
SELECT 
    ROUND(to_products / total_session * 100, 2) AS lander_click_rate,
    ROUND(to_mrfuzzy / to_products * 100, 2) AS products_click_rate,
    ROUND(to_cart / to_mrfuzzy * 100, 2) AS mrfuzzy_click_rate,
    ROUND(to_shipping / to_cart * 100, 2) AS cart_click_rate,
    ROUND(to_billing / to_shipping * 100, 2) AS shipping_click_rate,
    ROUND(to_thankyou / to_billing * 100, 2) AS billing_click_rate
FROM total_sessions_for_each_page2;

```
<img width="766" height="45" alt="image" src="https://github.com/user-attachments/assets/0e1fd9c4-d8b9-46e8-80df-4b06aa9ec6d8" />
Findings: Looks like we should focus on the lander, Mr. Fuzzy page , and the billing page , which have the lowest click rates.

# 12. Conversion Rates for Billing Pages
Background: Based on conversion funnel analysis, Morgan tested an updated billing page(/billing -2). She wants a comparison between the old vs new billing page.
```sql
-- Step 1: Finding the date when the new billing page (/billing-2) was introduced
SELECT 
    MIN(created_at) AS first_billing_2_introduction
FROM 
    website_pageviews
WHERE 
    pageview_url = '/billing-2';

-- Step 2: Finding the sessions having orders
CREATE TEMPORARY TABLE billing_sessions_with_orders AS
SELECT
    wp.website_session_id,
    wp.pageview_url,
    o.order_id
FROM
    website_pageviews wp
LEFT JOIN
    orders o
    ON o.website_session_id = wp.website_session_id
WHERE
    wp.pageview_url IN ('/billing', '/billing-2')
    AND wp.website_session_id >= 25325
    AND wp.created_at < '2012-11-10';

-- Step 3: Finding the conversion rate
SELECT
    pageview_url,
    COUNT(DISTINCT order_id) AS orders_count,    -- Counting distinct orders
    COUNT(DISTINCT website_session_id) AS sessions_count,  -- Counting distinct sessions
    ROUND(COUNT(DISTINCT order_id) / COUNT(DISTINCT website_session_id) * 100, 2) AS conversion_rate
FROM 
    billing_sessions_with_orders
GROUP BY 
    pageview_url;

```
<img width="544" height="78" alt="image" src="https://github.com/user-attachments/assets/1f3666ac-76d7-4809-9106-5885384639aa" />
Findings: The version of the billing page(/billing-2) has almost 63% conversion rate which is significantlly greater than previous billing page.

## Conclusion
These findings demonstrate the value of data-driven decision-making in improving marketing efficiency, website optimization, and overall business performance. By leveraging detailed metrics, A/B testing, and segment analysis, this project achieved meaningful improvements across traffic sources, user engagement, and conversion rates. The actionable insights provided form a strong foundation for continued growth and optimization, enabling better customer experiences and stronger financial outcomes.












