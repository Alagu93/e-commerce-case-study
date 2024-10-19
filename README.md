
# E-Commerce Case Study

## Project Description
This project involves data analysis for an e-commerce company to extract insights from various datasets to drive business strategies.

## Technologies Used
- MySQL
- SQL queries

## Datasets
- **Customers Dataset**: `customer_id`, `name`, `location`
- **Products Dataset**: `product_id`, `name`, `category`, `price`
- **Orders Dataset**: `order_id`, `order_date`, `customer_id`, `total_amount`
- **OrderDetails Dataset**: `order_id`, `product_id`, `quantity`, `price_per_unit`

## Queries

### 1. Market Segment Analysis
Identify the top 3 cities with the highest number of customers to determine key markets for targeted marketing and logistic optimization.
```sql
SELECT 
    cu.location AS location, 
    COUNT(DISTINCT cu.customer_id) AS number_of_customers
FROM 
    Customers cu
GROUP BY 
    cu.location
ORDER BY 
    number_of_customers DESC
LIMIT 3;
```

### 2. Engagement Depth Analysis
Determine the distribution of customers by the number of orders placed. This insight will help in segmenting customers into one-time buyers, occasional shoppers, and regular customers for tailored marketing strategies.
```sql
WITH CustomerOrderCount AS (
    SELECT 
        Customer_ID, 
        COUNT(Order_ID) AS NumberOfOrders
    FROM 
        Orders
    GROUP BY 
        Customer_ID
)
SELECT 
    NumberOfOrders, 
    COUNT(Customer_ID) AS CustomerCount
FROM 
    CustomerOrderCount
GROUP BY 
    NumberOfOrders
ORDER BY 
    NumberOfOrders ASC;
```

### 3. Purchase High Value Product
Identify products where the average purchase quantity per order is 2 but with a high total revenue, suggesting premium product trends.
```sql
SELECT 
    Product_Id, 
    AVG(Quantity) AS AvgQuantity,
    SUM(Quantity * price_per_unit) AS TotalRevenue
FROM 
    OrderDetails
GROUP BY 
    Product_Id
HAVING 
    AVG(Quantity) = 2
ORDER BY 
    TotalRevenue DESC;
```

### 4. Category-Wise Customer Reach
For each product category, calculate the unique number of customers purchasing from it. This will help understand which categories have wider appeal across the customer base.
```sql
WITH MonthlySales AS (
    SELECT 
        DATE_FORMAT(order_date, '%Y-%m') AS month,
        SUM(total_amount) AS TotalSales
    FROM 
        Orders
    GROUP BY 
        month
)
SELECT 
    ms1.month, 
    ms1.TotalSales,
    ROUND(
        (ms1.TotalSales - COALESCE(ms2.TotalSales, 0)) / NULLIF(ms2.TotalSales, 0) * 100, 
        2
    ) AS PercentChange
FROM 
    MonthlySales ms1
LEFT JOIN 
    MonthlySales ms2 ON ms1.month = DATE_FORMAT(DATE_ADD(ms2.month, INTERVAL 1 MONTH), '%Y-%m')
ORDER BY 
    ms1.month;
```

### 5. Sales Trend Analysis
Analyze the month-on-month percentage change in total sales to identify growth trends.
```sql
SELECT 
    DATE_FORMAT(order_date, '%Y-%m') AS month,
    SUM(total_amount) AS TotalSales,
    ROUND(
        (SUM(total_amount) - LAG(SUM(total_amount), 1) OVER (ORDER BY DATE_FORMAT(order_date, '%Y-%m'))) 
        / NULLIF(LAG(SUM(total_amount), 1) OVER (ORDER BY DATE_FORMAT(order_date, '%Y-%m')), 0) * 100, 
        2
    ) AS PercentChange
FROM 
    Orders
GROUP BY 
    month
ORDER BY 
    month;
```

### 6. Average Order Fluctuation
Examine how the average order value changes month-on-month. Insights can guide pricing and promotional strategies to enhance order value.
```sql
WITH MonthlyOrderValue AS (
    SELECT 
        DATE_FORMAT(order_date, '%Y-%m') AS Month,
        AVG(total_amount) AS AvgOrderValue
    FROM 
        Orders
    GROUP BY 
        DATE_FORMAT(order_date, '%Y-%m')
)
SELECT 
    mov.Month,
    mov.AvgOrderValue,
    ROUND(mov.AvgOrderValue - LAG(mov.AvgOrderValue) OVER (ORDER BY mov.Month), 2) AS ChangeInValue
FROM 
    MonthlyOrderValue mov
ORDER BY 
    ChangeInValue DESC;
```

### 7. Inventory Refresh Rate
Based on sales data, identify products with the fastest turnover rates, suggesting high demand and the need for frequent restocking.
```sql
SELECT 
    product_id, 
    COUNT(order_id) AS SalesFrequency
FROM 
    OrderDetails
GROUP BY 
    product_id
ORDER BY 
    SalesFrequency DESC
LIMIT 5;
```

### 8. Low Engagement Products
List products purchased by less than 40% of the customer base, indicating potential mismatches between inventory and customer interest.
```sql
WITH TotalCustomers AS (
    SELECT 
        COUNT(DISTINCT customer_id) AS TotalCustomerCount
    FROM 
        Customers
), ProductCustomerCount AS (
    SELECT 
        p.product_id, 
        p.name, 
        COUNT(DISTINCT o.customer_id) AS UniqueCustomerCount
    FROM 
        Products p
    JOIN 
        OrderDetails od ON p.product_id = od.product_id
    JOIN 
        Orders o ON od.order_id = o.order_id
    GROUP BY 
        p.product_id, 
        p.name
)
SELECT 
    pcc.Product_id,
    pcc.name AS Name, 
    pcc.UniqueCustomerCount
FROM 
    ProductCustomerCount pcc
CROSS JOIN 
    TotalCustomers tc
WHERE 
    pcc.UniqueCustomerCount < (0.40 * tc.TotalCustomerCount);
```

### 9. Customer Acquisition Trends
Evaluate the month-on-month growth rate in the customer base to understand the effectiveness of marketing campaigns and market expansion efforts.
```sql
WITH FirstPurchase AS (
    SELECT 
        CUSTOMER_ID, 
        MIN(Order_Date) AS FirstOrderDate
    FROM 
        Orders
    GROUP BY 
        CUSTOMER_ID
)
SELECT 
    DATE_FORMAT(FirstOrderDate, '%Y-%m') AS FirstPurchaseMonth, 
    COUNT(CUSTOMER_ID) AS TotalNewCustomers
FROM 
    FirstPurchase
GROUP BY 
    FirstPurchaseMonth
ORDER BY 
    FirstPurchaseMonth ASC;
```

### 10. Peak Sales Period Identification
Identify the months with the highest sales volume, aiding in planning for stock levels, marketing efforts, and staffing in anticipation of peak demand periods.
```sql
SELECT 
    DATE_FORMAT(STR_TO_DATE(order_date, '%Y-%m-%d'), '%Y-%m') AS Month,  
    SUM(total_amount) AS TotalSales                                  
FROM 
    Orders
GROUP BY 
    Month                                                             
ORDER BY 
    TotalSales DESC                                                    
LIMIT 3;      
```

## Insights and Results
1. **Which of the cities must be focused on as a part of marketing strategies?**  
   *Delhi, Jaipur, Chennai*

2. **As per the Engagement Depth Analysis question, what is the trend of the number of customers vs. number of orders?**  
   *As the number of orders increases, the customer count decreases.*

3. **As per the Engagement Depth Analysis question, which customer category does the company experience the most?**  
   *Occasional shoppers.*

4. **Among products with an average purchase quantity of two, which ones exhibit the highest total revenue?**  
   *Product 1.*

5. **Which product category needs more focus as it is in high demand among the customers?**  
   *Electronics.*

6. **As per Sales Trend Analysis question, during which month did the sales experience the largest decline?**  
   *February 2024.*

7. **As per Sales Trend Analysis question, what could be inferred about the sales trend from March to August?**  
   *Sales fluctuated with no clear trend.*

8. **Which month has the highest change in the average order value?**  
   *December.*

9. **Which product_id has the highest turnover rates and needs to be restocked frequently?**  
   *Product ID 7.*

10. **Why might certain products have purchase rates below 40% of the total customer base?**  
    *Poor visibility on the platform.*

11. **What could be a strategic action to improve the sales of underperforming products?**  
    *Implement targeted marketing campaigns to raise awareness and interest.*

12. **What can be inferred about the growth trend in the customer base from the result table?**  
    *There is a downward trend, implying that the marketing campaigns are not very effective.*

13. **Which months will require major restocking of products and increased staff?**  
    *September, December.*


