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
<!--select date_trunc('month',registration_date) as registration_month,
count(customer_id) as registration_count
from customers
where date_trunc('year',registration_date) >= '2017-01-01' AND 
	date_trunc('year',registration_date) < '2018-01-01'
group by date_trunc('month',registration_date)
order by date_trunc('month',registration_date)-->

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
ORDER BY DATE_TRUNC('month', registration_date);-->

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
1. Modify the template which contains the code from the previous exercise, so that it shows the conversion rate as a percentage. Round it to two decimal places.

SELECT ROUND(COUNT(first_order_id) *100.0/ COUNT(*)::numeric, 2) AS conversion_rate
FROM customers
WHERE registration_date >= '2017-01-01'
  AND registration_date <  '2018-01-01';

2. Find the conversion rate for each customer channel. Show the channel_name and conversion_rate columns. Display the conversion rates as percentages rounded to two decimal places.

SELECT
	ROUND(COUNT(first_order_id)*100.0/count(*),2) AS conversion_rate,
    channel_name
FROM customers cu
JOIN channels ch
ON ch.id= cu.channel_id
GROUP BY  channel_name

### Conversion rates in weekly cohorts
 Global lifetime conversion rate is a useful indicator, but now we'd like to run a deeper analysis. We'd like to find the conversion rate for weekly registration cohorts. This way, we can analyze which weekly cohorts had the best conversion rates and compare them against the campaigns we ran at that time. Take a look:

SELECT
  DATE_TRUNC('week', registration_date) AS week,
  ROUND(COUNT(first_order_id) * 100.0 / COUNT(*), 2) AS conversion_rate
FROM customers
GROUP BY DATE_TRUNC('week', registration_date)
ORDER BY DATE_TRUNC('week', registration_date);
Result:

week	conversion_rate
...
2017-01-02 00:00:00	81.82
2017-01-09 00:00:00	83.33
2017-01-16 00:00:00	60.00
2017-01-23 00:00:00	66.67
...
We used the DATE_TRUNC('period', column) function to extract the year and the week from the registration_date column. We included this column in three places: the SELECT, GROUP BY, and ORDER BY clauses. This way, we can compare conversion rates for all weeks in chronological order.

**Exercise**: 
Create a similar report showing conversion rates in monthly cohorts. Display the conversion rates as ratios (not percentages) rounded to three decimal places. Show the following columns: month and conversion_rate. Order the results by month.
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/4b9b11bc-2805-44a6-8851-92a26ec173d0)
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/b2013beb-2b6a-4a44-aa8d-8cc2700eb75a)


### Conversion rates in weekly cohorts – exercise

**Exercise:** Create a report containing the conversion rates for weekly registration cohorts in each registration channel, based on customers registered in 2017. Show the following columns: week, channel_name, and conversion_rate. Format the conversion rates as percentages, rounded to a single decimal place. Order the results by week and channel name.

<!--SELECT
  DATE_TRUNC('week', registration_date) AS week,  
  channel_name,
  ROUND(COUNT(first_order_id) * 100.0 / COUNT(*), 1) AS conversion_rate
FROM customers cu
JOIN channels ch
ON ch.id= cu.channel_id
WHERE DATE_TRUNC('year',registration_date) = '2017-01-01' 
GROUP BY DATE_TRUNC('week', registration_date),channel_name
ORDER BY DATE_TRUNC('week', registration_date),channel_name;-->
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/a7ed2b0c-a4d7-43aa-b12a-4b26c0420539)

## Time to first order
So far, we've only compared the number of customers who've made or didn't make a purchase. Now, we'd like to go deeper and analyze how much time customers take between registration and their first purchase. Check out the following query:

SELECT
  customer_id,
  first_order_date - registration_date AS days_to_first_order
FROM customers;
Result:

customer_id	days_to_first_order
1	NULL
2	10 days
3	19 days
4	NULL
...	...
In the second column, we created an interval end_date - start_date to calculate the interval between the user's registration and their first order.

Exercise
Show customers' emails and interval between their first purchase and the date of registration. Name the column difference.

Observe how the database displays the difference between two dates.
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/9de8fd68-a8ce-476c-ada8-cb0d50f1fc48)

![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/2b614d5a-2453-4264-b527-66a37dc1d68b)

## Global average time to first order
This is a good indicator of how long a typical customer takes to make the first purchase. Take a look:

SELECT AVG(first_order_date - registration_date) AS avg_days_to_first_order
FROM customers;

All we had to do was use AVG() with an interval inside.

Exercise
Find the average time from registration to first order for each channel. Show two columns: channel_name and avg_days_to_first_order

SELECT
	AVG(first_order_date - registration_date) AS avg_days_to_first_order,
    channel_name
FROM customers cu
JOIN channels ch
ON ch.id= cu.channel_id
GROUP BY  channel_name
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/42ab7f8d-a4dc-4dcc-a422-07f11c6e8a6e)

## Average time to first order in weekly cohorts
 Previously, we created a report showing conversion rates in weekly registration cohorts. In a similar way, we can create a report showing the average time from registration to first order in weekly registration cohorts. Take a look:

SELECT
  DATE_TRUNC('week', registration_date) AS week,
  AVG(first_order_date - registration_date) AS avg_days_to_first_order
FROM customers
GROUP BY DATE_TRUNC('week', registration_date)
ORDER BY DATE_TRUNC('week', registration_date);
Result:

week	avg_days_to_first_order
2016-12-12 00:00:00	20 days
2016-12-19 00:00:00	15 days 18:00:00
2016-12-26 00:00:00	12 days 16:00:00
...	...
The report looks very similar to the previous one. Once again, we used an interval to group and order the rows by weekly registration cohorts. The only difference is the avg_days_to_first_order column.

Exercise
Calculate the average number of days that passed between registration and the first order in quarterly registration cohorts. Show the following columns: quarter and avg_days_to_first_order. Order the results by year and quarter.

SELECT
  DATE_TRUNC('quarter', registration_date) AS quarter,
  AVG(first_order_date - registration_date) AS avg_days_to_first_order
FROM customers
GROUP BY DATE_TRUNC('quarter', registration_date)
ORDER BY DATE_TRUNC('quarter', registration_date);

## Average time to first order in weekly cohorts – exercise

Create a report of the average time to first order for weekly registration cohorts from 2017 in each registration channel. Show the following columns: week, channel_name, and avg_days_to_first_order. Order the results by the week.
<!--SELECT
	 DATE_TRUNC('week',registration_date) as week,
     channel_name,
	 AVG(first_order_date - registration_date) AS avg_days_to_first_order
FROM customers cu
JOIN channels ch
ON ch.id= cu.channel_id
WHERE DATE_TRUNC('year',registration_date) = '2017-01-01'
GROUP BY  DATE_TRUNC('week',registration_date),channel_name
ORDER BY DATE_TRUNC('week',registration_date),channel_name-->
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/b6e7f83c-64de-44a6-87ad-935840fdb2a0)
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/4367ba67-3683-48a1-a337-a4e7f1f1ba34)

## Filtering with intervals
Very well done! Let's leave dates for a while, and use some intervals to filter our data. Take a look at this query:

SELECT
  full_name,
  country,
  last_order_date
FROM customers
WHERE first_order_date - registration_date <= INTERVAL '7' day
For those who placed their first order within one week from registration, this query gets their names, countries, and when their last orders were placed.

Exercise
Let's practice filtering with intervals a bit. Find all customers who placed their first order within one month from registration, and their last order within three months from registration – let's see who's stopped ordering. For each customer show these columns: email, full_name, first_order_date, last_order_date.

SELECT
  email,
  full_name,
  first_order_date, 
  last_order_date
FROM customers
WHERE first_order_date - registration_date <= INTERVAL '1' month AND
	last_order_date - registration_date <= INTERVAL '3' month

 ![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/ff50ef92-39a4-4b11-a898-26681e690f15)

## Custom classifications
Excellent! To create the last report type in this part, we'll first need to learn how to create our own classifications.

Suppose we want to show each customer along with a custom category label. The label will indicate how long they took to make their first order.

SELECT
  customer_id,
  registration_date,
  first_order_date,
  CASE
    WHEN first_order_date IS NULL THEN 'no order'
    WHEN first_order_date - registration_date <= INTERVAL '7'  day THEN '1 week'
    WHEN first_order_date - registration_date <= INTERVAL '14' day THEN '2 weeks'
    ELSE 'more than 2 weeks'
  END AS time_to_first_order
FROM customers;
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/f0a89dde-f314-4ccf-a939-27397112e170)

![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/29a48d95-0f24-4239-885c-e8dd660b3343)

The new part in this query is the last column named time_to_first_order. It uses an SQL concept known as CASE WHEN. It checks the expressions introduced after each WHEN keyword. When it finds the first true expression, the corresponding value in the THEN part is assigned to the specified field.

In our query, we first check if the first_order_date column is NULL. If it is, we set the time_to_first_order value to 'no order'. If it isn't, we check if the period between registration and first purchase is equal to or less than 7 days. If it is, we set the time_to_first_order value to '1 week'. If it's not, we check the same period for equal to or less than 14 days (the value '2 weeks'). If this expression doesn't match either, we use the default value from the ELSE clause ('more than 2 weeks'). Note that each CASE WHEN expression must end with the word END.

Exercise
Our e-store has used three versions of the registration form:

'ver1' – introduced when the e-store started.
'ver2' – introduced on Mar 14, 2017.
'ver3' – introduced on Jan 1, 2018.
For each customer, select the customer_id, registration_date, and the form version the user filled in at the time of registration. Name this third column registration_form.

SELECT
  customer_id,
  registration_date,
  CASE
    WHEN registration_date < '2017-03-14' THEN 'ver1'
    WHEN registration_date < '2018-01-01' THEN 'ver2'
    ELSE 'ver3'
  END AS registration_form
FROM customers;
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/7d4e7ec7-69f3-43a1-9dbb-b1d00b4977b6)

## Multiple metrics in one query
Well done! The CASE WHEN construction can also be used to show multiple metrics in a single query. Take a look:

SELECT
  COUNT(CASE
    WHEN registration_date >= '2016-01-01'
     AND registration_date <  '2017-01-01'
    THEN customer_id
  END) AS registrations_2016,
  COUNT(CASE
    WHEN registration_date >= '2017-01-01'
     AND registration_date <  '2018-01-01'
    THEN customer_id
  END) AS registrations_2017
FROM customers;
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/16e504b4-2a9a-4dc2-9922-5db7b3ca7e9f)

We use COUNT() with a CASE WHEN inside. The purpose is to count only the users in each column who match the given criteria. For instance, the registrations_2016 column checks if the registration_date is in 2016. If it is, the customer_id is counted. If the condition isn't satisfied – and there is no alternative condition or ELSE part – CASE WHEN returns NULL and the customer isn't counted.

**COUNT(CASE WHEN...) is a technique used to include multiple metrics in different columns of the same report.**

**Exercise**: Show two metrics in two different columns:

order_on_registration_date – the number of people who made their first order within one day from their registration date.
order_after_registration_date – the number of people who made their first order after their registration date.

![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/13adf759-6e11-49ab-be7f-d60d2e60525d)
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/94336efe-1ec6-4b84-949a-30e3803b2ffa)
SELECT
  COUNT(CASE
    WHEN first_order_date - registration_date < INTERVAL '1' DAY
    THEN customer_id
  END) AS order_on_registration_date ,
  COUNT(CASE
    WHEN first_order_date - registration_date >= INTERVAL '1' DAY
    THEN customer_id
  END) AS order_after_registration_date
FROM customers;

## Conversion rate report
Great! We now want to create a conversion chart. This report should show weekly registration cohorts in rows and the number of conversions (i.e., first purchases) within the first week, the second week, etc. We want the end report to look like this:

![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/ff349b59-ec95-4e05-96f2-bdb702457e4e)

Here's the query:
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/cab33fc3-0128-4bc4-8c28-6b515a5bad37)

SELECT
  DATE_TRUNC('week', registration_date) AS week,
  COUNT(*) AS registered_count,
  COUNT(CASE
    WHEN first_order_id IS NULL
    THEN customer_id
    END) AS no_sale,
  COUNT(CASE
    WHEN first_order_date - registration_date <  INTERVAL '7' day
    THEN customer_id
    END) AS first_week,
  COUNT(CASE
    WHEN first_order_date - registration_date >= INTERVAL '7' day
     AND first_order_date - registration_date <  INTERVAL '14' day
    THEN customer_id
    END) AS second_week,
  COUNT(CASE
    WHEN first_order_date - registration_date >= INTERVAL '14' day
    THEN customer_id
    END) AS after_second_week
FROM customers
GROUP BY DATE_TRUNC('week', registration_date)
ORDER BY DATE_TRUNC('week', registration_date);
As usual, we used the DATE_TRUNC() function to extract the year and week of registration. Note that we also used the COUNT(CASE WHEN...) technique – this way, we could include multiple metrics in a single report.

Exercise
Create a conversion chart for monthly registration cohorts. Show the following columns:

month
registered_count
no_sale
three_days – the number of customers who made a purchase within 3 days from registration.
first_week – the number of customers who made a purchase during the first week but not within the first three days.
after_first_week – the number of customers who made a purchase after the 7th day.
Order the results by month. Be careful the case of column names and spaces in them.

SELECT
  DATE_TRUNC('month', registration_date) AS month,
  COUNT(*) AS registered_count,
  COUNT(CASE
    WHEN first_order_id IS NULL
    THEN customer_id
    END) AS no_sale,
  COUNT(CASE
    WHEN first_order_date - registration_date < INTERVAL '3' day
         THEN customer_id
    END) AS three_days,
  COUNT(CASE
    WHEN first_order_date - registration_date >=  INTERVAL '3' day
       AND first_order_date - registration_date <  INTERVAL '7' day
    THEN customer_id
    END) AS first_week,
  
  COUNT(CASE
    WHEN first_order_date - registration_date >= INTERVAL '7' day
    THEN customer_id
    END) AS after_first_week
FROM customers
GROUP BY DATE_TRUNC('month', registration_date)
ORDER BY DATE_TRUNC('month', registration_date);

![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/2b1180c5-9255-4244-91cb-3c8c3f263afb)

## Conversion rate report – exercise
Nice work! Time for an additional exercise.

Exercise
Create a conversion chart for monthly registration cohorts from 2017 for each channel. Show the following columns:

month – The registration month.
channel_name – The registration channel.
registered_count – The number of users registered.
no_sale – The number of users who never made a purchase.
one_week – The number of users who made a purchase within 7 days.
after_one_week – The numbers of users who made a purchase after the first week.
Order the results by month.

SELECT
  DATE_TRUNC('month', registration_date) AS month,
  channel_name,
  COUNT(*) AS registered_count,
  COUNT(CASE
    WHEN first_order_id IS NULL
    THEN customer_id
    END) AS no_sale,
  COUNT(CASE
    WHEN first_order_date - registration_date <  INTERVAL '7' day
    THEN customer_id
    END) AS one_week,
  COUNT(CASE
    WHEN first_order_date - registration_date >= INTERVAL '7' day
    THEN customer_id
    END) AS after_one_week
FROM customers cu
JOIN channels ch
ON ch.id = cu.channel_id
WHERE DATE_TRUNC('year', registration_date) = '2017-01-01'
GROUP BY DATE_TRUNC('month', registration_date),channel_name
ORDER BY DATE_TRUNC('month', registration_date),channel_name;
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/8bb35e1f-007d-4c52-969d-b2c8985bd343)
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/a0a1f7a5-981e-40c5-b7bf-c7aeaf1dbc22)

## Summary
Well done! It's time to wrap things up. Let's do a quick summary of what we've learned:

Conversion rate is the count of customers that performed a specific desired action divided by the count of all customers.

To calculate the ratio of all customers who ever placed an order to all registered customers, use:
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/af9b923e-90b4-48b3-a1e1-9466ad7ce14a)
**COUNT(*)::numeric is required to change it to float division. Otherwise,  it does integer division.**
<!SELECT
  ROUND(COUNT(first_order_id) / COUNT(*)::numeric, 2) AS conversion_rate
FROM customers;>

To show conversion rates (as percentages) in weekly registration cohorts, use the DATE_TRUNC() function:
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/6bf991e2-f369-4692-8cce-f041387960b8)
**::numeric is not needed if we use 100.0 as '.0' denotes its a float type**
<!SELECT
  DATE_TRUNC('week', registration_date) AS week,
  ROUND(COUNT(first_order_id) * 100.0 / COUNT(*), 2) AS conversion_rate
FROM customers
GROUP BY DATE_TRUNC('week', registration_date)
ORDER BY DATE_TRUNC('week', registration_date);>

To calculate the time from registration to first order, use:
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/5dcb22b2-22d9-4a9a-b287-afa403c83d0f)
<!SELECT
  customer_id,
  first_order_date - registration_date AS days_to_first_order
FROM customers;>

To create a conversion chart, use multiple COUNT(CASE WHEN...) instances:
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/6fb5c531-e5ac-4960-8dea-e02fa39da52b)
<!--SELECT
  DATE_TRUNC('week', registration_date) AS week,
  ...
  COUNT(CASE
    WHEN first_order_date - registration_date <  INTERVAL '7' day
    THEN customer_id
    END) AS one_week,
  COUNT(CASE
    WHEN first_order_date - registration_date >= INTERVAL '7' day
     AND first_order_date - registration_date <  INTERVAL '14' day
    THEN customer_id
    END) AS two_weeks,
  ...
FROM customers
GROUP BY DATE_TRUNC('week', registration_date)
ORDER BY DATE_TRUNC('week', registration_date);-->

**Exercise 1:** 
Find the global average time that passed between the registration and the first order. Name the column avg_time_to_first_order.

SELECT
	AVG(first_order_date-registration_date) avg_time_to_first_order
FROM customers

**Exercise 2:**

Create a report with conversion rates in weekly cohorts. Show the following columns: week, and conversion_rate. Show the conversion rates as percentages, rounded to two decimal places. Order the results by year and week.
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/159918a1-144a-4f4f-ab64-854ac0ff7990)

# Customer activity 

First, we'll look at active customers. We'll learn how to count the number of active customers – in total and in registration cohorts. Then we'll determine the average value of customer orders. Finally, we'll define "good customers" and see if they have any common characteristics that we can use in our marketing campaigns.

### The orders table
Before we dive into our new report types, let's take a look at the tables we'll query. First, there's the orders table. It keeps track of all orders placed in our e-store. Let's examine some of its rows.

The customers table – recap
Good! The other table we'll need in this part is customers. We've used it before, but this time we'll especially need the last_order_date column.

Exercise
Count the number of customers who placed their most recent order in 2018. Name the column: count_2018_customers.

SELECT COUNT(*) AS count_2018_customers

FROM customers

WHERE DATE_TRUNC('year',last_order_date) ='2018-01-01'

### Number of active customers
Good! Now let's dig into some customer activity analysis. 

- The number of active customers is an important metric in any business, especially in businesses offering their products or services online.
- Sometimes it's easy to say who is an active customer: for a subscription service, someone with an active subscription is an active user, for a mobile app someone who has the app installed and uses it once a week is an active user, etc.
- For an e-store, the definition of an "active customer" is not as straightforward. There are some customers who place orders every now and then, but there are also some who haven't been active for a long time.
- In our online supermarket, regular customers typically place one order a week, but we'll define "active customers" as all customers who've placed an order within the last 30 days.

  Let's count how many active customers we have:
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/5da0c5aa-9f70-483d-ab9d-d5c3de235ec2)

SELECT COUNT(*) AS active_customers
FROM customers
WHERE CURRENT_DATE - last_order_date < INTERVAL '30' day;

In the WHERE clause, we created an interval: we subtracted last_order_date from CURRENT_DATE. This lets us check if the given customer's most recent order was within the last 30 days (by comparing the difference to the INTERVAL '30' day).

**Exercise** 
Find the number of active customers in each country. Show two columns: country and active_customers (number of those who have placed an order within the last 30 days). Do you see any major differences between countries?

![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/a8f87627-f928-4e37-8368-b5d60446e33d)
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/52eb7973-4d00-4d35-9742-1faaae6d1652)

### Number of active customers in time-period cohorts
Perfect! The general number of active customers is a good start, but now we'd like **to see how many active customers there are in each weekly registration cohort**. We can then compare these numbers against the registration campaigns we ran each week and see which marketing activities yielded the greatest number of active customers. Take a look:

![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/a8869fc1-275d-4a47-b7d3-d931934133f2)
SELECT
  DATE_TRUNC('week', registration_date) AS week,
  COUNT(*) AS active_customers
FROM customers
WHERE CURRENT_DATE - last_order_date < INTERVAL '30' day
GROUP BY DATE_TRUNC('week', registration_date)
ORDER BY DATE_TRUNC('week', registration_date);

Compared to the previous report, we added one DATE_TRUNC() invocation (to extract years and weeks) in three places (in SELECT, GROUP BY, and ORDER BY clauses). This shows us the number of active customers in each cohort.

**Exercise** 
Find the number of active customers (those who placed an order within the last 30 days) that registered in 2017. Use monthly registration cohorts and show two columns: month and active_customers. Order the rows by month.

![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/65b6f500-f95f-47d4-8b06-90b67b524be5)

**Exercise** 
Find the number of active customers in quarterly registration cohorts. Active customers are customers who've made a purchase in the last 14 days. Show two columns: quarter, and active_customers. Order the rows by quarter.

![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/15aa3245-c8d1-401c-a4f2-023847ba8e0c)


### Average order value 
Great! Knowing which customers are active is important, but **it's equally important to understand how much revenue our customers generate**. We now want to know the average order value for each weekly registration cohort. Check it out:

![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/6dc11ef1-c82a-45c3-98f8-539b7b3cfc63)
SELECT
  DATE_TRUNC('week', registration_date) AS week,
  AVG(total_amount) AS average_order_value
FROM customers c
JOIN orders o
  ON c.customer_id = o.customer_id
GROUP BY DATE_TRUNC('week', registration_date)
ORDER BY DATE_TRUNC('week', registration_date);

In the query above, we need data from two tables: customers (to get registration dates) and orders (to get the total amount for each order). Then, we used AVG(total_amount) to calculate the average order value.

Exercise
What was the average order value for weekly registration cohorts from 2017 for orders shipped to Germany? Show two columns: week and average_order_value, and order the results by week in descending order.
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/8efbc16e-cfe2-402b-b328-025c450374b9)

### Average order values
Well done! Now, let's analyze our e-store's "good customers." Each business will use its own definition of a good customer that is based on its business model. **In our e-store, we'll define a "good customer" as a customer whose average order value is above the general average order value for all customers.** Analyzing such customers may help us understand what makes customers spend more. This, in turn, can help us decide which marketing campaigns we should focus on.

First, we need to see the average order value for each individual customer:

![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/7c17b8f1-1916-4b15-9886-abc334c5ef5d)

SELECT
  c.customer_id,
  AVG(total_amount) AS avg_order_value
FROM customers c
JOIN orders o
  ON c.customer_id = o.customer_id
GROUP BY c.customer_id
ORDER BY AVG(total_amount);

The query is quite straightforward: we join the tables customers and orders so that we can calculate AVG(total_amount) and get some additional information about the customer.

**Exercise** 
Run the query from the template and analyze the results. What do you think about them? Look at the differences between customers. Are their average values similar or completely different? Why do you think this is?

### Average order value per customer
Good job! In the previous report, we could see the average order value for each customer. We're looking for good customers, so we want to know the general average order values, aggregated over all customers. Here's the query:

WITH average_per_customer AS (
  SELECT
    c.customer_id,
    AVG(total_amount) AS avg_order_value
  FROM customers c
  JOIN orders o
    ON c.customer_id = o.customer_id
  GROUP BY c.customer_id
)

SELECT AVG(avg_order_value) AS global_avg_order_value
FROM average_per_customer;
![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/f6e6564a-1e09-4d5f-bc5a-e51eac8f8b00)

In the query above, we use a concept known as **a CTE (Common Table Expression)**. It works like a temporary table and has its own name (average_per_customer in this case) which we can use later in the outer query. 

A CTE is introduced before the main query in the following way: WITH cte_name AS (). Inside the parentheses, we write the query we need. Remember to provide column aliases with the keyword AS so that you can refer to them in the outer query. In this case, the inner query is very similar to our previous report: for each customer, it calculates the average order value for this customer.

After the closing parenthesis, we write our outer query. Note that we put the CTE's name in the FROM clause. In this case, we simply calculate the average order value based on all customer-level averages calculated in the CTE.

The result of the query is 1636.622.

**Exercise** 
Find each country's average order value per customer. Show two columns: country and avg_order_value. Sort the results by average order value, in ascending order.

First, add the country column in the CTE. Then, use the column country in the outer query's GROUP BY.

![image](https://github.com/mythilyram/Customer-behavior-analysis/assets/123518126/0a5a2f75-7b25-4de3-bfb0-f71ab2095a1f)

**Exercise** 
Find out the average number of orders placed in the last 180 days by customers who have been active (made a purchase) in the last 30 days. Name the column avg_order_count.

In the Common Table Expression determine the number of orders placed in the last 180 days by each active customer. Then, in the outer query determine the average.
