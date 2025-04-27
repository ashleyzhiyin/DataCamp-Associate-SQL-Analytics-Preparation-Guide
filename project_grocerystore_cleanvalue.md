- 检查missing
- 数据清洗（替换missing/统一类型/中位数/大小写）

# 一、数据清洗案例
## 1. 先快速查看原始表

```sql
SELECT * FROM products LIMIT 10;
```

发现问题：

- 有些字段是 `'-'`，表示缺失。
    
- `weight` 单位不统一，有些是 `'500 grams'`，有些直接数字。
    
- `stock_location` 大小写不统一，有的是 `'beijing'`，有的是 `'Beijing'`，有的是 `'BEIJING'`。
    

---

##  2. 类型处理

- nominal（分类变量，比如 `product_type`, `brand`, `stock_location`） → 都用 `TEXT`。
    
- continuous（数值变量，比如 `weight`, `price`, `average_units_sold`） → 转成 `NUMERIC`。
    

---

## 3. 清洗规则

- 如果是 `'-'` → 用 `REPLACE(column,'-','Unknown')` 变成 `'Unknown'`。
    
- 如果是 `weight` 带 `'grams'`，用 `REPLACE(weight,'grams','')` 去掉，转成数字。
    
- 大小写统一 → 用 `UPPER()` 或 `LOWER()`。
    
- 如果 `weight` 或 `price` 为空，要用 **中位数**填补，而不是随便填0。
    

---

##  4. 求中位数 (Median)

- 用 `PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY column)` 求中位数！
    

例子：

```sql
SELECT 
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY (REPLACE(weight, 'grams', '')::numeric)) AS median_weight,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY price) AS median_price
FROM products;
```

---

##  5. 最后形成干净版表（clean_data）

清理+填补之后，做成一个CTE（common table expression, 不改动原表格）：

```sql
WITH 
median_value AS (
    SELECT 
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY (REPLACE(weight, 'grams', '')::numeric)) AS median_weight,
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY price) AS median_price
    FROM products
),
clean_data AS (
    SELECT 
        product_id,
        COALESCE(REPLACE(product_type, '-', 'Unknown')::text, 'Unknown') AS product_type,
        COALESCE(REPLACE(brand, '-', 'Unknown')::text, 'Unknown') AS brand,
        ROUND(
            COALESCE(REPLACE(weight, 'grams', '')::numeric, median_value.median_weight), 
            2
        ) AS weight,
        ROUND(
            COALESCE(price, median_value.median_price), 
            2
        ) AS price,
        COALESCE(average_units_sold, 0) AS average_units_sold,
        EXTRACT(YEAR FROM TO_DATE(CONCAT(COALESCE(year_added, 2022)::text, '-01-01'),'YYYY-MM-DD')) AS year_added,
        UPPER(COALESCE(stock_location, 'Unknown')::text) AS stock_location
    FROM 
        products, median_value
)
SELECT * FROM clean_data;
```


## 总结

|步骤|做什么|
|:--|:--|
|查看原表|`SELECT * FROM products LIMIT 10;`|
|nominal转text|`REPLACE('-', 'Unknown')::text`|
|continuous转numeric|`REPLACE('grams','')::numeric`|
|发现问题|有`'-'`、单位问题、大写小写问题|
|先求median|`PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY column)`|
|清洗填补|`COALESCE(字段, median值)`|
|大小写统一|`UPPER()`|



---

#  二、SQL 通用清洗模板

```sql
-- 通用数据清洗模板
-- 适用场景：原表有缺失值（'-'、NULL）、单位不统一、大写小写混乱等问题
-- 原则：不修改原表，生成一个临时干净版表 clean_data

-- 第一步：建立CTE，求连续变量的中位数
WITH 
median_value AS (
    SELECT 
        -- 求 weight 中位数，去掉单位 'grams' 后转为数值
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY (REPLACE(weight_column, 'grams', '')::numeric)) AS median_weight,
        
        -- 求 price 中位数
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY price_column) AS median_price
    FROM raw_table_name
),

-- 第二步：建立清洗版临时表（Common Table Expression）
clean_data AS (
    SELECT 
        -- 主键直接保留
        primary_key_column,
        
        -- nominal字段：统一处理缺失值、去掉'-'、大写统一
        UPPER(TRIM(COALESCE(REPLACE(nominal_column1, '-', 'Unknown')::text, 'Unknown'))) AS nominal_column1_clean,
        UPPER(TRIM(COALESCE(REPLACE(nominal_column2, '-', 'Unknown')::text, 'Unknown'))) AS nominal_column2_clean,
        
        -- continuous字段：去单位，转数值，缺失时用中位数填补
        ROUND(
            COALESCE(REPLACE(weight_column, 'grams', '')::numeric, median_value.median_weight),
            2
        ) AS weight_clean,
        ROUND(
            COALESCE(price_column, median_value.median_price),
            2
        ) AS price_clean,
        
        -- 其他数值字段：比如销量，缺失时补0
        COALESCE(continuous_column1, 0) AS continuous_column1_clean,
        
        -- 时间字段：比如 year_added，缺失时补2022，并统一转成年份
        EXTRACT(YEAR FROM TO_DATE(CONCAT(COALESCE(year_column, 2022)::text, '-01-01'), 'YYYY-MM-DD')) AS year_clean,
        
        -- stock_location：大写统一
        UPPER(COALESCE(location_column, 'Unknown')::text) AS location_clean

    FROM 
        raw_table_name, median_value  -- 注意这里把中位数表一起引入
)

-- 第三步：从清洗表 clean_data 里提取最终需要的数据
SELECT *
FROM clean_data;
```

---

## 使用方法

按照下面的方法转换字段：

|需要替换|示例|
|:--|:--|
|raw_table_name|products|
|primary_key_column|product_id|
|nominal_column1|product_type|
|nominal_column2|brand|
|weight_column|weight|
|price_column|price|
|continuous_column1|average_units_sold|
|year_column|year_added|
|location_column|stock_location|

## 额外备注：

CTE的作用就是：
- **Common Table Expression**：公共临时表达式
- **不会修改原始表（products）**，只是内存里临时生成一个表
- **好处**：方便逻辑拆分，代码更清晰，易于调试
    
