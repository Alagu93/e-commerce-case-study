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
1. **Customer**: 
--Identify the top 3 cities with the highest number of customers to determine key markets for targeted marketing and logistic optimization.
SELECT cu.location AS location, COUNT(DISTINCT cu.customer_id) AS number_of_customers
FROM Customers cu
GROUP BY cu.location
ORDER BY number_of_customers DESC
LIMIT 3;

2. **Product**: 
   - SQL query for best-selling products
   - SQL query for average product price by category

3. **Orders**: 
  - Determine the distribution of customers by the number of orders placed. This insight will help in segmenting customers into one-time buyers, occasional shoppers, and regular customers for tailored marketing strategies.

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

4. **OrderDetails**: 
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

## Insights and Results
1.Which of the cities must be focused as a part of marketing strategies?
Delhi,Jaipur,chennai
2.As per the Engagement Depth Analysis question, What is the trend of the number of customers v/s number of orders?
 As the Number of orders increases, the Customer count decreases. 
3.As per the Engagement Depth Analysis question, Which customers category does the company experiences the most?
 Occasional shoppers 
