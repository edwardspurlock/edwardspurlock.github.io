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

The <code>JupySQL</code> library is a fork of the earlier ipython-sql library and adds iPython 'magic' commands <code>%sql</code> and <code>%%sql</code>. This allows me to use SQL code in Jupyter notebook cells without wrapping the SQL in Python strings. 

The one-line <code>%sql</code> magic is used to run a single line of SQL in a notebook cell and can be used to mix Python and SQL code in a single cell, as in cell [2] below: 
```
[2]: print("This is a line of Python")     # this is a Python comment
     %sql sqlite:///test.db      -- this is a SQL comment, which uses two dashes (--) and not a # sign
     print("This is another line of Python")    # this is another Python comment
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

Besides <code>JupySQL</code>, I'll use <code>SQLAlchemy</code>. SQLAlchemy simplifies connecting to databases (including SQLite) from Python.

## Loading the CSV file in Jupyter notebook
When I started researching this project, I found an article that explained [how to load a CSV file into a SQLite database in Jupyter notebook](https://www.linkedin.com/pulse/accessing-sqlite3-database-from-jupyter-notebook-using-varun-lobo/). The author used the Pandas.read_csv() and DataFrame.to_sql() methods to import the CSV data and export it to a SQLite database. I tried the author's process, but I had to go beyond the example to make it work correctly with my data. 

As I noted in a previous project, I [used Python to scrape 15 years of web data from the public Toastmasters Distinguished Club Program dashboards](/projects/python/scrape-DCP.html). During this time span, Toastmasters completely revamped the educational program of communication and leadership projects. For the first 8 years (July 2008 through June 2016), the Distinguished Club Program (DCP) dashboards tracked the traditional educational program. From July 2016 until June 2020, the DCP dashboards tracked club performance in both the traditional and the new educational program. Starting in July 2020, the traditional program was discontinued, and the DCP dashboards only tracked achievements in the new program, Pathways.

In order to make the format of my CSV files consistent from year to year, [I added nulls (as empty strings) to the first eight years and last three years of data](https://github.com/espurlock/Scrape-Toastmasters-club-perf-data/blob/e271d43399903af955889f0d21511a7639b9ea1a/Extract_Club_data_from_HTML.py#L222). When I tried importing these data files into a database table using GUI tools like the SQLIte Database Browser or Oracle SQL Developer, the import worked perfectly - the empty strings were interpreted as SQL **NULL**s, exactly as I wanted. 

However, when I tried using the <code>Pandas.read_csv()</code> and <code>Pandas.DataFrame.to_sql()</code> methods, I found I had to convert the empty strings to Python <code>None</code> values. When I wrote the lines of data out to the SQLite database, it appeared that the Python <code>None</code> values were stored in SQLite as text values of "None", rather than as SQL **NULL**s as I expected.

Since I only have 15 years of data to deal with, I could have created tables and imported the CSV files using the DB Browser for SQLite GUI. Instead, I used SQL commands in a %%sql notebook cell to create the table, then used Python's <code>Path</code> and <code>subprocess</code> modules in another notebook cell to run SQLite's <code>.import</code> command-line directive:
 ```
    import subprocess
    from pathlib import Path
    db_name = Path('test.db').resolve()
    csv_file = Path('2022-2023.csv').resolve()
    table_name = 'July2022June2023'
    result = subprocess.run(['sqlite3', str(db_name), '-cmd', '.mode csv',
                         '.import --skip 1 ' + str(csv_file).replace('\\','\\\\') + ' ' + table_name],
                        capture_output = True)
 ```
This appeared to work perfectly, but when I checked the <code>test.db</code> file using DB Browser for SQLite, I discovered that the [SQLite <code>.import</code> command-line directive imports empty strings in the CSV file to empty strings (not **NULL**s) in the database](https://sqlite.org/forum/info/cf99368fe4c4512e).

I verified that the DB Browser GUI tool for SQLite converts unquoted empty strings in CSVs to **NULL**s in the database:  

![screenshot showing SQL import to table in DB Browser for SQLite](/assets/images/sql/SQL_import_using_GUI.png)

I then opened the database file in Jupyter notebook, and viewed the same fields:  

![screenshot showing NULL data values as viewed in Jupyter notebook](/assets/images/sql/SQL_NULLs_viewed_in_Jupyter.png)

I realized that the original approach (using <code>Pandas.read_csv()</code> and <code>DataFrame.to_sql()</code>) to import the data in Jupyter notebook had worked correctly. What had appeared to be text values of **None** were actually **NULL**s, which is what I wanted all along.

I edited the notebook, restoring the <code>Pandas.read_csv()</code> and <code>DataFrame.to_sql()</code> code. 

I found I needed to define column names:
```
dcp_columns = (
         'District', 'Division', 'Area', 
         'Club Number', 'Club Name', 
...
         'Club Distinguished Status',
         'Program Year', 'Month', 'Date'
)
```
and also define the data types for each column:
```
dcp_datatypes = {
    'District': str, 
...
    'Program Year': str,    # oldest available Program Year is '2008-2009', most recent full year is '2022-2023' 
    'Month': 'Int8',    # numeric values of 1 to 12 inclusive 
    'Date': str    # as of the last day of each month - values like '2008-07-31', '2023-06-30'
}
```

The final version of <code>.read_csv()</code> looks like this:
```
df = pd.read_csv(
    '2022-2023.zip',   # .read_csv() can read both zipped and unzipped .CSV files
    low_memory=False,    # default is True, which reads the file in chunks, which may cause mixed type inference
    encoding='ISO-8859-1',  # default is utf-8, which wasn't working with some non-U.S.A. club names
    names=dcp_columns,   # specify the names parameter to override the header row in the file
    skiprows=1,    # skip the header row when names are specified
    dtype=dcp_datatypes  # specify datatypes to ensure that null numeric values will not be typed as float  
)
```

Writing the Pandas DataFrame to SQL is easy, by comparison:
```
%sql sqlite:///test.db
conn=sqlite3.connect('test.db')
df.to_sql(name='July2022June2023', con=conn,  index=False, if_exists='replace')
```

And once the DataFrame has been written to the SQLite table, we can start using SQL commands:
```
%sql SELECT * FROM July2022June2023 LIMIT 5;
```




## Set up Jupyter notebook for SQL from scratch (local install on Windows)
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

3. Install Jupyter Notebook, SQLAlchemy, and JupySQL
```
pip install notebook
pip install sqlalchemy
pip install jupysql
```

4. Start Jupyter Notebook
```
jupyter notebook
```
The Jupyter 'tree' browser should open automatically

5. Create a new Notebook using the 'New' button and select the Python 3 kernel if prompted
![screenshot showing Jupyter Notebook New Item dialog](/assets/images/sql/Jupyter_tree_browser.png)

6. Import <code>pandas</code> and <code>sqlite3</code>, then load the JupySQL library:
```
import pandas as pd
import sqlite3
%load_ext sql
```

7. Create a Python tuple or list containing the column names:
```
dcp_columns = (
         'District', 'Division', 'Area', 
         'Club Number', 'Club Name', 
         'Club Status', 'Mem. Base', 'Active Members', 'Goals Met', 
         '_2_CCs', '_2_more_CCs', '_1_AC', '_1_more_AC', '_1_CL_AL_DTM', '_1_more_CL_AL_DTM',
         '_4_L1s', '_2_L2s', '_2_more_L2s', '_2_L3s', '_1_L4_L5_DTM', '_1_more_L4_L5_DTM',
         'New Members', 'Add. New Members',
         'Off. Trained Round 1', 'Off. Trained Round 2',
         'Mem. dues on time Oct & Apr', 'Off. List On Time',
         'Club Distinguished Status',
         'Program Year', 'Month', 'Date'
)
```

8. Define the datatypes for the columns:
```
dcp_datatypes = {
    'District': str, 
    'Division': str, 
    'Area': str,
    'Club Number': 'Int64',
    'Club Name': str,
    'Club Status': str,
    'Mem. Base': 'Int16',   # most clubs have less than 30 members, but could exceed 255
    'Active Members': 'Int16',  # as above, could exceed 255
    'Goals Met': 'Int8',    # max 10 goals possible
    # most clubs have fewer than 10 goals earned in any category, so Int8 should be sufficient:
    '_2_CCs': 'Int8', '_2_more_CCs': 'Int8', '_1_AC': 'Int8', '_1_more_AC': 'Int8', '_1_CL_AL_DTM': 'Int8', '_1_more_CL_AL_DTM': 'Int8',
    '_4_L1s': 'Int8', '_2_L2s': 'Int8', '_2_more_L2s': 'Int8', '_2_L3s': 'Int8', '_1_L2_L5_DTM': 'Int8', '_1_more_L4_L5_DTM': 'Int8',
    'New Members': 'Int8', 'Add. New Members':'Int16',   # max 4 for 'New Members', 'Add. New Members' could exceed 255
    'Off. Trained Round 1': 'Int8', 'Off. Trained Round 2': 'Int8',   # max 7
    'Mem. dues on time Oct & Apr': 'Int8', 'Off. List On Time': 'Int8',   # max 2
    'Club Distinguished Status': str,    # values of 'D', 'S', 'P', or NULL
    'Program Year': str,    # oldest available Program Year is '2008-2009', most recent full year is '2022-2023' 
    'Month': 'Int8',    # numeric values of 1 to 12 inclusive 
    'Date': str    # as of the last day of each month - values like '2008-07-31', '2023-06-30'
}
```

9. Load the CSV file or ZIPped CSV file into a Pandas dataframe:
```
df = pd.read_csv(
    '2022-2023.zip',   # .read_csv() can read both zipped and unzipped .CSV files
    low_memory=False,    # default is True, which reads the file in chunks, which may cause mixed type inference
    encoding='ISO-8859-1',  # default is utf-8, which wasn't working with some non-U.S.A. club names
    names=dcp_columns,   # specify the names parameter to override the header row in the file
    skiprows=1,    # skip the header row when names are specified
    dtype=dcp_datatypes  # specify datatypes to ensure that null numeric values will not be typed as float  
)
```

10. Connect to an existing database file (or create a new one) and write the Pandas dataframe to a table in the database:
```
%sql sqlite:///test.db
conn=sqlite3.connect('test.db')
df.to_sql(name='July2022June2023', con=conn,  index=False, if_exists='replace')
```
(you can also use <code>if_exists='append'</code> to add data to an existing table, or <code>ifexists='fail'</code> to abort the <code>.to_sql()</code> write (<code>'fail'</code> is the default) )

11. You can now use SQL commands to query the table:
```
%sql SELECT * FROM July2022June2023 LIMIT 5
```

## Starting up a new Jupyter Notebook with SQL after initial setup (local Windows)
1. Go to the directory where Jupyter Notebook was installed, activate the virtual environment, and start Jupyter Notebook:
```
PS > cd c:\Users\User\Python\SQL-in-Python
PS C:\Users\User\Python\SQL-in-Jupyter > .\Scripts\Activate.ps1
(SQL-in-Python) PS C:\Users\User\Python\SQL-in-Jupyter > jupyter notebook
```

2. In the Jupyter 'tree' browser, create a New Notebook:
![screenshot showing Jupyter Notebook New Item dialog](/assets/images/sql/Jupyter_tree_browser.png)

3. Import <code>sqlite3</code>, load the JupySQL library, and connect to the database file:
```
import sqlite3
%load_ext sql
%sql sqlite:///test.db
```

4. You can now use SQL commands to query the table:
```
%%sql
SELECT 
    (SELECT COUNT(DISTINCT "Club Number") 
     FROM July2022June2023
     WHERE Month = 7   -- July is the start of the Toastmasters year
    ) AS StartingClubCount,
    (SELECT COUNT(DISTINCT "Club Number")
     FROM July2022June2023
     WHERE Month = 6   -- June is the end of the Toastmasters year
    ) AS EndingClubCount;
```

## Set up a notebook for SQL projects in Github Codespaces
1. Create a [new repo](https://github.com/edwardspurlock/SQL-in-Jupyter) on Github and initialize with README

2. Click on the green Code button and create a new Codespace in the new repo: 
![Create a Codespace in a Github repository](/assets/images/sql/Create_Codespace.png)

3. In the terminal in the Codespace IDE, install Jupyter notebook, sqlalchemy, and ipython-sql:
    ```
    pip install notebook
    pip install sqlalchemy
    pip install ipython-sql
    ```
*Note: JupySQL is supposed to be a drop-in replacement for ipython-sql, but I was not able to get SQL integration working in the Codespace environment until I uninstalled JupySQL and installed ipython-sql in its place*

4. Run <code>jupyter notebook</code> to start Jupyter notebook. The Codespace may crash. Click **Reload** on the error message to restart the Codespace, and the <code>jupyter notebook</code> command should re-run automatically, showing:
```
[I 2024-01-20 23:06:03.582 ServerApp] Jupyter Server 2.12.3 is running at:
[I 2024-01-20 23:06:03.582 ServerApp] http://localhost:8888/tree?token=ffe93****************************************116
```

5. Ctrl-click on the 
   <code>http://localhost:8888/tree?token=...</code> 
   link to open in a new tab
![screenshot showing link to open Jupyter server](/assets/images/sql/Jupyter_server_ctrl_click.png)

6. Go back to the original tab, copy the token, and log in with the token. You can also create a password if you don't want to have to copy the token each time you start Jupyter Notebook in the future. 

7. Go back to the Codespaces editor tab and drag a CSV file or ZIPped CSV file from a local directory to the Explorer pane of the Codespaces IDE (this will make it available in the notebook)  

![screenshot showing Codespace Explorer pane](/assets/images/sql/Codespace_Explorer_pane.png)   

You can also drag a CSV file or ZIPped file directly into the Jupyter notebook 'tree' browser, but this takes a few seconds longer than dragging the file into the Codespace Explorer pane.

8. In the Jupyter 'tree' browser tab, use File | New | Notebook
![screenshot showing Jupyter notebook 'tree' browser](/assets/images/sql/Jupyter_tree_browser.png)  

to open a notebook in a new tab

## Start running SQLite
Steps to run SQLite in Jupyter notebook and import a CSV file:

1. In a new Jupyter notebook, create a new cell and import <code>sqlite3</code> and <code>pandas</code>:
    ```
    import sqlite3
    import pandas as pd
    ```

3. Load the JupySQL library:
    ```
    %load_ext sql
    ```

4. Create a database file:
    ```
    %sql sqlite:///test.db
    ```

5. Create a tuple containing the column names:
    ```
dcp_columns = (
         'District', 'Division', 'Area', 
         'Club Number', 'Club Name', 
         'Club Status', 'Mem. Base', 'Active Members', 'Goals Met', 
         '_2_CCs', '_2_more_CCs', '_1_AC', '_1_more_AC', '_1_CL_AL_DTM', '_1_more_CL_AL_DTM',
         '_4_L1s', '_2_L2s', '_2_more_L2s', '_2_L3s', '_1_L4_L5_DTM', '_1_more_L4_L5_DTM',
         'New Members', 'Add. New Members',
         'Off. Trained Round 1', 'Off. Trained Round 2',
         'Mem. dues on time Oct & Apr', 'Off. List On Time',
         'Club Distinguished Status',
         'Program Year', 'Month', 'Date'
)
    ```

6. Define the datatypes for each column:
    ```
dcp_datatypes = {
    'District': str, 
    'Division': str, 
    'Area': str,
    'Club Number': 'Int64',
    'Club Name': str,
    'Club Status': str,
    'Mem. Base': 'Int16',   # most clubs have less than 30 members, but could exceed 255
    'Active Members': 'Int16',  # as above, could exceed 255
    'Goals Met': 'Int8',    # max 10 goals possible
    # most clubs have fewer than 10 goals earned in any category, so Int8 should be sufficient:
    '_2_CCs': 'Int8', '_2_more_CCs': 'Int8', '_1_AC': 'Int8', '_1_more_AC': 'Int8', '_1_CL_AL_DTM': 'Int8', '_1_more_CL_AL_DTM': 'Int8',
    '_4_L1s': 'Int8', '_2_L2s': 'Int8', '_2_more_L2s': 'Int8', '_2_L3s': 'Int8', '_1_L2_L5_DTM': 'Int8', '_1_more_L4_L5_DTM': 'Int8',
    'New Members': 'Int8', 'Add. New Members':'Int16',   # max 4 for 'New Members', 'Add. New Members' could exceed 255
    'Off. Trained Round 1': 'Int8', 'Off. Trained Round 2': 'Int8',   # max 7
    'Mem. dues on time Oct & Apr': 'Int8', 'Off. List On Time': 'Int8',   # max 2
    'Club Distinguished Status': str,    # values of 'D', 'S', 'P', or NULL
    'Program Year': str,    # oldest available Program Year is '2008-2009', most recent full year is '2022-2023' 
    'Month': 'Int8',    # numeric values of 1 to 12 inclusive 
    'Date': str    # as of the last day of each month - values like '2008-07-31', '2023-06-30'
}
    ```

7. Load the CSV file or ZIPped CSV file into a Pandas dataframe:
```
df = pd.read_csv(
    '2022-2023.csv',   # .read_csv() can read both zipped and unzipped .CSV files
    low_memory=False,    # default is True, which reads the file in chunks, which may cause mixed type inference
    encoding='ISO-8859-1',  # default is utf-8, which wasn't working with some non-U.S.A. club names
    names=dcp_columns,   # specify the names parameter to override the header row in the file
    skiprows=1,    # skip the header row when names are specified
    dtype=dcp_datatypes  # specify datatypes to ensure that null numeric values will not be typed as float  
)
```

8. Write the dataframe to a new table in the database file:
```
conn=sqlite3.connect('test.db')
df.to_sql(name='July2022June2023', con=conn,  index=False, if_exists='replace')
```

9. Now you can use SQL commands directly in Jupyter:
```
%%sql
SELECT * FROM July2022June2023 limit 3;
```


## To create a new notebook in the SQL-in-Jupyter Codespace
1. Go to the [SQL-in-Jupyter Codespace](https://github.com/codespaces/refactored-space-memory-xg5549qwp6w3p46r). The Codespace will open in the Github Visual Studio IDE. 

2. In the terminal window, enter <code>jupyter notebook</code> to start the Jupyter 'tree' explorer in a new browser tab.

3. If you want to create a new database file from a CSV file or ZIPped CSV file, open the **SQL_Setup.ipynb** notebook and change the names of:
* the CSV / ZIP input file
```
df = pd.read_csv(
    '2022-2023.csv',   # change this line in this cell, e.g. 2021-2022.zip
...  # leave the other lines in this notebook cell as they are
)
```
* the name of the database file:
```
conn=sqlite3.connect('new_database_file.db')
```
* the name of the database table:
```
df.to_sql(name='tbl_2021_2022', con=conn,  index=False, if_exists='replace')
```

4. Save the **SQL_Setup.ipynb** file with a new name


## To use an existing table in an existing database file:
1. Connect to the Codespace and open the Jupyter notebook 'tree' browser as in steps 1 and 2 above

2. Open the **SQL_analysis_skeleton.ipynb** notebook

3. Save a copy of the notebook under a new name

4. Run the first three cells:
```
[1] : import sqlite3
[2] : %load_ext sql
[3] : %sql sqlite:///<existing database file>.db
```

5. Add new cells using the %sql (single-line) or %%sql (multi-line) magics to run SQL against the database:
```
[4] : %%sql
    SELECT 
        (SELECT COUNT(DISTINCT "Club Number") 
        FROM July2022June2023
        WHERE Month = 7   -- July is the start of the Toastmasters year
        ) AS StartingClubCount,
        (SELECT COUNT(DISTINCT "Club Number")
        FROM July2022June2023
        WHERE Month = 6   -- June is the end of the Toastmasters year
        ) AS EndingClubCount;
```