---
layout: posts
title:  "Using SQLite in Jupyter notebooks"
date:   2024-01-12 12:00:00 +0000
categories: SQL
highlight_home: true
tags: SQL Jupyter Codespaces
description: Notes on setting up a Jupyter notebook for SQL projects
header:
 overlay_image: /assets/images/encrypted_data.png
 teaser: /assets/images/Jupyter_logo.svg
---
# SQLite in Jupyter notebook
<!--more-->
Jupyter notebooks are a great way to explore and analyze data using Python or R. SQL is not built in to Jupyter notebooks, but's possible to use SQL to analyze data in a Jupyter notebook, after a bit of setup work.

## Tools used
It's possible to connect to many different databases from Jupyter notebook, but for this project I wanted to have everything installed on a single system. **SQLite** allows one to create a database file on a local machine or cloud server and read/write the database file using SQL queries. SQLite is included in Python, so it can be used from a Python script or from a standard Jupyter notebook by running <code>import sqlite3</code>, with no need to install a connector or other software.

However, using SQLite in Python involves embedding the SQL queries in Python strings and using Python methods to query the database and process the results. This is not a big deal when using SQLite as a data store for a larger Python program (such as [scraping web data](/projects/python/scrape-DCP.html#alternate_script)), but for doing data analysis where the emphasis is on SQL, the additional Python code detracts from the SQL queries.

The <code>ipython-sql</code> library adds iPython 'magic' commands <code>%sql</code> and <code>%%sql</code>. This allows me to use SQL code in Jupyter notebook cells without wrapping the SQL in Python strings. 

The one-line <code>%sql</code> magic is used to run a single line of SQL in a notebook cell and can be used to mix Python and SQL code in a single cell, as in cell [2] below: 
```
[2]:                             # this is a Python comment
     %sql sqlite:///test.db      -- this is a SQL comment, which uses two dashes (--) and not a # sign
                                 # this is another Python comment
```
The rest of the line after the <code>%sql</code> magic is interpreted as SQL, including comments, which need to be marked using SQL-style '--' rather than Python's '#'

The multi-line <code>%%sql</code> magic marks an entire cell as being SQL:
```
[3]: %%sql
     CREATE TABLE IF NOT EXISTS July2022June2023 (
     District: TEXT,
     Division: TEXT,
     AREA: TEXT,
     "Club Number": INT
     );
```

Besides <code>ipython-sql</code>, I'll use <code>SQLAlchemy</code>. SQLAlchemy simplifies connecting to databases (including SQLite) from Python.

## Loading the CSV file - Pandas versus SQLite .import
When I started researching this project, I found an article that explained [how to load a CSV file into a SQLite database in Jupyter notebook](https://www.linkedin.com/pulse/accessing-sqlite3-database-from-jupyter-notebook-using-varun-lobo/). The author used the Pandas.read_csv() and DataFrame.to_sql() methods to import the CSV data and export it to a SQLite database. I tried this process, but I ran into a problem with my data. 

As I noted in a previous project, I [used Python to scrape 15 years of web data from the public Toastmasters Distinguished Club Program dashboards](/projects/python/scrape-DCP.html). During this time span, Toastmasters completely revamped the educational program of communication and leadership projects. For the first 8 years (July 2008 through June 2016), the Distinguished Club Program (DCP) dashboards tracked the traditional educational program. From July 2016 until June 2020, the DCP dashboards tracked club performance in both the traditional and the new educational program. Starting in July 2020, the traditional program was discontinued, and the DCP dashboards only tracked achievements in the new program, Pathways.

In order to make the format of my CSV files consistent from year to year, I added nulls (as empty strings) to the first eight years and last three years of data. When I tried importing these data files into a database table using GUI tools like the SQLIte Database Browser or Oracle SQL Developer, the import worked perfectly - the empty strings were interpreted as SQL **Null**s, exactly as I wanted. 

However, when I tried using the <code>Pandas.read_csv()</code> and <code>Pandas.DataFrame.to_sql()</code> methods, I found I had to convert the empty strings to Python <code>None</code> values. When I wrote the lines of data out to the SQLite database, it appeared that the Python <code>None</code> values were stored in SQLite as text values of "None", rather than as SQL **Null**s as I expected.

I decided to use SQLite's built-in **Import** function. Since I only have 15 years of data to deal with, I could have created tables and imported the CSV files using the **DB Browser for SQLite** GUI. Instead, I used SQL commands in a %%sql notebook cell to create the table:
```
%%sql
    CREATE TABLE IF NOT EXISTS July2022June2023 (
    "District" TEXT,
    "Division" TEXT,
...
    "Month" INT,
    "Date" TEXT
    );
```
 then used Python's <code>subprocess</code> library to run SQLite's <code>.import</code> command-line directive:
 ```
    db_name = Path('test.db').resolve()
    csv_file = Path('2022-2023.csv').resolve()
    table_name = 'July2022June2023'
    result = subprocess.run(['sqlite3', str(db_name), '-cmd', '.mode csv',
                         '.import --skip 1 ' + str(csv_file).replace('\\','\\\\') + ' ' + table_name],
                        capture_output = True)
 ```




## Set up Jupyter notebook for SQL projects locally (Windows)
1. In PowerShell, go to the directory where you want to set up the notebook
```
cd c:\Users\User\Python
```
2. Create a Python virtual environment and activate it in PowerShell
```
py -m venv SQL-in-Python
cd SQL-in-Python
.\Scripts\Activate.ps1
```
3. Install Jupyter Notebook, SQLAlchemy, and ipython-sql
```
pip install notebook
pip install sqlalchemy
pip install ipython-sql
```
4. Start Jupyter Notebook
```
jupyter notebook
```
The Jupyter 'tree' browser should open automatically
5. Create a new Notebook using the 'New' button and select the Python 3 kernel if prompted
6. Import <code>sqlite3, subprocess, Path</code>, and load the ipython-sql library
```
import sqlite3
import subprocess
from pathlib import Path
%load_ext sql
```
7. Create a new database file, or connect to an existing file in the Jupyter home directory
```
%sql sqlite:///test.db
```
8. Create an empty table in the database
```
    %%sql
    CREATE TABLE IF NOT EXISTS July2022June2023 (
    "District" TEXT,
    "Division" TEXT,
    "Area" TEXT,
    "Club Number" INT,
    "Club Name" TEXT,
    "Club Status" TEXT,
    "Mem. Base" INT,
    "Active Members" INT,
    "Goals Met" INT,
    "_2_CCs" INT,
    "_2_more_CCs" INT,
    "_1_AC" INT,
    "_1_more_AC" INT,
    "_1_CL_AL_DTM" INT,
    "_1_more_CL_AL_DTM" INT,
    "_4_L1s" INT,
    "_2_L2s" INT,
    "_2_more_L2s" INT,
    "_2_L3s" INT,
    "_1_L4_L5_DTM" INT,
    "_1_more_L4_L5_DTM" INT,
    "New Members" INT,
    "Add. New Members" INT,
    "Off. Trained Round 1" INT,
    "Off. Trained Round 2" INT,
    "Mem. dues on time Oct & Apr" INT,
    "Off. List on Time" INT,
    "Club Distinguished Status" TEXT,
    "Program Year" INT,
    "Month" INT,
    "Date" TEXT
    );
```
9. Use the <code>sqlite</code> command line tool to load the CSV file into the table:
    ```
    db_name = Path('test.db').resolve()
    csv_file = Path('2022-2023.csv').resolve()
    result = subprocess.run([
        r'c:\Users\User\Downloads\SQLiteDatabaseBrowserPortable\tools\sqlite3', 
        str(db_name), '-cmd', '.mode csv',
        '.import --skip 1 ' + str(csv_file).replace('\\','\\\\') + ' July2022June2023'],
        capture_output = True)
    ```
10. You can now use SQL commands to query the database:
```
%%sql
SELECT * FROM July2022June2023 LIMIT 3;
```

## Set up a notebook for SQL projects in Github Codespaces
1. Create a [new repo SQL-in-Jupyter](https://github.com/edwardspurlock/SQL-in-Jupyter) on Github and initialize with README
2. Open a new [Codespace (Silver-Spork)](https://silver-spork-vj665g7vgrgc66xr.github.dev/) from the new repo 
3. Install Jupyter notebook, sqlalchemy, and ipython-sql, then run jupyter notebook:
    ```
    pip install notebook
    pip install sqlalchemy
    pip install ipython-sql
    jupyter notebook
    ```
4. Ctrl-click on the 
   <code>http://localhost:8888/tree?token=...</code> 
   link to open in a new tab
5. Go back to the original tab, copy the token, log in with the token, and create a password 
6. Use File | New | Notebook 
   to open a notebook in a new tab
7. Go back to the Codespaces VS editor tab
8. Drag a CSV file from a local directory to the Explorer pane of the Codespaces Explorer pane (this will make it available in the notebook)

## Start running SQLite
Steps to run SQLite in Jupyter notebook and import a CSV file:

1. Imports:
    ```
    import sqlite3
    import subprocess
    from pathlib import Path
    ```

3. Load the SQL Alchemy library:
    ```
    %load_ext sql
    ```

4. Connect to the database file (create it if it doesn't exist):
    ```
    %sql sqlite:///test.db
    ```

5. Create a table:
    ```
    %%sql
    CREATE TABLE IF NOT EXISTS July2022June2023 (
    "District" TEXT,
    "Division" TEXT,
    "Area" TEXT,
    "Club Number" INT,
    "Club Name" TEXT,
    "Club Status" TEXT,
    "Mem. Base" INT,
    "Active Members" INT,
    "Goals Met" INT,
    "_2_CCs" INT,
    "_2_more_CCs" INT,
    "_1_AC" INT,
    "_1_more_AC" INT,
    "_1_CL_AL_DTM" INT,
    "_1_more_CL_AL_DTM" INT,
    "_4_L1s" INT,
    "_2_L2s" INT,
    "_2_more_L2s" INT,
    "_2_L3s" INT,
    "_1_L4_L5_DTM" INT,
    "_1_more_L4_L5_DTM" INT,
    "New Members" INT,
    "Add. New Members" INT,
    "Off. Trained Round 1" INT,
    "Off. Trained Round 2" INT,
    "Mem. dues on time Oct & Apr" INT,
    "Off. List on Time" INT,
    "Club Distinguished Status" TEXT,
    "Program Year" INT,
    "Month" INT,
    "Date" TEXT
    );
    ```

6. Use the <code>sqlite</code> command line tool to load the CSV file into the table:
    ```
    db_name = Path('test.db').resolve()
    csv_file = Path('2022-2023.csv').resolve()
    result = subprocess.run(['sqlite3', str(db_name), '-cmd', '.mode csv',
                         '.import --skip 1 ' + str(csv_file).replace('\\','\\\\') + ' July2022June2023'],
                        capture_output = True)
    ```

7. Now you can use SQL commands directly in Jupyter:
    ```
    %%sql
    SELECT * FROM July2022June2023 limit 3;
    ```