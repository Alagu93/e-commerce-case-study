# E-Commerce Case Study

## Project Description
This project involves data analysis for an e-commerce company to extract insights from various datasets to drive business strategies.

## Technologies Used
- MySQL
- SQL queries

## Datasets
- **Customers Dataset**: customer_id, name, location
- **Products Dataset**: product_id, name, category, price
- **Orders Dataset**: order_id, order_date, customer_id, total_amount
- **OrderDetails Dataset**: order_id, product_id, quantity, price_per_unit

## Queries

**Market segment analysis**: 
Identify the top 3 cities with the highest number of customers to determine key markets for targeted marketing and logistic optimization.
SELECT cu.location AS location, COUNT(DISTINCT cu.customer_id) AS number_of_customers
FROM Customers cu
GROUP BY cu.location
ORDER BY number_of_customers DESC
LIMIT 3;

**Engagement Depth Analysis**: 
Determine the distribution of customers by the number of orders placed. This insight will help in segmenting customers into one-time buyers, occasional shoppers, and regular customers for tailored marketing strategies.

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

 **purchase high value product**: 
  Identify products where the average purchase quantity per order is 2 but with a high total revenue, suggesting premium product trends.
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
    
**category-wise customer reach**:
   For each product category, calculate the unique number of customers purchasing from it. This will help understand which categories have wider appeal across the customer base.
   WITH MonthlySales AS (
    SELECT 
        DATE_FORMAT(order_date, '%Y-%m') AS month,  -- Format the order date to YYYY-MM
        SUM(total_amount) AS TotalSales  -- Sum up total sales for each month
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
    MonthlySales ms2 ON ms1.month = DATE_FORMAT(DATE_ADD(ms2.month, INTERVAL 1 MONTH), '%Y-%m')  -- Join to get previous month
ORDER BY 
    ms1.month;
    
**Sales Trend Analysis**:
Analyze the month-on-month percentage change in total sales to identify growth trends.
SELECT 
    DATE_FORMAT(order_date, '%Y-%m') AS month,  -- Format the order date to YYYY-MM
    SUM(total_amount) AS TotalSales,  -- Sum up total sales for each month
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
    
**Average Order FLuctuation**:
Examine how the average order value changes month-on-month. Insights can guide pricing and promotional strategies to enhance order value.
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
    
**Inventory refresh rate**:
Based on sales data, identify products with the fastest turnover rates, suggesting high demand and the need for frequent restocking.
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

**Low engagement products**:
List products purchased by less than 40% of the customer base, indicating potential mismatches between inventory and customer interest.
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
        OrderDetails od 
    ON 
        p.product_id = od.product_id
    JOIN 
        Orders o 
    ON 
        od.order_id = o.order_id
    GROUP BY 
        p.product_id, 
        p.name
)
SELECT 
    pcc.Product_id,
    pcc.name as Name, 
    pcc.UniqueCustomerCount
FROM 
    ProductCustomerCount pcc
CROSS JOIN 
    TotalCustomers tc
WHERE 
    pcc.UniqueCustomerCount < (0.40 * tc.TotalCustomerCount);
    
**customer acquistion trends**:
Evaluate the month-on-month growth rate in the customer base to understand the effectiveness of marketing campaigns and market expansion efforts.
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
    
**Peak sales period Identification**:
Identify the months with the highest sales volume, aiding in planning for stock levels, marketing efforts, and staffing in anticipation of peak demand periods.
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

## Insights and Results
1.Which of the cities must be focused as a part of marketing strategies?
Delhi,Jaipur,chennai
2.As per the Engagement Depth Analysis question, What is the trend of the number of customers v/s number of orders?
 As the Number of orders increases, the Customer count decreases. 
3.As per the Engagement Depth Analysis question, Which customers category does the company experiences the most?
 Occasional shoppers 
4.Among products with an average purchase quantity of two, which ones exhibit the highest total revenue?
Product 1
5.Which product category needs more focus as it is in high demand among the customers?
Electronics
6.As per Sales Trend Analysis question, During which month did the sales experience the largest decline?
Feb 2024
7.As per Sales Trend Analysis question, What could be inferred about the sales trend from March to August?
sales fluctuated with no clear trend
8.Which month has the highest change in the average order value?
December
9.Which product_id has the highest turnover rates and needs to be restocked frequently?
7
10.Why might certain products have purchase rates below 40% of the total customer base?
Poor visibility on the platform
11.After running an analysis to identify products purchased by less than 40% of the customer base, it was found that a few products have lower purchase rates than expected.What could be a strategic action to improve the sales of these underperforming products?
 Implement targeted marketing campaigns to raise awareness and interest. 
12.What can be inferred about the growth trend in the customer base from the result table?
 It is downward trend which implies the marketing campaign are not much effective. 
13.Which months will require major restocking of product and increased staffs? 
september,december
