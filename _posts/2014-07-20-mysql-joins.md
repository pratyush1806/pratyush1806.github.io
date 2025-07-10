---
layout: post
title: "MySQL Joins"
author: pratyush
categories: [ Tutorial ]
image: assets/images/mysql-joins.webp
description: "Real life use cases of MySQL Joins"
comments: true
---

MySQL joins are a fundamental concept in database management, allowing you to combine rows from two or more tables based on related columns between them. This is incredibly useful when you have data distributed across multiple tables, and you need to retrieve and analyze information that spans these tables.

### INNER JOIN
The INNER JOIN, also known as EQUIJOIN, is used to retrieve records that have matching values in both tables. It effectively filters out rows that do not have corresponding matches in the other table. This type of join is often used when you want to extract data that exists in multiple related tables.

Scenario: E-commerce Platform

Imagine you're managing an e-commerce platform with two tables: `customers` and `orders`. You want to retrieve a list of customers who have placed orders along with their order dates.

Query
```
SELECT customers.name, orders.order_date
FROM customers
INNER JOIN orders ON customers.customer_id = orders.customer_id;
```

![Inner Join](/assets/images/inner-join.png)

Use Case: This query would provide you with a list of customers who have made purchases, allowing you to analyze their purchasing behavior and trends.

### LEFT JOIN
The LEFT JOIN retrieves all records from the left table and the matching records from the right table. If there's no match, the result will contain NULL values for the columns from the right table. This is useful when you want to keep all records from the left table regardless of whether they have matching entries in the right table.

Scenario: Content Management System

Suppose you're working on a content management system with tables for `authors` and `articles`. You want to retrieve a list of all authors along with the titles of their articles (if any).

Query
```
SELECT authors.name, articles.title
FROM authors
LEFT JOIN articles ON authors.author_id = articles.author_id;
```

![Left Join](/assets/images/left-join.png)

Use Case: This query enables you to see a list of authors and the articles they've written, even if some authors haven't written any articles yet. It helps you identify active and less active authors.

### RIGHT JOIN
The RIGHT JOIN is similar to the LEFT JOIN but retrieves all records from the right table and the matching records from the left table. The result will contain NULL values for the columns from the left table in case of no match.

Scenario: HR Management System

In an HR database, you have tables for `employees` and `departments`. You want to retrieve a list of all departments along with the employees who work in them.

Query
```
SELECT departments.department_name, employees.employee_name
FROM departments
RIGHT JOIN employees ON departments.department_id = employees.department_id;
```

![Right Join](/assets/images/right-join.png)

Use Case: This query allows you to see a list of departments and the employees in each department, even if a department doesn't have any employees yet. It helps HR managers manage workforce distribution.

### FULL JOIN
> MySQL does not support FULL JOIN keyword. But, a FULL JOIN on two tables can be achieved by using three keywords: LEFT JOIN, RIGHT JOIN and UNION keywords.

Scenario: Analytics and Reporting

Suppose you're working on an analytics project where you have tables for `website_visitors` and `purchases`. You want to retrieve a list of all visitors and any purchases they've made.

Query
```
SELECT website_visitors.name, purchases.purchase_date
FROM website_visitors
LEFT JOIN purchases ON website_visitors.visitor_id = purchases.visitor_id;
UNION
SELECT website_visitors.name, purchases.purchase_date
FROM website_visitors
RIGHT JOIN purchases ON website_visitors.visitor_id = purchases.visitor_id;
```

Use Case: This query would provide a comprehensive view of all website visitors and their purchase history. It's useful for generating detailed reports on customer engagement and conversion rates.

### SELF JOIN
A SELF JOIN is a variation of the conventional join types, where a table is joined with itself. It's an invaluable tool for scenarios where you have hierarchical or related data within the same table.

Scenario: Organizational Hierarchy

Imagine you're building a database for an organization's employee hierarchy. You have a table employees with columns for `employee_id`, `employee_name`, and `supervisor_id`. You want to retrieve a list of employees and their supervisors' names.

Query
```
SELECT e1.employee_name AS employee, e2.employee_name AS supervisor
FROM employees e1
LEFT JOIN employees e2 ON e1.supervisor_id = e2.employee_id;
```

Use Case: This query would allow you to create a clear view of the organizational structure, showing each employee along with their respective supervisor. It's useful for visualizing reporting relationships within the organization.