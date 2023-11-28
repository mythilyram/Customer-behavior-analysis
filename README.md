# Customer-behavior-analysis
Acquisition, conversion, customer activity, retention, and churn

In this Project, We`ll analyze customer behavior using PostgreSQL. We'll use information from an imaginary online supermarket to analyze the entire customer lifecycle.

- **Acquisition** is the stage when the customers **learn more about the company**'s offerings from **visits to the website**, or by **testing products** in a store. In an online business, this stage **ends with customer registration on the website**.

- **Conversion** is the stage where customers **purchase a product or service**.

- **Retention**, Now that a new customer has been acquired, the focus is to **help the customer derive satisfaction and value from the product and services**. In this stage, **you can measure customer activity**: how much they purchase, how often they use your product, etc.

- **Customer churn** is the final stage in the customer lifecycle. It is  Inevitably, customers will stop using your product. Customer churn is the **percentage of customers who stopped using your product or service during a certain time frame.**

Understanding the customer lifecycle is very important for all businesses. We'll prepare commonly used reports to analyze different stages of the customer lifecycle. 

## The data model
It's based on Microsoft's "Northwind" data model. You can find the original database [here](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/northwind-pubs).

Our modified database consists of eight tables. You can see the entire database diagram below. 
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/12fd33e8-98ad-4932-b3cd-3db20d35e43a)

### Customer registrations in a time period
- Let's start with customer acquisition.
- We'll begin by counting how many new registrations occurred in a given period.
- By observing how the registration count changes over time, we'll be able to assess whether our promotional campaigns have worked or not.

We'll discuss the following reports:

- The number of registrations in the given period.
- The number of registrations in the current period of time (current week, current month, etc.).
- The number of registrations over time, tallying registrations in each month or in each week.

To count the number of registrations in 2016, we can use the following query:

![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/b084aed0-87eb-49f7-b1f3-f00976fdaaf4)

The query is quite simple. Just remember **proper time filtering: use the 'yyyy-mm-dd' format (year, month, day) when you write the dates**.

Exercise
How many customers registered in the first six months of 2017? Name the column registration_count.
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/72c97c2f-693d-41e9-87f0-bdedccb1e4d4)

### Registrations in a time period with interval arithmetic
Perfect! To count registrations in 2016, we use the following WHERE clause:
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/5236f9bc-7fcb-41df-9683-57f3e2d34c59)

There is also an alternative way of writing this query. Take a look:

![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/0af7ff1e-d079-4e2b-9a43-5f4596a8c15a)
<!--SELECT COUNT(customer_id) AS registration_count_2016
FROM customers
WHERE registration_date >= '2016-01-01'
  AND registration_date <  '2016-01-01'::date + INTERVAL '1' year;-->
This time, **instead of providing the end date specifically, we used the INTERVAL 'number' datepart function**. 

It takes two arguments:

- number – the number of intervals to add.
- datepart – the interval unit we want to add, such as: year or month.
In our example, '2016-01-01'::date + INTERVAL '1' year means "add one year to the date of Jan 1, 2016". Remember to **cast the string with the date to date (by typing ::date)**, so the database knows that you want to operate on time values.

Exercise
Modify the template which contains the query from the previous exercise, so that it uses the INTERVAL. (We want to see the first half of the year 2017.)
