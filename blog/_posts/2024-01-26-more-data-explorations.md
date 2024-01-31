---
layout: posts
title:  "Data exploration using SQL"
date:   2024-01-26 12:00:00 +0000
categories: SQL
highlight_home: true
tags: SQL Jupyter
description: More exploration Toastmasters Distinguished Club Program data
header:
 overlay_image: /assets/images/encrypted_data.png
 teaser: /assets/images/sql-database-generic-svgrepo-com.svg
excerpt: How did the Covid-19 pandemic affect Toastmasters Clubs?
 
---
# Further exploration of Toastmasters Club performance data
<!-- more -->
I've [imported fifteen years of Toastmasters Club Performance data into a single database table](/projects/sql/import-explore-cleanup.html#combining-all-15-years-into-one-convenient-table). What does the data show for the past 4 years, during the Covid-19 pandemic?

I'll start by creating a new Jupyter Notebook, importing the sqlite3 library, enabling the %sql and %%sql magic commands, connecting to the database file, and setting the display limit to "None" to display more than the default 10 results:
```
[1]: import sqlite3
     %load_ext sql
     %sql sqlite:///all_years.sqlite
     %config SqlMagic.displaylimit = None
```
## Listing tables in a SQLite database
For the first query, I'll refresh my memory of the names of the tables in the database:
```
[2]: %sql SELECT name FROM sqlite_master WHERE type='table';
```
<table><thead><tr><th>name</th></tr></thead>
<tbody><hr><tr><td>tbl_0809</td></tr><tr><td>tbl_1011</td></tr><tr><td> <em>[12 rows not shown]</em> </td></tr>
<tr><td>tbl_2223</td></tr><tr><td>tbl_all</td></tr></tbody></table>

## Counting Clubs, Active Members, and Members per Club
Next, I'll pick up where I left off in the previous project - listing the total number of clubs, total number of members, and average members per club at the end of each program year:
```
[3]: %%sql
      SELECT Date, printf("%,d", COUNT("Club Number")) AS "Total Clubs", 
                   printf("%,d", SUM("Active Members")) AS "Total Members", 
                   printf("%.2f", AVG("Active Members")) AS "Average Members/Club"
      FROM tbl_all WHERE Month = 6 GROUP BY Date
```

| Date | Total Clubs | Total Members | Average Members/Club |
| ---- | ---- | ---- | ---- |
| 2009-06-30 | 12,024 | 239,287 | 19.90 |
| 2010-06-30 | 12,514 | 251,318 | 20.08 |
| 2011-06-30 | 13,139 | 263,954 | 20.09 |
| 2012-06-30 | 13,653 | 271,962 | 19.92 |
| 2013-06-30 | 14,850 | 284,200 | 19.14 |
| 2014-06-30 | 15,393 | 303,728 | 19.73 |
| 2015-06-30 | 16,119 | 320,277 | 19.87 |
| 2016-06-30 | 16,771 | 333,752 | 19.90 |
| 2017-06-30 | 17,472 | 338,050 | 19.35 |
| 2018-06-30 | 17,905 | 338,442 | 18.90 |
| 2019-06-30 | 18,162 | 340,427 | 18.74 |
| 2020-06-30 | 18,104 | 300,921 | 16.62 |
| 2021-06-30 | 18,521 | 288,217 | 15.56 |
| 2022-06-30 | 16,828 | 256,172 | 15.22 |
| 2023-06-30 | 16,014 | 255,257 | 15.94 |

The impact of the Covid-19 epidemic is plain to see in the Total Members count and Average Members per Club. Total membership dropped by nearly 40,000 between June 2019 and June 2020, with an average loss of more than 2 members per club. 

The number of clubs dropped by only 58 from 2019 to 2020, and actually rose by over 400 in the next year. However, this increase hid the fact that a lot of clubs were having trouble recruiting and retaining members in late 2020 and through 2021. Clubs with fewer than 8 members are at risk of suspension after one full 6-month renewal period (October-March or April-September), and may be closed permanently if they do not bring themselves back to at least 8 members.

## Covid-19's impact on Distinguished Club counts
Toastmasters maintains the Club Performance dashboards so that club and district leaders can track their progress in the Distinguished Club Program (a.k.a. DCP). Clubs are recognized as Distinguished, Select Distinguished, or President's Distinguished by achieving goals, including having members complete levels in the educational program, recruiting new members, filing club paperwork on time, and having the club officers attending training twice a year.

There are 10 goals in the DCP. The requirements for the three status levels are:
* <strong>Distinguished</strong>: earn 5-6 goals
* <strong>Select Distinguished</strong>: earn 7-8 goals
* <strong>President's Distinguished</strong>: earn 9-10 goals  

In addition, clubs need to finish the year with 20 members, or have an increase of at least 3 members from the membership base count at the start of the program year.

How did Covid-19 affect the year-end counts of Distinguished Clubs?
### Using GROUP BY to count Club Distinguished Status 
To answer this, I did a count of clubs and used the SQL <code>GROUP BY</code> clause to create subtotals for Program Year and Club Distinguished Status:
```
[4]: %%sql 
     SELECT "Program Year", "Club Distinguished Status", COUNT("Club Number") 
     FROM tbl_all
     WHERE Month = 6
     GROUP BY "Program Year", "Club Distinguished Status";
```
<table><thead><tr><th>Program Year</th><th>Club Distinguished Status</th><th>COUNT("Club Number")</th></tr></thead>
<tbody><hr>
<tr><td>2008-2009</td><td>None</td><td>6532</td></tr>
<tr><td>2008-2009</td><td>D</td><td>1683</td></tr>
<tr><td>2008-2009</td><td>P</td><td>2372</td></tr>
<tr><td>2008-2009</td><td>S</td><td>1437</td></tr>
<tr><td>2009-2010</td><td>None</td><td>6547</td></tr>
<tr><td>2009-2010</td><td>D</td><td>1657</td></tr>
</tbody>
</table>
*[54 rows not shown]*

This works, but displays each status on a separate row - 60 rows in all. It would make a lot more sense as a 2-D table, like a pivot table in Microsoft Excel or Google Sheets.

## Pivoting the Distinguished Club data
### Using a correlated subquery
The easiest way to display data in columns is to use a **correlated subquery**, which is a subquery inside an outer query. A correlated subquery runs once for each row of the outer query, similar to a nested loop in traditional computer programming. For example:
```
[5]: %%sql 
     SELECT 
       "Program Year", 
       (SELECT COUNT("Club Number") FROM tbl_all as inner WHERE Month = 6 AND "Club Distinguished Status" IS NULL
        AND inner."Program Year" = outer."Program Year") AS "Not Distinguished",
       (SELECT COUNT("Club Number") FROM tbl_all as inner WHERE Month = 6 AND "Club Distinguished Status" = 'D'
        AND inner."Program Year" = outer."Program Year") AS Distinguished,
       COUNT("Club Number")
     FROM tbl_all as outer WHERE Month = 6
     GROUP BY "Program Year";
```
<table><thead>
  <tr><th>Program Year</th><th>Not Distinguished</th>
      <th>Distinguished</th><th>COUNT("Club Number")</th></tr>
</thead><hr>
<tbody>
<tr><td>2008-2009</td><td>6532</td><td>1683</td><td>12024</td></tr>
<tr><td>2009-2010</td><td>6547</td><td>1657</td><td>12514</td></tr>
<tr><td>2010-2011</td><td>6872</td><td>1582</td><td>13139</td></tr>
</tbody></table>
*[12 rows not shown]*  

This is the easiest way, but it's not the best way. Each column requires a full pass through the <code>tbl_all</code> table, so even this incomplete example, with just two correlated subquery columns, required nearly 30 seconds to complete. Each additional column will require another correlated subquery, increasing the runtime. Adding columns to show the percentage breakdown of each Distinguished status would make the full query very complex.

### Using a Common Table Expression
A Common Table Expression (CTE) is another form of SQL subquery. It uses the WITH keyword to define a named virtual (in-memory) table that is created before the rest of the query is run:
```
[6]: %%sql
     WITH DistTotals AS (
         SELECT "Program Year", "Club Distinguished Status", COUNT("Club Number") AS N
         FROM tbl_all
         WHERE Month = 6
         GROUP BY "Program Year", "Club Distinguished Status"
     )
     SELECT "Program Year", SUM(N) AS Total,
       (SELECT N FROM DistTotals AS inner WHERE "Club Distinguished Status" IS NULL
         AND inner."Program Year" = outer."Program Year") AS "Not Distinguished",
       (SELECT N FROM DistTotals AS inner WHERE "Club Distinguished Status" = "D"
         AND inner."Program Year" = outer."Program Year") AS Distinguished,
       (SELECT N FROM DistTotals AS inner WHERE "Club Distinguished Status" = "S"
         AND inner."Program Year" = outer."Program Year") AS "Select Distinguished",
       (SELECT N FROM DistTotals AS inner WHERE "Club Distinguished Status" = "P"
         AND inner."Program Year" = outer."Program Year") AS "President's Distinguished" 
     FROM DistTotals AS outer
     GROUP BY "Program Year";
```
The first section of code (<code>WITH DistTotals AS (</code>...) is the Common Table Expression. It uses the same query from [4] above to create a virtual table named <code>DistTotals</code>. This virtual table is then used in correlated subqueries in the rest of the query. The advantage is that instead of each subquery having to run through all 2.7+ million rows of <code>tbl_all</code>, each column only has to run through the 60 rows of <code>DistTotals</code>. As a result, this query with 6 columns takes less than 2 seconds to run, instead of the half minute required by the query with only 4 columns in [5] above.

### Using a nested CTE
To compute the percentages of Not Distinguished, Distinguished, Select Distinguished, and President's Distinguished clubs, I used a **nested CTE**:
```
[7]: %%sql
     WITH DistinguishedCounts AS (
       WITH DistTotals AS (
           SELECT "Program Year", "Club Distinguished Status", COUNT("Club Number") AS N
           FROM tbl_all
           WHERE Month = 6
           GROUP BY "Program Year", "Club Distinguished Status"
       )
       SELECT "Program Year", SUM(N) AS Total,
         (SELECT N FROM DistTotals AS inner WHERE "Club Distinguished Status" IS NULL
           AND inner."Program Year" = outer."Program Year") AS Not_Dist,
         (SELECT N FROM DistTotals AS inner WHERE "Club Distinguished Status" = "D"
           AND inner."Program Year" = outer."Program Year") AS Dist,
         (SELECT N FROM DistTotals AS inner WHERE "Club Distinguished Status" = "S"
           AND inner."Program Year" = outer."Program Year") AS SelectDist,
         (SELECT N FROM DistTotals AS inner WHERE "Club Distinguished Status" = "P"
           AND inner."Program Year" = outer."Program Year") AS PresDist 
       FROM DistTotals AS outer
       GROUP BY "Program Year"
     )
     SELECT 
         "Program Year", Total AS "Total Clubs", 
         Not_Dist AS "Not Dist.",
           PRINTF("%.2f", 100.0*Not_Dist/Total) AS "% Not Dist.",
         Dist AS "Dist.",                   
           PRINTF("%.2f", 100.0*Dist/Total) AS "% Dist.",
         SelectDist AS "Select Dist.",    
           PRINTF("%.2f", 100.0*SelectDist/Total) AS "% Select Dist.",
         PresDist AS "President's Dist.", 
           PRINTF("%.2f", 100.0*PresDist/Total) AS "% Pres. Dist."
         FROM DistinguishedCounts;
```

This query takes the <code>DistTotals</code> CTE from [6] above, and wraps it in another CTE to create a virtual table named <code>DistinguishedCounts</code>. This virtual table includes the count for each status as well as the total number of clubs for each program year, so calculating the percentage of each status for each program year is very simple. The result of this query is:
<table>
<thead><tr>
  <th>Program Year</th><th>Total Clubs</th>
  <th>Not Dist.</th><th>% Not Dist.</th>
  <th>Dist.</th><th>% Dist.</th>
  <th>Select Dist.</th><th>% Select Dist.</th>
  <th>President's Dist.</th><th>% Pres. Dist.</th>
</tr></thead><hr>
<tbody>
<tr><td>2008-2009</td><td>12024</td>
  <td>6532</td><td>54.32</td>
  <td>1683</td><td>14.00</td>
  <td>1437</td><td>11.95</td>
  <td>2372</td><td>19.73</td></tr>
<tr><td>2009-2010</td><td>12514</td>
  <td>6547</td><td>52.32</td>
  <td>1657</td><td>13.24</td>
  <td>1523</td><td>12.17</td>
  <td>2787</td><td>22.27</td></tr>
<tr><td>2010-2011</td><td>13139</td>
  <td>6872</td><td>52.30</td>
  <td>1582</td><td>12.04</td>
  <td>1660</td><td>12.63</td>
  <td>3025</td><td>23.02</td></tr>
<tr><td>2011-2012</td><td>13653</td>
  <td>7063</td><td>51.73</td>
  <td>1578</td><td>11.56</td>
  <td>1724</td><td>12.63</td>
  <td>3288</td><td>24.08</td></tr>
<tr><td>2012-2013</td><td>14850</td>
  <td>7639</td><td>51.44</td>
  <td>1976</td><td>13.31</td>
  <td>1911</td><td>12.87</td>
  <td>3324</td><td>22.38</td></tr>
<tr><td>2013-2014</td><td>15393</td>
  <td>7573</td><td>49.20</td>
  <td>2072</td><td>13.46</td>
  <td>2025</td><td>13.16</td>
  <td>3723</td><td>24.19</td></tr>
<tr><td>2014-2015</td><td>16119</td>
  <td>7969</td><td>49.44</td>
  <td>2064</td><td>12.80</td>
  <td>2146</td><td>13.31</td>
  <td>3940</td><td>24.44</td></tr>
<tr><td>2015-2016</td><td>16771</td>
  <td>8180</td><td>48.77</td>
  <td>2158</td><td>12.87</td>
  <td>2247</td><td>13.40</td>
  <td>4186</td><td>24.96</td></tr>
<tr><td>2016-2017</td><td>17472</td>
  <td>9024</td><td>51.65</td>
  <td>2127</td><td>12.17</td>
  <td>2127</td><td>12.17</td>
  <td>4194</td><td>24.00</td></tr>
<tr><td>2017-2018</td><td>17905</td>
  <td>9653</td><td>53.91</td>
  <td>2004</td><td>11.19</td>
  <td>2104</td><td>11.75</td>
  <td>4144</td><td>23.14</td></tr>
<tr><td>2018-2019</td><td>18162</td>
  <td>9597</td><td>52.84</td>
  <td>1955</td><td>10.76</td>
  <td>1588</td><td>8.74</td>
  <td>5022</td><td>27.65</td></tr>
<tr><td>2019-2020</td><td>18104</td>
  <td>10209</td><td>56.39</td>
  <td>968</td><td>5.35</td>
  <td>1123</td><td>6.20</td>
  <td>5804</td><td>32.06</td></tr>
<tr><td>2020-2021</td><td>18521</td>
  <td>11449</td><td>61.82</td>
  <td>1243</td><td>6.71</td>
  <td>1094</td><td>5.91</td>
  <td>4735</td><td>25.57</td></tr>
<tr><td>2021-2022</td><td>16828</td>
  <td>11159</td><td>66.31</td>
  <td>963</td><td>5.72</td>
  <td>981</td><td>5.83</td>
  <td>3725</td><td>22.14</td></tr>
<tr><td>2022-2023</td><td>16014</td>
  <td>9626</td><td>60.11</td>
  <td>1574</td><td>9.83</td>
  <td>1242</td><td>7.76</td>
  <td>3572</td><td>22.31</td></tr>
</tbody></table>			

## The impact of Covid-19 on Distinguished Clubs
The table shows that the percentage of clubs that were not Distinguished or better rose by over 3.5% at the start of the Covid-19 pandemic in 2020. The percent of Distinguished clubs (clubs which achieved 5 or 6 goals in 2019-2020) fell by around 50%, while Select Distinguished clubs fell by almost 30%. Interestingly, the number and percentage of President's Distinguished clubs (clubs which achieved 9 or 10 goals) rose by more than 4% in 2019-2020.

In 2020-2021, the number of clubs increased from 18,104 to 18,521, but the number of "Not Distinguished" clubs increased by more than 1,200. The number of President's Distinguished clubs fell by more than 1,000.

The number of President's Distinguished clubs dropped by another 1,000 clubs in 2021-2022. Although the number of "Not Distinguished" clubs fell, the overall number of clubs dropped from 18,521 to 16,828, causing the "Not Distinguished" percentage to rise to 66.31 - nearly 2/3 of the surviving clubs, the largest percentage in the past 15 years.

Although the total number of clubs dropped by more than 800 in 2022-2023, the number of Distinguished and Select Distinguished clubs rose by 611 and 261 respectively. The number of President's Distinguished clubs fell by only 153. This smaller drop combined with the increases in the other two categories caused the overall percentage of "Not Distinguished" clubs to recede to 60.11%. This is still higher than any point before the pandemic, and it remains to be seen whether the numbers of clubs and Active Members have turned a corner from the pandemic.