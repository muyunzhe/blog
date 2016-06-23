---
title: 'Hive Date Type:TIMESTAMP'
comments: true
categories: technology
tags: database
date: 2016-05-31 18:06:29
---
The underlying Impala data type for date and time data is TIMESTAMP, which has both a date and a time portion.
<!--more-->
A data type used in CREATE TABLE and ALTER TABLE statements, representing a point in time.
## 时间范围
hive里面TIMESTAMP Data Type，它的范围从1400-01-01 to 9999-12-31。
## INTERVAL expressions
You can perform date arithmetic by adding or subtracting a specified number of time units, using the INTERVAL keyword and the + and - operators or date_add() and date_sub() functions. You can specify units as YEAR[S], MONTH[S], WEEK[S], DAY[S], HOUR[S], MINUTE[S], SECOND[S], MILLISECOND[S], MICROSECOND[S], and NANOSECOND[S]. You can only specify one time unit in each interval expression, for example INTERVAL 3 DAYS or INTERVAL 25 HOURS, but you can produce any granularity by adding together successive INTERVAL values, such as timestamp_value + INTERVAL 3 WEEKS - INTERVAL 1 DAY + INTERVAL 10 MICROSECONDS.
For example:
```
select now() + interval 1 day;
select date_sub(now(), interval 5 minutes);
insert into auction_details
  select auction_id, auction_start_time, auction_start_time + interval 2 days + interval 12 hours
  from new_auctions;
  ```
## Conversions
 Impala automatically converts STRING literals of the correct format into TIMESTAMP values. Timestamp values are accepted in the format YYYY-MM-DD HH:MM:SS.sssssssss, and can consist of just the date, or just the time, with or without the fractional second portion. For example, you can specify TIMESTAMP values such as '1966-07-30', '08:30:00', or '1985-09-25 17:45:30.005'. You can cast an integer or floating-point value N to TIMESTAMP, producing a value that is N seconds past the start of the epoch date

 ## Partitioning
 Although you cannot use a TIMESTAMP column as a partition key, you can extract the individual years, months, days, hours, and so on and partition based on those columns. Because the partition key column values are represented in HDFS directory names, rather than as fields in the data files themselves, you can also keep the original TIMESTAMP values if desired, without duplicating data or wasting storage space. See Partition Key Columns for more details on partitioning with date and time values.
 Examples:
 ```
 select cast('1966-07-30' as timestamp);
select cast('1985-09-25 17:45:30.005' as timestamp);
select cast('08:30:00' as timestamp);
select hour('1970-01-01 15:30:00');         -- Succeeds, returns 15.
select hour('1970-01-01 15:30');            -- Returns NULL because seconds field required.
select hour('1970-01-01 27:30:00');         -- Returns NULL because hour value out of range.
select dayofweek('2004-06-13');             -- Returns 1, representing Sunday.
select dayname('2004-06-13');               -- Returns 'Sunday'.
select date_add('2004-06-13', 365);         -- Returns 2005-06-13 with zeros for hh:mm:ss fields.
select day('2004-06-13');                   -- Returns 13.
select datediff('1989-12-31','1984-09-01'); -- How many days between these 2 dates?
select now();                               -- Returns current date and time in UTC timezone.

create table dates_and_times (t timestamp);
insert into dates_and_times values
  ('1966-07-30'), ('1985-09-25 17:45:30.005'), ('08:30:00'), (now());
  ```
