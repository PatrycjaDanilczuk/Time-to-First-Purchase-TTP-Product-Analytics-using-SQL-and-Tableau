# Time to First Purchase (TTP) - Project Overview

## 1. Defining business problem
Identify how much time it takes for a user to make a purchase on your website. Show the duration dynamic daily.

## 2. Dataset description

Use  **raw_events table** hosted in **BigQuery** project. Data in raw_events table captures a lot of events from users based on their timestamps.

The dataset contains 31 columns and 4.295.584 rows.


<details>

**<summary>Click to expand raw_events table schema</summary>**

### raw_events schema

| Field name | Type | Mode |
|---------------|-----------|-----------|
| event_date | STRING | NULLABLE |
| event_timestamp | INTEGER | NULLABLE |
| event_name |	 STRING | NULLABLE |
| event_value_in_usd| FLOAT| NULLABLE|
| user_id | STRING| NULLABLE|
| user_pseudo_id| STRING| NULLABLE|
| user_first_touch_timestamp| INTEGER| NULLABLE|
| category| STRING| NULLABLE |
| mobile_model_name | STRING| NULLABLE |
| mobile_brand_name |STRING | NULLABLE |
| operating_system | STRING | NULLABLE |
| language | STRING | NULLABLE |
| is_limited_ad_tracking| STRING | NULLABLE |
| browser | STRING | NULLABLE |
| browser_version | STRING | NULLABLE |
| country | STRING | NULLABLE |
| medium | STRING | NULLABLE |
| name | STRING | NULLABLE |
| traffic_source | STRING | NULLABLE |
| platform | STRING | NULLABLE |
| total_item_quantity | INTEGER | NULLABLE |
| purchase_revenue_in_usd | FLOAT | NULLABLE |
| refund_value_in_usd | FLOAT | NULLABLE |
| shipping_value_in_usd | FLOAT | NULLABLE |
| tax_value_in_usd | FLOAT | NULLABLE |
| transaction_id	 | STRING | NULLABLE |
| page_title | STRING | NULLABLE |
| page_location	 | STRING | NULLABLE |
| source | STRING | NULLABLE |
| page_referrer	 | STRING | NULLABLE |
| campaign | STRING | NULLABLE |

</details>


## 3.	Defining Time to First Purchase (TTP)

### Option 1: Time from First Visit to Purchase in Calendar Time (TTP Calendar)

This approach measures the total time in days (or hours) from the user's very first visit to the website until they make their first purchase. 
It includes all the days between the first visit and purchase, even if the user was inactive during some of those days.

**Important note:** event_timestamp in raw_events table captures time in UTC (Coordinated Universal Time), which is a standard used to establish timezones worldwide.
To capture if a user made a purchase for example the same day as the day of his first visit to a site, the UTC time of events should be converted to the user's local time 
(we need the same day for the user, not the same day according to UTC).

### Option 2. Sum of Active Time on Website - Session Time (TTP Active)

This method measures the total time users actively spend on a site across all sessions before making their first purchase. 
Instead of including the entire calendar duration, this focuses only on the time they were actually engaged.

**Important note:** there are no session identifiers in the raw_events table, therefore it is necessary to model sessions, calculate session duration and sum sessions duration per user.

#### User session model applied:

User session typically represents a continuous period of user activity on a website or application. We’ll consider a session as a sequence of events by the same user with relatively short gaps between them. 
A single user can come to a website on multiple days and using multiple devices. User’s activity on different devices should be considered as separate sessions. If the time difference between two events is less than a certain threshold, 
we consider them as part of the same session. If the time difference exceeds the threshold, we start a new session. As a benchmark, Google Analytics defaults to 30 minutes as a threshold for session gap.

For this project User session has been defined as a sequence of events per user per category (desktop, mobile, tablet). Since one user can come to a website using different devices, events on different devices are 
considered as parts of separate sessions. Gap in session aggregating events to separate sessions = 30 minutes.

For sequence of events per user per category:

- if the difference between events is less than or equals 30 minutes, we consider them as part of the same session,

- if the difference between events is longer than 30 minutes, this will be considered as a gap in session, starting a new session.

Defined user session model implies, that one user can have multiple sessions on multiple devices.

## 4.	Preparing and processing data

### Creating a table of unique users with data relevant for the analysis from raw_events table using SQL

Out of over 4 mln events from raw_events table created a table of 4.330 records of unique users with relevant data from their first visit to a site (first event of the user by timestamp) to their first purchase, in the given period of time. 

Table from this query has been used for analysis in the next steps of the project. To view the code go to the uploaded SQL file [Product Analyst Project SQL code](https://github.com/TuringCollegeSubmissions/pdanil-PA.1.3/blob/main/Product%20Analyst%20Project%20SQL%20code)

<details>

**<summary>Click to expand the SQL code logic details</summary>**

### SQL code logic explanation:

1. Creating a table mapping countries to their time zones, to convert UTC timestamp to local time for each user
2. Checking duplicates by comparing SELECT COUNT() and SELECT DISTINCT(COUNT()) - no duplicates in raw_events table
3. Selecting columns for analysis: user_pseudo_id, category, medium, browser, country, purchase_revenue_in_usd, event_date, event_timestamp, event_name
4. Converting event_timestamp into datetime in microseconds; converting UTC into local time (evant_time_local)
5. Identifying time of first visit to a site - MIN(event_time_local)
6. Adding purchase flag for further transformations
7. Assigning users who made a purchase and number of purchase to get the first purchase events only
8. Identifying the first purchase time
9. Selecting all events from first visit to first purchase for converted users
10. Assigning sessions per user
11. Calculating sessions duration
12. Calculating active time to first purchase (sum of sessions duration per user)
13. Calculating days to first purchase
14. Assigning Segments of TTP and calculating other data relevant for the analysis
15. Selecting unique users with relevant data per user

</details>

## 5.	Analyzing data
Using Tableau for further analysis
Preparing Dashboard with relevant visuals. The link to the dashboard can be found here: [TTP Dashboard](https://public.tableau.com/app/profile/pat.dan/viz/TTPDashboard/DOverview)

## 6.	Communicating analysis results and providing actionable insights
Preparing presentation for business. The presentation had been uploaded in the PDF file: [Product Analyst Presentation for Business](https://github.com/TuringCollegeSubmissions/pdanil-PA.1.3/blob/main/Product%20Analyst%20prsentation%20for%20business.pdf)

Use Tableau [TTP Dashboard](https://public.tableau.com/app/profile/pat.dan/viz/TTPDashboard/DOverview) for more details.
