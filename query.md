# Insert data
Tải dữ liệu về dạng .sql và insert vào db fpt:
```bash
mysql -u root -p fpt < "C:\Users\HI.WELCOME TO NET\Downloads\create_mavenfuzzyfactory.sql"
```

# Implement query

## 1. Analyze Website Growth Trend (Sessions & Orders)

```sql
SELECT 
    YEAR(ws.created_at) AS yr,
    QUARTER(ws.created_at) AS qtr,
    COUNT(DISTINCT ws.website_session_id) AS sessions,
    COUNT(DISTINCT o.order_id) AS orders
FROM
    website_sessions ws
        LEFT JOIN
    orders o ON ws.website_session_id = o.website_session_id
GROUP BY yr , qtr
ORDER BY yr , qtr;
```

## 2. Measure Company Performance

```sql
SELECT 
    YEAR(ws.created_at) AS yr,
    QUARTER(ws.created_at) AS qtr,
    ROUND(COUNT(DISTINCT o.order_id) * 1.0 / COUNT(DISTINCT ws.website_session_id),
            4) AS session_to_order_conv_rate,
    ROUND(AVG(o.price_usd), 2) AS revenue_per_order,
    ROUND(SUM(o.price_usd) * 1.0 / COUNT(DISTINCT ws.website_session_id),
            6) AS revenue_per_session
FROM
    website_sessions ws
        LEFT JOIN
    orders o ON ws.website_session_id = o.website_session_id
GROUP BY yr , qtr
ORDER BY yr , qtr;
```
## 3. Analyze Growth Across Different Categories

```sql
SELECT 
    YEAR(o.created_at) AS yr,
    QUARTER(o.created_at) AS qrt,
    SUM(CASE
        WHEN
            ws.utm_source = 'gsearch'
                AND ws.utm_campaign = 'nonbrand'
        THEN
            1
        ELSE 0
    END) AS gsearch_nonbrand_orders,
    SUM(CASE
        WHEN
            ws.utm_source = 'bsearch'
                AND ws.utm_campaign = 'nonbrand'
        THEN
            1
        ELSE 0
    END) AS bsearch_nonbrand_orders,
    SUM(CASE
        WHEN ws.utm_campaign = 'brand' THEN 1
        ELSE 0
    END) AS brand_search_orders,
    SUM(CASE
        WHEN ws.utm_source = 'nonbrand' THEN 1
        ELSE 0
    END) AS organic_type_in_orders,
    SUM(CASE
        WHEN ws.utm_source IS NULL THEN 1
        ELSE 0
    END) AS direct_type_in_orders
FROM
    orders o
        LEFT JOIN
    website_sessions ws ON o.website_session_id = ws.website_session_id
GROUP BY yr , qrt
ORDER BY yr , qrt;
```
## 4. Conversion Rate Analysis by Category

```sql
-- Tỷ lệ chuyển đổi (Conversion Rate) theo năm, quý và nguồn truy cập
SELECT 
    YEAR(ws.created_at) AS yr,
    QUARTER(ws.created_at) AS qrt,
    -- Tỷ lệ chuyển đổi cho Google Search Nonbrand
    ROUND(SUM(CASE WHEN ws.utm_source = 'gsearch' AND ws.utm_campaign = 'nonbrand' AND o.order_id IS NOT NULL THEN 1 ELSE 0 END) * 1.0 /
          COUNT(CASE WHEN ws.utm_source = 'gsearch' AND ws.utm_campaign = 'nonbrand' THEN 1 ELSE NULL END), 4) AS gsearch_nonbrand_conv_rt,
    -- Tỷ lệ chuyển đổi cho Bing Search Nonbrand
    ROUND(SUM(CASE WHEN ws.utm_source = 'bsearch' AND ws.utm_campaign = 'nonbrand' AND o.order_id IS NOT NULL THEN 1 ELSE 0 END) * 1.0 /
          COUNT(CASE WHEN ws.utm_source = 'bsearch' AND ws.utm_campaign = 'nonbrand' THEN 1 ELSE NULL END), 4) AS bsearch_nonbrand_conv_rt,
    -- Tỷ lệ chuyển đổi cho Brand Search
    ROUND(SUM(CASE WHEN ws.utm_campaign = 'brand' AND o.order_id IS NOT NULL THEN 1 ELSE 0 END) * 1.0 /
          COUNT(CASE WHEN ws.utm_campaign = 'brand' THEN 1 ELSE NULL END), 4) AS brand_search_conv_rt,
    -- Tỷ lệ chuyển đổi cho Organic Search
    ROUND(SUM(CASE WHEN ws.utm_source = 'nonbrand' AND o.order_id IS NOT NULL THEN 1 ELSE 0 END) * 1.0 /
          COUNT(CASE WHEN ws.utm_source = 'nonbrand' THEN 1 ELSE NULL END), 4) AS organic_search_conv_rt,
    -- Tỷ lệ chuyển đổi cho Direct Traffic
    ROUND(SUM(CASE WHEN ws.utm_source IS NULL AND o.order_id IS NOT NULL THEN 1 ELSE 0 END) * 1.0 /
          COUNT(CASE WHEN ws.utm_source IS NULL THEN 1 ELSE NULL END), 4) AS direct_type_in_conv_rt
FROM website_sessions ws
LEFT JOIN orders o ON ws.website_session_id = o.website_session_id
GROUP BY yr, qrt
ORDER BY yr, qrt;

```

## 5. Conversion Rate Analysis by Category

```sql
SELECT
    YEAR(o.created_at) AS yr,
    MONTH(o.created_at) AS no,
    SUM(CASE WHEN p.product_name = 'The Original Mr. Fuzzy' THEN oi.price_usd ELSE 0 END) AS mrfuzzy_rev,
    SUM(CASE WHEN p.product_name = 'The Original Mr. Fuzzy' THEN (oi.price_usd - oi.cogs_usd) ELSE 0 END) AS mrfuzzy_marg,
    SUM(CASE WHEN p.product_name = 'The Forever Love Bear' THEN oi.price_usd ELSE 0 END) AS lovebear_rev,
    SUM(CASE WHEN p.product_name = 'The Forever Love Bear' THEN (oi.price_usd - oi.cogs_usd) ELSE 0 END) AS lovebear_marg,
    SUM(CASE WHEN p.product_name = 'The Birthday Sugar Panda' THEN oi.price_usd ELSE 0 END) AS birthdaybear_rev,
    SUM(CASE WHEN p.product_name = 'The Birthday Sugar Panda' THEN (oi.price_usd - oi.cogs_usd) ELSE 0 END) AS birthdaybear_marg,
    SUM(CASE WHEN p.product_name = 'The Hudson River Mini bear' THEN oi.price_usd ELSE 0 END) AS minibear_rev,
    SUM(CASE WHEN p.product_name = 'The Hudson River Mini bear' THEN (oi.price_usd - oi.cogs_usd) ELSE 0 END) AS minibear_marg,
    SUM(oi.price_usd) AS total_revenue,
    SUM(oi.price_usd - oi.cogs_usd) AS total_margin
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
GROUP BY yr, no
ORDER BY yr, no;
```


## 6. Revenue and Profit Analysis


```sql
SELECT 
    YEAR(wp.created_at) AS yr,
    MONTH(wp.created_at) AS no,
    -- Tổng số phiên truy cập đến trang sản phẩm
    COUNT(CASE WHEN wp.pageview_url = '/products' THEN 1 ELSE NULL END) AS sessions_to_product_page,
    -- Tổng số phiên tiếp tục từ trang sản phẩm sang trang khác
    COUNT(CASE WHEN wp.pageview_url = '/products' AND wp.website_pageview_id IN (
        SELECT DISTINCT wp1.website_pageview_id
        FROM website_pageviews wp1
        WHERE wp1.pageview_url != '/products' AND wp1.created_at > wp.created_at
    ) THEN 1 ELSE NULL END) AS click_to_next,
    -- Tỷ lệ chuyển đổi từ trang sản phẩm sang trang tiếp theo
    ROUND(COUNT(CASE WHEN wp.pageview_url = '/products' AND wp.website_pageview_id IN (
        SELECT DISTINCT wp1.website_pageview_id
        FROM website_pageviews wp1
        WHERE wp1.pageview_url != '/products' AND wp1.created_at > wp.created_at
    ) THEN 1 ELSE NULL END) * 1.0 /
          COUNT(CASE WHEN wp.pageview_url = '/products' THEN 1 ELSE NULL END), 4) AS clickthrough_rt,
    -- Tổng số đơn hàng
    COUNT(DISTINCT o.order_id) AS orders,
    -- Tỷ lệ chuyển đổi từ trang sản phẩm sang đơn hàng
    ROUND(COUNT(DISTINCT o.order_id) * 1.0 /
          COUNT(CASE WHEN wp.pageview_url = '/products' THEN 1 ELSE NULL END), 4) AS products_to_order_rt
FROM website_pageviews wp
LEFT JOIN orders o ON wp.website_session_id = o.website_session_id
GROUP BY yr, no
ORDER BY yr, no;
```


