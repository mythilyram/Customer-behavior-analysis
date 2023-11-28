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

![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/61a938f7-65be-40ae-891d-e08f9bb6639b)

### Calculating the beginning of the current period
In analyzing customer behavior **it is often important to compute the number of customers registered this month, this quarter, this year**, etc. We’ll now learn how to calculate the beginning of the current time period in SQL. In PostgreSQL, there is a function named **DATE_TRUNC()**. Take a look:

![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/7b871c17-3871-48df-a17f-03f677c017a9)
<!SELECT DATE_TRUNC('month', CURRENT_TIMESTAMP) AS current_month_start;>
The expression CURRENT_TIMESTAMP returns the current timestamp, that is the current date and time.

The DATE_TRUNC() function has two arguments:

- The text defining precision, e.g., 'minute', 'hour', 'day', 'month', 'quarter', or 'year'.
- The date that is to be rounded down with a precision defined in the first argument.

The DATE_TRUNC() function returns a timestamp with unwanted details removed, e.g., the result of:
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/4c7a8d34-c6be-4adf-8566-db07148db218)
<!--SELECT DATE_TRUNC('day', '2019-09-09 12:31:08.5'::timestamp)-->
is 2019-09-09 00:00:00.0 and the result of:
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/0f43f51f-b976-4105-badc-627282d6d92c)
<!SELECT DATE_TRUNC('year', '2018-07-10'::timestamp)>
is 2018-01-01 00:00:00.0.

Exercise
In a column named current_year_start, show the beginning of the current year using the function DATE_TRUNC().
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/64229c3a-d67e-4a29-ba4e-3f28ae839515)

### Registration count for the current time period
We can now calculate the registration count for the current week, month, year, etc. with this query:
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/b7bb9cfa-e1cb-440f-9d4b-143b024cacfb)

<!--SELECT COUNT(customer_id) AS registration_count
FROM customers
WHERE registration_date >= DATE_TRUNC('year', CURRENT_TIMESTAMP);-->
We used the DATE_TRUNC() function to find the beginning of the year to get the number of registrations in the current year.

Exercise
Show the number of registrations in the current week. Name the column registrations_current_week.
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/7d595841-8d7b-40bf-b66b-9401a3a33fef)

### Registration over time report
We now want **to find out how customer acquisition has changed over time**. This will help us **understand if we're currently attracting more users (or not)**.

To that end, we'll compare the registration count values for various periods (year to year, month to month, etc.). We can use the DATE_TRUNC() function to create such reports:
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/405b5fd8-0620-4c34-8931-bb47086b4c44)
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/12cbf5a3-5a09-4115-b1c3-fcadf296480d)

<!SELECT
  DATE_TRUNC('year', registration_date) AS registration_year,
  COUNT(customer_id) AS registration_count
FROM customers
GROUP BY DATE_TRUNC('year', registration_date)
ORDER BY DATE_TRUNC('year', registration_date);
Result:

registration_year	registration_count
2017-01-01 00:00:00	332
2018-01-01 00:00:00	409
2019-01-01 00:00:00	259>
In our example, DATE_TRUNC('year', registration_date) returns the first day year of user registration (2016, 2017, 2018, ...). We can use DATE_TRUNC() with other periods, such as 'quarter', 'month', 'week', etc. to group data by a different period.

Note that we also added an ORDER BY clause to make sure the registration count values are shown in chronological order.

Exercise
Create a report containing the 2017 monthly registration counts. Show the registration_month and registration_count columns. Order the results by month.
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/b1c53448-9db6-484e-9e28-56b26646fa16)
<!select date_trunc('month',registration_date) as registration_month,
count(customer_id) as registration_count
from customers
where date_trunc('year',registration_date) >= '2017-01-01' AND 
	date_trunc('year',registration_date) < '2018-01-01'
group by date_trunc('month',registration_date)
order by date_trunc('month',registration_date)>

### Registration over time report – exercise
Good job! We can also create a more advanced version of this report by choosing a different grouping method. For instance, suppose we need to find the registration count for each quarter of each year. This will allow us to compare values for the same quarter across different years.

SELECT
  DATE_TRUNC('quarter', registration_date) AS registration_quarter,
  COUNT(customer_id) AS registration_count
FROM customers
GROUP BY DATE_TRUNC('quarter', registration_date)
ORDER BY DATE_TRUNC('quarter', registration_date);
Result:

registration_year	registration_count
...
2016-10-01 00:00:00	7
2017-01-01 00:00:00	79
2017-04-01 00:00:00	85
...
Note that we use DATE_TRUNC() function in three places: in the SELECT, GROUP BY, and ORDER BY clauses.

Exercise
Find the registration count for each month in each year. Show the following columns: registration_month, and registration_count. Order the results by the month.
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/ae649e12-7cf5-43ea-acee-348fea89a2ee)
<!-- SELECT
  DATE_TRUNC('month', registration_date) AS registration_month,
  COUNT(customer_id) AS registration_count
FROM customers
GROUP BY DATE_TRUNC('month', registration_date)
ORDER BY DATE_TRUNC('month', registration_date);>

## **Customer cohorts**
A customer cohort is **a group of customers that share a common characteristic over a certain time period.** For instance, a group of customers that registered in a certain week can be a weekly registration cohort. Cohorts are often used when creating reports related to customer behavior.

Now, let's take a look at the customer cohorts for a given channel. **A channel defines how we obtain a customer: from social media, via friend referral**, etc.

### Customer cohort report
We now want to create a report that will count registration values in annual customer cohorts based on the channel. To that end, we can use the following query:

SELECT
  DATE_TRUNC('year', registration_date) AS registration_year,
  channel_name,
  COUNT(*) AS registration_count
FROM customers cu
JOIN channels ch
  ON cu.channel_id = ch.id
GROUP BY DATE_TRUNC('year', registration_date), channel_name
ORDER BY DATE_TRUNC('year', registration_date);
Result:

registration_year	channel_name	registration_count
2016-01-01 00:00:00	Direct	2
2016-01-01 00:00:00	Organic Search	5
2017-01-01 00:00:00	Referral	16
...
The customers table contains the channel ID for each user, but we join it with the channels table to get channel names for the report. Note that we only show channel names in the report, but we group by both the channel IDs and channel names. If two channels with different IDs had the same name (for whatever reason), this trick would prevent us from summing up their values improperly.

Exercise

Create an extended version of the report shown in the explanation. Instead of annual customer cohorts per channel, show weekly customer cohorts per channel in each year.

Show the following columns: registration_week, channel_name, and registration_count.

Order the results by week.
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/4292422e-db57-4500-a6c9-5170463e64e8)

Exercise

Create a report to show the weekly counts of registration cohorts in 2017, based on the customer country. Show the following columns: registration_week, country, and registration_count. Order the results by week.
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/6e4f3a5e-8284-4e89-a278-93dcb5cf8f5f)

## Summary
Perfect! It's time to wrap things up. What have we learned in this section?

To count registrations during a given period of time, use:
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/c4d5c867-fb67-40c6-85bd-4b26ef257c5d)
<!--SELECT COUNT(customer_id) AS ...
FROM customers
WHERE registration_date >= ...
  AND registration_date <  ...;-->
To count registrations from the current period (e.g., the current year), use:
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/321c0244-4910-4e60-b677-7412981c1099)
<!--SELECT COUNT(customer_id) AS registration_count
FROM customers
WHERE registration_date >= DATE_TRUNC('year', CURRENT_TIMESTAMP);-->
To show a report of registration counts for all years, use:
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/4e9c0671-f63f-46f8-9640-33d6191a0ea5)
<!--SELECT
  DATE_TRUNC('year', registration_date) AS registration_year,
  COUNT(customer_id) AS registration_count
FROM customers
GROUP BY DATE_TRUNC('year', registration_date)
ORDER BY DATE_TRUNC('year', registration_date);-->
A customer cohort is a group of customers that share a common characteristic over a certain time period. To show annual registration cohorts per channel, use:
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/2a3b78d0-b773-4c7b-9157-a75781251d8a)
<!--SELECT
  DATE_TRUNC('year', registration_date) AS registration_year,
  channel_name,
  COUNT(*) AS registration_count
FROM customers cu
JOIN channels ch
  ON cu.channel_id = ch.id
GROUP BY DATE_TRUNC('year', registration_date), channel_name
ORDER BY DATE_TRUNC('year', registration_date);-->

Exercise
1. Show the number of registrations in the current year. Name the column registrations_this_year.
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/e400c50f-262a-45be-9d8f-ef92f314c176)

2. Create a report showing the weekly counts of registration cohorts in 2017 based on the customer channel. Show the following columns: registration_week, channel_name, and registration_count. Sort the result by the week.
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/f6542951-d21e-40f8-ac4f-868451d52ca7)

#  customer conversion

Customer conversion is defined as "any desired event or action that is related to customers." A conversion occurs when a customer completes a desired action, such as signing up for the newsletter, social media sharing, filling out a form, or making a purchase. We typically talk about conversion rates, which are the percentages of prospective customers who take a specific action that we want.

In our data model, we'll take a look at registered customers. How many of them make their first purchase within one week of registration, two weeks of registration, etc.? Customers may register without making any purchase simply to get our newsletter; they'll make their first purchase only when they get a newsletter with an offer that appeals to them. In this case, analyzing conversion rates can help us find out which newsletter campaigns perform better than the others.

We'll learn how to compute the total conversion rate in a given period. Then, we'll analyze how long – for each weekly registration cohort – it takes our customers to make their first order. Finally, we'll learn how to prepare a customer conversion chart that shows weekly registration cohorts in rows and the number of conversions (i.e., first purchases) within one week, two weeks, etc. A customer conversion chart can look like this:

![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/4802c99d-245a-4d6c-a431-a8cc17e2325c)

### Important columns in the customers table
First, let's take another look at the customers table. It contains some important columns that we didn't use in the previous part.
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/fa00f40f-4899-4f24-9a16-5a7bab28e23d)

### Global lifetime conversion rate – step 1
 we want to know the **percentage of registered customers who've made at least one purchase**. This is a **good indicator of the general condition of our business.**

**To calculate the rate, we first need to find the total number of customers and the number of customers who've made at least one purchase.** That's quite straightforward:

![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/432e2366-dfb7-4563-bb5c-ae75feb61beb)
<!--SELECT
  COUNT(first_order_id) AS customers_with_purchase,
  COUNT(*) AS all_customers
FROM customers;-->
Recall that **COUNT(*) counts all rows in a table**(in this case, all the customers), while **COUNT(column) only counts non-NULL values** in the specified column. Customers who didn't make any purchases have NULL first_order_id values, so they won't be added to the customers_with_purchase column.

**Exercise:** 
Among customers registered in 2017, show how many made at least one purchase (name the column customers_with_purchase) and the number of all the customers registered in 2017 (name the column all_customers).
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/e1ad6070-5950-451d-ab2d-a4c3f3548942)

### Global lifetime conversion rate – step 2

![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/7876e770-3953-42ae-89eb-1fc8923e11c0)

<!--SELECT ROUND(COUNT(first_order_id) / COUNT(*)::numeric, 2) AS conversion_rate
FROM customers;-->
We simply divided the number of made-a-purchase customers by the total number of registered customers. Notice that **we cast the denominator to a numeric type using COUNT(*)::numeric. Both the numerator and the denominator are integers, so PostgreSQL would use integer division by default. The result would be incorrect, so we had to cast one of the values to force float division**.

We also used the ROUND(value, number) function, which rounds the given value to the given number of decimal places.

**Exercise**:Find the lifetime conversion rate among customers who registered in 2017. Show the result in a column named conversion_rate. Round the result to four decimal places.
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/0747b17d-6204-4786-a6f5-67b525c33755)

### Global lifetime conversion percentage
 We can also express the conversion rate as a percentage. Take a look:

![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/95962181-ed58-40e1-949b-164f027bae84)
SELECT ROUND(COUNT(first_order_id)** * 100.0 **/ COUNT(*), 2) AS conversion_rate
FROM customers;

As you can see, we multiplied the numerator by 100.0 to get a percentage. **Note that 100.0 is a floating point number. Multiplying the numerator by 100.0 means the whole expression will use float division. That's why we didn't have to cast values to a different data type this time.**

Exercise
Modify the template which contains the code from the previous exercise, so that it shows the conversion rate as a percentage. Round it to two decimal places.

SELECT ROUND(COUNT(first_order_id) *100.0/ COUNT(*)::numeric, 2) AS conversion_rate
FROM customers
WHERE registration_date >= '2017-01-01'
  AND registration_date <  '2018-01-01';
