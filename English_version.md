# 1. Chart Types

|Chart Type|Definition|Selection Scenario|Example|
|:--|:--|:--|:--|
|**Histogram**|Displays the distribution of **continuous data** by dividing the data into intervals (bins) and counting frequencies.|Used to observe the **distribution of continuous data**, such as central tendency and dispersion.|Employee salary distribution, patient waiting times, weight of Chicago residents, social media usage time among young people.|
|**Bar Chart**|Displays the count or frequency of **discrete data**, typically used for comparing **size differences between different categories**.|**Suitable for categorical data**, such as gender, region, product types, etc.|**Employee satisfaction**: How many people fall into each satisfaction category (e.g., very satisfied, satisfied, neutral, dissatisfied).|
|**Box Plot**|Displays the distribution of data, including quartiles, median, and outliers.|Compare **distribution differences between groups** or identify **outliers**.|Satisfaction score analysis. (Check the range from Q1 to Q3)|
|**Line Chart**|Uses lines to connect data points, showing how values change with a certain variable (usually time).|Mainly used for showing **trends in time series data**.|Budget changes categorized by type, spending data over multiple years.|
|**Scatter Plot**|Displays the relationship between two **continuous variables** using points.|Study the **correlation between two continuous variables**.|Relationship between advertising spend and sales revenue, bill amount and tips (with light blue confidence bands), sleep hours and mood, new subscribers per 100k spend increase.|
|**Heatmap**|Uses color intensity to represent the magnitude of values, usually displayed in a two-dimensional matrix.|Used to observe the **value distribution density or intensity under the combination of two categorical variables**.|Mouse clicks and movements on a bank homepage, emergency room wait times segmented by hour and day of the week.|
|**Pivot Table**|Classifies and aggregates data based on one or more dimensions, displayed in a table format.|Aggregates and compares **numeric data across multiple categorical variables**.|Comparison of average salaries for different job titles, aggregation of numeric data across two categorical dimensions.|

---

**Quick Summary for Memory:**

- **Distribution** → Histogram, Box Plot
    
- **Trend over Time** → Line Chart
    
- **Relationship between Two Continuous Variables** → Scatter Plot (with regression line if necessary)
    
- **Density/Frequency of Two Categorical Combinations** → Heatmap
    
- **Aggregation of Values by Categories** → Pivot Table
    

# 2. Key SQL Functions and Syntax Summary

### 2.1 Calculating Mean and Standard Deviation

- **Mean** → Use the `AVG()` function
    
    `SELECT AVG(score) AS mean_score FROM table_name;`
    
- **Standard Deviation** → Use the `STDDEV()` function
    
    `SELECT STDDEV(score) AS std_dev_score FROM table_name;`
    

### 2.2 Calculating Percentiles (Quartiles)

- **25% and 50% Percentiles (Median)** → Use `PERCENTILE_CONT()`
    
    ```
    SELECT    
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY Price) AS percentile_25, 
    PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY Price) AS percentile_50 
    FROM products;
    ```
    

### 2.3 Calculating Price Range (Max - Min)

- **Price Range** `MAX(Price) - MIN(Price)`
    

### 2.4 Calculating Standard Deviation for Volume

- **Overall Volume Standard Deviation** `STDDEV(volume)`
    
- **Group by Year to Calculate Volume Standard Deviation** `STDDEV(volume) AS stddev_volume FROM sales GROUP BY year;`
    

### 2.5 Finding the Most Common Shipping Cost

- **If `MODE()` is supported**
    
    `SELECT MODE() WITHIN GROUP (ORDER BY shipping_cost) AS most_common_shipping_cost FROM orders;`
    
- **If `MODE()` is not supported, use GROUP BY + ORDER BY**
    
    ```
      SELECT shipping_cost, COUNT(*) AS frequency 
      FROM orders 
      GROUP BY shipping_cost 
      ORDER BY frequency DESC LIMIT 1;
    ```
    

---

# 3. Experimental Design

## 1. Variable Types

**Confounding Variable**

- Definition: A variable that influences both the independent and dependent variables, which can obscure the true relationship.
    
- Example: When studying the effect of exercise on weight loss, eating habits could be a confounding variable.
    

**Exposure**

- Definition: The factor to which the subjects are exposed.
    
- Example: Smoking habits.
    

**Response**

- Definition: The outcome or change being measured.
    
- Example: Whether someone develops lung disease.
    

**Control**

- Definition: The conditions that are kept unchanged in an experiment to eliminate other influences.
    
- Example: Keeping sleep duration the same for all participants.
    

---

## 2. Common Distributions

### 2.1 Common Distributions

**Geometric Distribution**

- Definition: The number of failures before the first success.
    
- Example: Tossing a coin and counting how many times you get tails before you get heads.
    

**Binomial Distribution**

- Definition: The number of successes in a fixed number of trials, with each trial having a fixed probability of success.
    
- Example: Tossing a coin 10 times and counting how many heads you get.
    

**Exponential Distribution**

- Definition: The time until the occurrence of an event in continuous time.
    
- Example: Waiting for the next bus.
    

**Poisson Distribution**

- Definition: The number of events that occur in a fixed time or space, with events happening independently at a constant rate.
    
- Example: The number of customer complaints received daily.
    

### 2.2 Non-Normal Distribution Tests for Independence

- Definition: When data does not follow a normal distribution, traditional tests like the T-test cannot be used. Non-parametric tests are used instead.
    
- Correct Method: **Chi-square test for independence**.
    
- Example: Investigating if gender is related to whether people like watching basketball.
    

### 2.2 Summary of Distributions

|Distribution Type|Definition|Example|Mnemonic|
|---|---|---|---|
|Poisson|Number of events in a fixed time period|Number of emails received per hour|Poisson counts events|
|Exponential|Time until an event occurs|Time until the next bus arrives|Exponential waiting time|
|Geometric|Number of failures before the first success|Number of tails before the first head in coin tosses|Geometric failures|
|Binomial|Number of successes in a fixed number of trials|Number of heads in 10 coin tosses|Binomial success count|
|Normal|Continuous bell-shaped distribution|Distribution of student scores|Normal bell-shaped distribution|
|T Distribution|Used when sample size is small and population variance is unknown|Comparing popularity between two small groups (e.g., two tea shops with small samples)|Comparing two groups|
|F Distribution|Used to compare variances of more than two samples|Comparing the effect of three different study methods on student performance|Comparing multiple groups|
|Chi-square test for independence|Tests for independence between categorical variables|Investigating whether gender is related to liking basketball|Chi-square tests relationships|

---

## 3. Research Design Types

|Type|Definition|Example|
|---|---|---|
|**Experimental**|Manipulate **independent variable** to observe its effect on the **dependent variable**.|One group drinks coffee, the other drinks water, and then we measure who runs faster.|
|**Correlational**|Observe whether variables are **correlated**, but do not manipulate them.|Examining the correlation between sleep time and academic performance.|
|**Descriptive**|Simply **describes phenomena** without exploring causal relationships.|Surveying how many hours students use their phones each day.|
|**Explanatory**|Further explores **why** a phenomenon occurs and identifies mechanisms or causes.|Studying why people who exercise regularly have lower stress levels.|

---

## 4. Null Hypothesis and Alternative Hypothesis

**Null Hypothesis (H₀)**

- Definition: The hypothesis that assumes "no change, no effect, no relationship" to start with.
    
- Use: See if the **null hypothesis can be rejected**. If data shows "change," we **reject the null hypothesis**. If the data shows "no significant change," we **fail to reject the null hypothesis**.
    

**Alternative Hypothesis (H₁)**

- Definition: Assumes there is a difference or effect.
    
- Example: A new drug improves the cure rate significantly.
    

**Null Hypothesis & Alternative Hypothesis**

- They are opposites.
    
- If the null hypothesis is rejected, there is **sufficient evidence** to support the alternative hypothesis.
    
- Null Hypothesis H₀: The effect of the new drug is the same as the old drug. Alternative Hypothesis H₁: The new drug is more effective. If the p-value is <0.05, the null hypothesis is rejected, and there is **sufficient evidence** to support the new drug being more effective.
    

---

## 5. Hypothesis Testing

### 5.1 Pre-Test Designs

|Design Type|Characteristics|Example|Purpose|
|---|---|---|---|
|Random Two-Group Pretest-Posttest|Pre-test and post-test to compare changes|Measure scores before and after teaching with different methods|To detect changes after treatment.|
|Random Two-Group Posttest Only|Only post-test, avoiding pre-test effects|Compare scores of two groups after treatment|Avoid pre-test effects, but can't confirm baseline similarity.|
|Solomon Four Group|Combines pre-test and no pre-test groups to fully test interaction effects|Some students take pre-test, some don't, then all are treated similarly|Best for testing interaction effects.|
|Factorial|Tests two or more variables and their interactions|Investigating the combined effect of teaching methods and study time on scores|Analyzes the effects of multiple factors and their interaction.|

Example:

- You want to test if a new math tutoring course improves student performance.
    
- However, you're concerned that giving students a pre-test might make them overly aware of the exam content, affecting the results.
    
- **Solomon four-group design** helps distinguish:
    
    - Whether the tutoring is effective.
        
    - Whether the pre-test itself affected performance.
        

### 5.2 Statistical Significance of Estimation Points

- Measures whether an estimated value in a sample is representative (i.e., whether
