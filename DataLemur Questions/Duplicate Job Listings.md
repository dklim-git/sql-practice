[TOC]

# Duplicate Job Listings

Assume you are given the table below that shows job postings for all companies on the LinkedIn platform. Write a query to get the number of companies that have posted duplicate job listings. Duplicate job listings refer to two jobs at the same company with the same title and description.

`job_listings` table:

| Column Name |  Type   |
| :---------: | :-----: |
|   job_id    | integer |
| company_id  | integer |
|    title    | string  |
| description | string  |

`job_listings` example input:

| job_id | company_id |      title       |                         description                          |
| :----: | :--------: | :--------------: | :----------------------------------------------------------: |
|  248   |    827     | Business Analyst | Business analyst evaluates past and current business data with the primary goal  of improving decision-making processes within organizations. |
|  149   |    845     | Business Analyst | Business analyst evaluates past and current business data with the primary goal  of improving decision-making processes within organizations. |
|  945   |    345     |   Data Analyst   | Data analyst reviews data to identify key insights into a business's customers and ways the data can be used to solve problems. |
|  164   |    345     |   Data Analyst   | Data analyst reviews data to identify key insights into a business's customers and ways the data can be used to solve problems. |
|  172   |    244     |  Data Engineer   | Data engineer works in a variety of settings to build systems that collect,  manage, and convert raw data into usable information for data scientists and business analysts to interpret. |

---

### Solution Steps

The prompt asks to find the number of companies with duplicate postings:

```postgresql
select company_id, title, description, count(*)
from job_listings
group by company_id, title, description
having count(*) > 1
```

To find instances where a company has listed the same job listing more than once, grouping by `company_id`, `title`,  and `description` will match exact values from the three columns.  `count(*)` will display the number of times a job listing has appeared.  `having` will filter `count(*)` to only display listings that have been posted more than once.

> **Remember that the where clause cannot be used here because it only filters rows from the original dataset.**

The query above will display all companies that has a duplicate job listing (along with job title and description), so it can be used as a CTE to return the count of rows (final solution):

```postgresql
with repeats as (
	select company_id, title, description, count(*)
	from job_listings
	group by company_id, title, description
	having count(*) > 1
)

select count(*) from repeats
```

`repeats` contains a temporary table that'll output the ID of compaies that have duplicate listings.  Querying the count of rows from this table will reveal the total number of companies with multiple postings.

---

### Notes

In my initial attempt, I tried using an alias for `count(*)`, but PostgreSQL 14 handles aliases differently compared to MySQL:

```postgresql
with repeats as (
	select company_id, title, description, count(*) as dupes
	from job_listings
	group by company_id, title, description
	having dupes > 1
)

select count(*) from repeats
```

You'll receive an error saying:

> column "dupes" does not exist. FYI: in PostgreSQL all identifiers (including column names) that are not double-quoted are folded to lowercase. So if you're using any uppercased letters in the names and want to specify them in SQL-query, please double-quote them first. (LINE: 5)

An alias isn't needed here, but will help with the readability of the query.  I **think** this code would work in MySQL, so I'll leave this here until I discover otherwise.

---

### Alternative Solution

```postgresql
select count(*)
from (
    select company_id, title, description, count(*)
    from job_listings
    group by company_id, title, description
    having count(*) > 1
) as s1
```

Found an accepted solution in the comments section using a subquery.  I would prefer to use a CTE as it'll be more optimized for this prompt and has better readability.  The same principle is used here, querying the count of the table that contains companies that have duplicate postings.

---

### Prompt Details

Prompt can be found on [DataLemur](https://datalemur.com/prompts/duplicate-job-listings), is easy rated, and found in Linkedin interviews.
