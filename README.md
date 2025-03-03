# SQL Subqueries - Lab

## Introduction

Now that you've seen how `subqueries` work, it's time to get some practice writing them! Not all of the queries will require subqueries, but all will be a bit more complex and require some thought and review about aggregates, grouping, ordering, filtering, joins and subqueries. Good luck!  
## Objectives

You will be able to:

* Write `subqueries` to decompose complex queries

## CRM Database ERD

Once again, here's the schema for the CRM database you'll continue to practice with.

<img src="https://curriculum-content.s3.amazonaws.com/data-science/images/Database-Schema.png" width="600">

## Connect to the Database

As usual, start by importing the necessary packages and connecting to the database `data.sqlite`.

```python
# Your code here; import the necessary packages
import pandas as pd 
import sqlite3
# Your code here; create the connection
conn = sqlite3.connect('data.sqlite')
## Write an Equivalent Query using a Subquery
```

The following query works using a `JOIN`. Rewrite it so that it uses a subquery instead.

``` python
SELECT
    customerNumber,
    contactLastName,
    contactFirstName
FROM customers
JOIN orders 
    USING(customerNumber)
WHERE orderDate = '2003-01-31'
;
```

```python
# brian-answer
# -----------------------------------------------------------------
    # q = """
    # SELECT customerNumber, contactLastName, contactFirstName, orderDate
    # FROM customers 
    # JOIN orders 
    # USING(customerNumber)
    # WHERE orderDate = '2003-01-31';
    # """
# -----------------------------------------------------------------
q = """
SELECT customerNumber, contactLastName, contactFirstName, orderDate 
FROM customers
JOIN orders
USING(customerNumber)
WHERE customerNumber IN 
    (SELECT customerNumber FROM orders WHERE orderDate = '2003-01-31');
"""
# -----------------------------------------------------------------
pd.read_sql(q, conn)
## Select the `Total Number` of Orders for Each `Product Name`
```
Sort the results by the total number of items sold for that product.

```python
# brian-answer
q = """
    SELECT
        productName,
        COUNT(orderNumber) AS numberOrders,
        SUM(quantityOrdered) AS totalUnitsSold
    FROM 
        products
    JOIN orderdetails
        USING (productCode)
    GROUP BY 
        productName
    ORDER BY 
        totalUnitsSold DESC;
"""
pd.read_sql(q, conn)
## Select the `Product Name` and the `Total Number` of People Who Have Ordered Each Product
```

Sort the results in descending order.
### A quick note on the SQL  `SELECT DISTINCT` statement:
The `SELECT DISTINCT` statement is used to return only distinct values in the specified column. In other words, it removes the duplicate values in the column from the result set.

Inside a table, a column often contains many duplicate values; and sometimes you only want to list the unique values. If you apply the `DISTINCT` clause to a column that has `NULL`, the `DISTINCT` clause will keep only one NULL and eliminates the other. In other words, the DISTINCT clause treats all `NULL` “values” as the same value.

```python
# Your code here
# Hint: because one of the tables we'll be joining has duplicate customer numbers, you should use DISTINCT
q = """
    SELECT 
        productName, COUNT(DISTINCT customerNumber) AS numPurchasers
    FROM 
        products
    JOIN orderdetails
        USING(productCode)
    JOIN orders
        USING(orderNumber)
    GROUP BY 
        productName
    ORDER BY 
        numPurchasers DESC;
"""
pd.read_sql(q, conn)
## Select the `Employee Number`, `First Name`, `Last Name`, `City (of the office)`, and `Office Code` of the `Employees` Who Sold Products That Have Been Ordered by Fewer Than 20 people.
```

This problem is a bit tougher. To start, think about how you might break the problem up. Be sure that your results only list each employee once.

```python
# brian-added
q = """
    SELECT
        DISTINCT employeeNumber,
        officeCode,
        o.city,
        firstName,
        lastName
    FROM 
        employees AS e
    JOIN offices AS o
        USING(officeCode)
    JOIN customers AS c
        ON e.employeeNumber = c.salesRepEmployeeNumber
    JOIN orders
        USING(customerNumber)
    JOIN orderdetails
        USING(orderNumber)
    WHERE productCode IN (
        SELECT productCode FROM products
        JOIN orderdetails
            USING(productCode)
        JOIN orders
            USING(orderNumber)
        GROUP BY 
            productCode
        HAVING COUNT(DISTINCT customerNumber) < 20);
"""
pd.read_sql(q, conn)
```

## Select the Employee Number, First Name, Last Name, and Number of Customers for Employees Whose Customers Have an Average Credit Limit Over 15K

```python
# brian-added
q = """
    SELECT
        employeeNumber,
        firstName,
        lastName,
        COUNT(customerNumber) AS numCustomers
    FROM 
        employees AS e
    JOIN customers As c
        ON e.employeeNumber = c.salesRepEmployeeNumber
    GROUP BY 
        employeeNumber
    HAVING 
        AVG(creditLimit) > 15000;
"""
pd.read_sql(q, conn)
```

## Finally, `close` the connection.

```python
conn.close()
```

## Summary

In this lesson, you got to practice some more complex SQL queries, some of which required subqueries. There's still plenty more SQL to be had though; hope you've been enjoying some of these puzzles!