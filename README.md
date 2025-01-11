# 🔎 Customers and Products Analysis

## 📌 Introduction
Conduct Sales Data Analysis by answering below questions:

- Question 1: Which products should we order more of or less of?
- Question 2: How should we tailor marketing and communication strategies to customer behaviors?
- Question 3: How much can we spend on acquiring new customers?

## Database structure
<img width="400" alt="image" src=>

## 🔑 Questions and answers

### Write a query to compute the low stock for each product using a correlated subquery
Low Stock = SUM(quantityOrdered) / quantityInStock
````sql
SELECT productCode, ROUND(SUM(quantityOrdered) * 1.0 /
(SELECT quantityInStock 
FROM products p 
WHERE od.productCode = p.productCode), 2) AS Low_Stock

FROM orderdetails od
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10
````
<img width="400" alt="image" src=>

### Write a query to compute the product performance for each product
Low Stock = SUM(quantityOrdered) / quantityInStock
````sql
SELECT productCode, SUM(quantityOrdered * priceEach) AS Product_Performance
FROM orderdetails
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10
````
<img width="400" alt="image" src=>

### Combine the previous queries using a Common Table Expression (CTE) to display priority products for restocking using the IN operator
````sql
WITH low_stock AS (
SELECT productCode, 
ROUND(SUM(quantityOrdered) * 1.0 /
(SELECT quantityInStock 
FROM products p
WHERE od.productCode = p.productCode), 2) AS Low_Stock

FROM orderdetails od
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10)

SELECT productCode, SUM(quantityOrdered * priceEach) AS Product_Performance
FROM orderdetails
WHERE productCode in (SELECT productCode FROM low_stock)
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10
````
<img width="400" alt="image" src=>

In above picture, we know what products to order more and order less. Which answers the first question.

### Computing profit for each customer 
````sql
SELECT CustomerNumber, SUM(quantityOrdered * (priceEach - buyPrice)) AS Revenue
FROM products p
JOIN orderdetails od
ON p.productcode = od.productCode
JOIN orders o
ON od.ordernumber = o.orderNumber
GROUP BY 1
````
<img width="400" alt="image" src=>

### Find TOP 5 VIP customers that bring in the most profit for the store
````sql
WITH rev AS (SELECT CustomerNumber, SUM(quantityOrdered * (priceEach - buyPrice)) AS Revenue
FROM products p
JOIN orderdetails od
ON p.productcode = od.productCode
JOIN orders o
ON od.ordernumber = o.orderNumber
GROUP BY 1)

SELECT contactLastName, contactFirstName, city, country, Revenue
FROM customers c
JOIN rev r
ON c.customerNumber = r.customerNumber
ORDER BY 5 DESC
LIMIT 5
````
<img width="400" alt="image" src=>

This answers the second question. We can have events to drive loyalty for the VIPs
Also, we can launch a campaign for the less engaged customers that bring in less profit

### Finding number of new customers arriving each month
````sql
WITH 

payment_with_year_month_table AS (
SELECT *, 
       CAST(SUBSTR(paymentDate, 1,4) AS INTEGER)*100 + CAST(SUBSTR(paymentDate, 6,7) AS INTEGER) AS year_month
  FROM payments p
),

customers_by_month_table AS (
SELECT p1.year_month, COUNT(*) AS number_of_customers, SUM(p1.amount) AS total
  FROM payment_with_year_month_table p1
 GROUP BY p1.year_month
),

new_customers_by_month_table AS (
SELECT p1.year_month, 
       COUNT(DISTINCT customerNumber) AS number_of_new_customers,
       SUM(p1.amount) AS new_customer_total,
       (SELECT number_of_customers
          FROM customers_by_month_table c
        WHERE c.year_month = p1.year_month) AS number_of_customers,
       (SELECT total
          FROM customers_by_month_table c
         WHERE c.year_month = p1.year_month) AS total
  FROM payment_with_year_month_table p1
 WHERE p1.customerNumber NOT IN (SELECT customerNumber
                                   FROM payment_with_year_month_table p2
                                  WHERE p2.year_month < p1.year_month)
 GROUP BY p1.year_month
)

SELECT year_month, 
       ROUND(number_of_new_customers*100/number_of_customers,1) AS number_of_new_customers_props,
       ROUND(new_customer_total*100/total,1) AS new_customers_total_props
  FROM new_customers_by_month_table;
````
<img width="400" alt="image" src=>

As you can see, the number of clients has been decreasing since 2003, and in 2004, we had the lowest values. 

The year 2005, which is present in the database as well, isn't present in the table above, this means that the store has not had any new customers since September of 2004. This means it makes sense to spend money acquiring new customers.

To determine how much money we can spend acquiring new customers, we can compute the Customer Lifetime Value (LTV), which represents the average amount of money a customer generates. We can then determine how much we can spend on marketing.

### Determine how much money we can spend acquiring new customers, we can compute Customer Lifetime Value (LTV), which represents the average amount of money a customer generates
````sql
WITH rev AS (SELECT CustomerNumber, SUM(quantityOrdered * (priceEach - buyPrice)) AS Revenue
FROM products p
JOIN orderdetails od
ON p.productcode = od.productCode
JOIN orders o
ON od.ordernumber = o.orderNumber
GROUP BY 1)

SELECT ROUND(AVG(Revenue), 2) as LTV
FROM rev
````

<img width="400" alt="image" src=>

This is how much we can spend to accquire new customers
