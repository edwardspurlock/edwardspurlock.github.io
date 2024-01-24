---
layout: posts
title:  "Data import, exploration, and cleanup using SQL"
date:   2024-01-22 12:00:00 +0000
categories: SQL
highlight_home: true
tags: SQL Jupyter
description: Initial data exploration and cleanup of Toastmasters Distinguished Club Program data
header:
 overlay_image: /assets/images/encrypted_data.png
 teaser: /assets/images/sql-database-generic-svgrepo-com.svg
excerpt:
 
---
# Toastmasters Club Performance data exploration and cleanup using SQL
<!--more-->
Now that I'm able to [use SQLite in a Jupyter notebook](/projects/sql/SQL-in-Jupyter.html), it's time to import the [Club Performance data that I scraped from the Toastmasters website](/projects/python/scrape-DCP.html) so I can do some exploration and (if necessary) cleanup.

I downloaded CSV files from my [Google Cloud Storage archive](/projects/python/scrape-DCP.html#making_data_available), ZIPped them, and moved the ZIP files to my Jupyter notebook home to make them available to Jupyter notebooks.

## Importing the data

Previously, I migrated the data for one year (2022-2023) into a SQLite database file named 'test.db'. For this project, I initially decided to create database files and tables named with the last digits of each CSV file's program year name. For example, the input file for program year 2008-2009 was named '2008-2009.zip'. I created a SQLite data file named '0809.db', and created a table named 'tbl_0809' inside the file.

To make it easier to create these individual database files and tables, I created a short Python function:
```
def import_csv_to_sqlite(input_csv, db_filename=None):
    dcp_columns = (
         'District', 'Division', 'Area', ...
    # the rest of `dcp_columns` is the same as in the previous project

    dcp_datatypes = {
        'District': str, 'Division': str, 'Area': str, ...
    # the rest of `dcp_datatypes` is the same as in the previous project

    m = re.match("^20(\d\d).20(\d\d)\.(csv|zip)$" , input_csv)
    if m:
        tbl_name = 'tbl_' + m.group(1) + m.group(2)
        if not db_filename:
            db_filename = m.group(1) + m.group(2) + '.db'
    else:
        print(f"Input filename `{input_csv}` does not match expected `20xx-20yy.csv` or `20xx-20yy.zip`")
        return  # exit if input filename doesn't match expected CSV/ZIP file

    if os.path.exists(input_csv):
        df = pd.read_csv(
            input_csv,   # .read_csv() can read both zipped and unzipped .CSV files
            low_memory=False,    # default is True, which reads the file in chunks,
                                 # which may cause mixed type inference
            encoding='ISO-8859-1',  # default is utf-8, which wasn't working with some non-U.S.A. club names
            names=dcp_columns,   # specify the names parameter to override the header row in the file
            skiprows=1,    # skip the header row when names are specified
            dtype=dcp_datatypes  # specify datatypes to ensure that
                                 # null numeric values will not be typed as float  
        )
        print(f"Creating SQL table {tbl_name} in database file {db_filename}")
        conn=sqlite3.connect(db_filename)
        df.to_sql(name=tbl_name, con=conn,  index=False, if_exists='replace')
```

I then ran this function on a couple of the ZIPped CSV files:
```
[3]: import_csv_to_sqlite('2008-2009.zip')
Creating SQL table tbl_0809 in database file 0809.db

[4]: import_csv_to_sqlite('2009-2010.zip')
Creating SQL table tbl_0910 in database file 0910.db
```

## Initial data exploration - counting clubs
With a couple of years imported, I started by counting the number of clubs each month:
```
[5]: %%sql sqlite:///0809.db
     SELECT Date, COUNT("Club Number") FROM tbl_0809 GROUP BY Date ORDER BY Date;
```

| Date | COUNT("Club Number") |
| ---- | ---- |
| 2008-07-31 | 11568 |
| 2008-08-31 | 11598 |
| 2008-09-30 | 11458 |
| 2008-10-31 | 11526 |
| 2008-11-30 | 11577 |
| 2008-12-31 | 11628 |
| 2009-01-31 | 11692 |
| 2009-02-28 | 11763 |
| 2009-03-31 | 11840 |
| 2009-04-30 | 11631 |
| 2009-05-31 | 11741 |
| 2009-06-30 | 12024 |

The Toastmasters About page currently shows around 14,200 clubs, so 11,500 - 12,000 clubs in 2008-2009 seems reasonable. 

What about 2009-2010?
```
[6]: %%sql sqlite:///0910.db
     SELECT Date, COUNT("Club Number") FROM tbl_0910 GROUP BY Date ORDER BY Date;
```

| Date | COUNT("Club Number") |
| ---- | ---- |
| 2009-07-31 | 12075 |
| 2009-08-31 | 12134 |
| 2009-09-30 | 12184 |
| 2009-10-31 | 12100 |
| 2009-11-30 | 12148 |
| 2009-12-31 | 12198 |
| 2010-01-31 | 12270 |
| 2010-02-28 | 12317 |
| 2010-03-31 | 12034 |
| 2010-04-30 | 12169 |
| 2010-05-31 | 12248 |
| <span style="color:red;">2010-06-30</span> | <span style="color:red;">25028</span> |

Wait, the number of clubs more than doubled from May to June? That doesn't seem right!

```
[7]: %%sql
     SELECT Date, COUNT(DISTINCT "Club Number"), COUNT("Club Number")
     FROM tbl_0910 WHERE Date = '2010-06-30' GROUP BY Date ORDER BY Date;
```
<table>
<thead><tr><th>Date</th><th>COUNT(DISTINCT "Club Number")</th><th>COUNT("Club Number")</th></tr></thead>
<tbody><tr><td>2010-06-30</td><td>12514</td><td>25028</td></tr></tbody>
</table>

25,028 == 2 * 12,514, so it appears that all the rows of data for June 2010 have been duplicated. 

```
[8]: %%sql
      CREATE TABLE IF NOT EXISTS tbl_0910_distinct AS SELECT DISTINCT * FROM tbl_0910;

      SELECT Date, COUNT("Club Number") 
      FROM tbl_0910_distinct WHERE Date in ('2009-07-31','2009-08-31','2010-05-31','2010-06-30') 
      GROUP BY Date ORDER BY Date;
```
<table>
<thead><tr><th>Date</th><th>COUNT("Club Number")</th></tr></thead>
<tbody><tr><td>2009-07-31</td><td>12075</td></tr><tr><td>2009-08-31</td><td>12134</td></tr>
<tr><td>2010-05-31</td><td>12248</td></tr><tr><td>2010-06-30</td><td>12514</td></tr></tbody>
</table>

That looks more reasonable.

## Importing the rest of the data

After thinking about it, I decided to create a separate table for each Toastmasters program year, but create all the tables in a single database file:
```
[9]: for y in range(2008,2023):
          input_csv = f"{y}-{y+1}.zip"
          import_csv_to_sqlite(input_csv, 'all_years.sqlite')
 ```

As noted above, 2009-2010 has duplicate data in June 2010, so we need to create a new table with the duplicates removed, remove the original tbl_0910, and rename the de-duplicated table:
```
[10]: %%sql sqlite:///all_years.sqlite
      CREATE TABLE tbl_0910_distinct AS SELECT DISTINCT * FROM tbl_0910;
      DROP TABLE tbl_0910;
      ALTER TABLE tbl_0910_distinct RENAME TO tbl_0910;
```

As a quick sanity check, I used a Python loop to create a SQL command to display the end-of-year Club Number count for each table in the database:
```
[11]: sql_cmd = []
      for y4 in range(8,23):
          tbl_name = f"tbl_{y4:02}{y4+1:02}"
          sql_cmd.append(f'SELECT Date, COUNT("Club Number") FROM {tbl_name} WHERE Month = 6 GROUP BY Date\n')
      sql_cmd = 'UNION\n'.join(sql_cmd) + 'ORDER BY Date;'
      print(sql_cmd)

SELECT Date, COUNT("Club Number") FROM tbl_0809 WHERE Month = 6 GROUP BY Date
UNION
SELECT Date, COUNT("Club Number") FROM tbl_0910 WHERE Month = 6 GROUP BY Date
UNION
...
SELECT Date, COUNT("Club Number") FROM tbl_2122 WHERE Month = 6 GROUP BY Date
UNION
SELECT Date, COUNT("Club Number") FROM tbl_2223 WHERE Month = 6 GROUP BY Date
ORDER BY Date;
```

I then ran the SQL command in a new Jupyter cell:
```
[12]: %%sql 
   {{sql_cmd}}
```

| Date | COUNT("Club Number") |
| ---- | ---- |
| 2009-06-30 | 12024 |
| 2010-06-30 | 12514 |
| 2011-06-30 | 13139 |
| 2012-06-30 | 13653 |
| 2013-06-30 | 14850 |
| 2014-06-30 | 15393 |
| 2015-06-30 | 16119 |
| 2016-06-30 | 16771 |
| 2021-06-30 | 18521 |
| 2022-06-30 | 16828 |
| 2023-06-30 | 16014 |
| 6/30/2017 | 17472 |
| 6/30/2018 | 17905 |
| 6/30/2019 | 18162 |
| 6/30/2020 | 18104 |

The Club Number counts look all right but...what's up with the dates for 2017, 2018, 2019, and 2020?

## Fixing the date formats

When I originally scraped the Distinguished Club Program data from the Toastmasters website, I found that I had to use a different method to gather the data for clubs in a couple of districts. After getting the data for these two districts, I used Excel to merge it with the rest of the data for the four affected program years (2016-2017, 2017-2018, 2018-2019, 2019-2020). At the time, I didn't notice that Excel had reformatted the Date column from 'yyyy-mm-dd' to 'mm/dd/yyyy' in those four CSV files.

To fix the problem, I could have:
* dropped the tables (tbl_1617, tbl_1718, tbl_1819, tbl_1920),
* fixed the Date column in the four CSVs using Python or Excel,
* re-uploaded the fixed CSVs, and
* re-created the tables. 

However, since this is a SQL project, I decided to fix the problem in the tables:
```
[13]: %%sql
      UPDATE tbl_1617
      SET Date =
          CASE
          WHEN Month IN (7,8) THEN SUBSTR("Program Year", 1, 4) || '-0' || Month || '-31'
          WHEN Month = 9 THEN SUBSTR("Program Year", 1, 4) || '-09-30'
          WHEN Month IN (10,12) THEN SUBSTR("Program Year", 1, 4) || '-' || Month || '-31'    
          WHEN Month = 11 THEN SUBSTR("Program Year", 1, 4) || '-11-30'
          WHEN Month IN (1,3,5) THEN SUBSTR("Program Year", 6, 4) || '-0' || Month || '-31'
          WHEN Month = 2 THEN 
              CASE 
              WHEN SUBSTR("Program Year", 6, 4) = '2020' THEN '2020-02-29'
              ELSE SUBSTR("Program Year", 6, 4) || '-02-28'
              END
          ELSE SUBSTR("Program Year", 6, 4) || '-0' || Month || '-30'
          END;
```

There are cleaner ways to do this in SQL, but this quick hack worked. I verified that all the dates in tbl_1617 were in the correct format by running:
```
%sql SELECT DISTINCT Month, Date FROM tbl_1617;
```

I then ran the <code>UPDATE</code> statement for tbl_1718, tbl_1819, and tbl_1920, and verified the <code>UPDATE</code>s by running the <code>SELECT DISTINCT</code> statement against each table. I then updated the <code>sql_cmd</code> to display the end-of-year member count and average members per club as well as club count:
```
[17]: sql_cmd = []
      for y4 in range(8,23):
          tbl_name = f"tbl_{y4:02}{y4+1:02}"
          sql_cmd.append(f'''SELECT Date, printf("%,d", COUNT("Club Number")) AS "Total Clubs", 
                       printf("%,d", SUM("Active Members")) AS "Total Members", 
                       printf("%.2f", AVG("Active Members")) AS "Average Members/Club"
                       FROM {tbl_name} WHERE Month = 6 GROUP BY Date\n''')
      sql_cmd = 'UNION\n'.join(sql_cmd) + 'ORDER BY Date;'
```

I then ran the <code>sql_cmd</code> in a new cell:
```
[18]: %sql {{sql_cmd}}
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

## Combining all 15 years into one convenient table
Finally, in order to minimize the amount of Python in future SQL exploration projects, I combined all 15 years into a single new database table:
```
[19]: %sql CREATE TABLE tbl_all AS SELECT * FROM tbl_0809;
      for i in range(9,23):
          tbl_name = f"tbl_{i:02}{i+1}"
          %sql INSERT INTO tbl_all SELECT * FROM {{tbl_name}};
```
I then checked the year-end Total Clubs, Total Members, and Members/Club for the entire tbl_all table:
```
[20]: %%sql
      SELECT Date, printf("%,d", COUNT("Club Number")) AS "Total Clubs", 
                   printf("%,d", SUM("Active Members")) AS "Total Members", 
                   printf("%.2f", AVG("Active Members")) AS "Average Members/Club"
      FROM {tbl_name} WHERE Month = 6 GROUP BY Date
```
and verified that the numbers were the same as before:

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

## Next steps
Since the CSV files in Google Cloud Storage still contain two different formats for the Date parameter, I need to do a short Python or Excel project to update the files for 2016-2017...2019-2020 to a consistent format. 

I will create additional SQL projects to continue exploring the Toastmasters Distinguished Club Program data.