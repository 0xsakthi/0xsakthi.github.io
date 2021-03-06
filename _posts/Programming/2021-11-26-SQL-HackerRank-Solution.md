---
title: "Sql-HackerRank - Solution"
classes: wide
tag: 
  - "mysql"
  - "Hackerrank"
  - "pb"
header:
  teaser: /assets/images/cpp.png
ribbon: Blue
description: "CheatSheet - Cpp"
categories:
  - pg
---
# Revising the Select Query I
*Query all columns for all American cities in the CITY table with populations larger than 100000. The CountryCode for America is USA.*

*The CITY table is described as follows:*
![image](https://user-images.githubusercontent.com/82046832/143572959-2245896d-59c7-40bd-bf0d-d60c956f6384.png)

**Query**
``` sql
select * from CITY where POPULATION>100000 and CountryCode="USA";
```
# Revising the Select Query II
*Query the NAME field for all American cities in the CITY table with populations larger than 120000. The CountryCode for America is USA.*

*The CITY table is described as follows:*
![image](https://user-images.githubusercontent.com/82046832/143573731-8c816566-c57a-4644-82b6-5de2ce2f5563.png)

**Query**
```sql
select NAME from CITY where POPULATION>120000 AND COUNTRYCODE="USA"
```
# SELECT ALL
*Query all columns (attributes) for every row in the CITY table.*

*The CITY table is described as follows:*
![image](https://user-images.githubusercontent.com/82046832/143574118-907aeb4f-7954-4f0f-af3e-3bcc2bb46407.png)
```sql
select * from CITY;
```
# Select By ID
![image](https://user-images.githubusercontent.com/82046832/143574452-ec1fb1e7-3bc7-43eb-8ba4-0a53822cc952.png)
```sql
select * from CITY where ID=1661;
```
# Japanese Cities' Attributes
*Query all attributes of every Japanese city in the CITY table. The COUNTRYCODE for Japan is JPN.*

*The CITY table is described as follows:*

![image](https://user-images.githubusercontent.com/82046832/143574659-4ff44177-b417-4a79-b7d7-2dd1adc3a74c.png)

```sql
select * from CITY where COUNTRYCODE="JPN";
```
# Japanese Cities' Names
*Query the names of all the Japanese cities in the CITY table. The COUNTRYCODE for Japan is JPN.*

*The CITY table is described as follows:*
![image](https://user-images.githubusercontent.com/82046832/143574880-dcdefe2d-2385-4ff6-9af2-3665d65b2348.png)
```sql
select Name from CITY where COUNTRYCODE="JPN"
```
# Weather Observation Station 1
*Query a list of CITY and STATE from the STATION table.*

*The STATION table is described as follows:*
![image](https://user-images.githubusercontent.com/82046832/143575152-b7823e85-2f72-4020-b822-149c8a672683.png)

```sql
select CITY,STATE from STATION;
```
# Weather Observation Station 3
*Query a list of CITY names from STATION for cities that have an even ID number. Print the results in any order, but exclude duplicates from the answer.*

*The STATION table is described as follows:*
![image](https://user-images.githubusercontent.com/82046832/143676538-35a46176-ed43-4cdf-be6e-f903e0bad5c0.png)

```sql
select distinct CITY from STATION where mod(ID,2)=0 order by CITY;
```
# Weather Observation Station 4
*Find the difference between the total number of CITY entries in the table and the number of distinct CITY entries in the table.*

*The STATION table is described as follows:*
![image](https://user-images.githubusercontent.com/82046832/143676584-db0e667a-49ca-4aca-8ff3-b06b7298d108.png)

```sql
select count(CITY) - count(distinct(CITY)) from STATION;
```
# Weather Observation Station 6

*Query the list of CITY names starting with vowels (i.e., a, e, i, o, or u) from STATION. Your result cannot contain duplicates.*

*The STATION table is described as follows:*
![image](https://user-images.githubusercontent.com/82046832/144748429-ef81c79f-c484-4616-ae84-9ce16c680864.png)

```sql
select distinct CITY from STATION where CITY like "a%" or CITY like "e%" or CITY like "i%" or CITY like "o%" or CITY like "u%"; 
```
# Weather Observation Station 7
*Query the list of CITY names ending with vowels (a, e, i, o, u) from STATION. Your result cannot contain duplicates.*
![image](https://user-images.githubusercontent.com/82046832/144748525-393d15f7-3859-4512-8567-825f0ee75351.png)
```sql
select distinct CITY from STATION where CITY like "%a" or CITY like "%e" or CITY like "%i" or CITY like "%o" or CITY like "%u";
```
# Weather Observation Station 8
*Query the list of CITY names from STATION which have vowels (i.e., a, e, i, o, and u) as both their first and last characters. Your result cannot contain duplicates.*
![image](https://user-images.githubusercontent.com/82046832/144748780-6e4e9047-9701-40cb-8e64-ced23a1a4864.png)
```sql
select distinct CITY from STATION where (CITY like "a%" or  CITY like "e%" or  CITY like "i%" or  CITY like "o%" or  CITY like "u%") and  (CITY like "%a" or  CITY like "%e" or  CITY like "%i" or  CITY like "%o" or  CITY like "%u")
```
# Weather Observation Station 9
*Query the list of CITY names from STATION that do not start with vowels. Your result cannot contain duplicates*
```sql
select distinct CITY from STATION where not (city like "a%" or city like "e%" or city like "i%" or city like "o%" or city like "u%");
```
# Weather Observation Station 10
*Query the list of CITY names from STATION that do not end with vowels. Your result cannot contain duplicates.*
```sql
select distinct CITY from STATION where not(CITY like "%a" or CITY like "%e" or CITY like "%i" or CITY like "%o" or CITY like "%u");
```
# Weather Observation Station 11
*Query the list of CITY names from STATION that either do not start with vowels or do not end with vowels. Your result cannot contain duplicates.*

![image](https://user-images.githubusercontent.com/82046832/144753865-c3e67899-c3ad-4cbe-88c0-e32c8baf1de0.png)

```sql
select distinct CITY from STATION where (not (CITY like "a%" or CITY like "e%" or CITY like "i%" or CITY like "o%" or CITY like "u%")) or (not (CITY like "%a" or CITY like "%e" or CITY like "%i" or CITY like "%o" or CITY like "%u"));
```
# Weather Observation Station 12
*Query the list of CITY names from STATION that do not start with vowels and do not end with vowels. Your result cannot contain duplicates.*

```sql
select distinct CITY from STATION where (not (CITY like "a%" or CITY like "e%" or CITY like "i%" or CITY like "o%" or CITY like "u%")) and (not (CITY like "%a" or CITY like "%e" or CITY like "%i" or CITY like "%o" or CITY like "%u"));
```
# Employee Salaries
![image](https://user-images.githubusercontent.com/82046832/144755400-c9f3c49b-daa8-4ba9-88a2-981d8c336f0d.png)
![image](https://user-images.githubusercontent.com/82046832/144755408-830e345e-0158-45b0-ae31-f47143cf3955.png)
 ```sql
  select name from Employee where salary>2000 and months<10;
 ```
 # 
