LOCAL CGI Script for Rest Services: Postgres to JSON

developed on cloud9

    Basic Ubuntu 20.04 LTS Workspace (Blank)
    PostgreSQL 10 +
    Python 3.8.2

Please check that .py files are executable ( step 00 )
If you install this form a git clone to a fresh cloud9 vm, do also step 0
otherwise jsut check to have already python 3.8.x + psycopg2 module installed.

To clone to Cloud9 workspace directly see https://docs.c9.io/docs/setting-up-github-workspace

------------------
How To Setup and Run
------------------
00 - set file permissions

    sudo chmod a+x rest.py

0 - install psycopg2

    sudo apt-get update
    sudo apt-get install libpq-dev python-dev
    sudo pip install psycopg2 (or>> sudo pip install psycopg2-binary)

    --- alternatively install psycopg2 in your env python
    * create env:
        pyhton3 -m venv venv
    * activate env
        . venv/bin/activate
    * install psycopg2 here
        pip install psycopg2 (or>> sudo pip install psycopg2-binary)

1 - Start Postgres Service

    sudo service postgresql start
    
2 - set postgre db (see https://community.c9.io/t/setting-up-postgresql/1573)

    psql
    user=# CREATE DATABASE "testdb";
    user-# \password dbuser
    Enter new password: 123456
    
3 - Create a table

    user=#\c testdb
    testdb=#CREATE TABLE mytable (id serial, name text);
    
4 - Insert some data

    testdb=#INSERT INTO mytable (id,name) VALUES (0,'Duccio');
    
5 - Set DB connection in /cgi-bin/rest.py

    07#  DBNAME = 'testdb'
    08#  HOST = 'localhost'
    09#  USER = 'dbuser'
    10#  PASSWORD = '123456'
    
5 - Start the CGI Script for Local Rest Service Server (set PORT in server.py)

    $ python -m http.server --cgi 8000
    
6 - Test the JSON response from an other Terminal:

    eg call the table we created before:
    
    curl 'http://localhost:8000/cgi-bin/rest.py?type=tab&schema=public&obj=mytable'
    
    will print
    
    [
      {
        "id": 0, 
        "name": "Duccio"
      }
    ]
    
    eg call a function that list all people that see green:
    
        First we add the column and we add some more data to the DB
        
        psql
        user=#\c testdb
        testdb=#ALTER TABLE mytable ADD COLUMN vision text;
        testdb=#UPDATE mytable set vision='green' WHERE name='Duccio';
        testdb=#INSERT INTO mytable (id,name,vision) VALUES (1,'Sorrenti','ocra');
        testdb=# CREATE OR REPLACE FUNCTION f_people_colorvision(color text) RETURNS TABLE(id int, name text) AS $$ SELECT id,name from mytable WHERE vision=$1 $$ LANGUAGE SQL; 

        then we test the Rest Call from terminal
        
        curl "http://localhost:8000/cgi-bin/rest.py?type=fun&schema=public&obj=f_people_colorvision&params=(color:'green')"
    
        that will print
        
        [
          {
            "id": "0", 
            "name": "Duccio"
          }
        ]
    
url - parameters:
    
    - type = tab (or) fun  //tab to call tables or views, fun to call functions
    
    - schema = schema_name // eg.public default postgres schema
    
    - obj = table_name (or) view_name (or) function_name
    
    - params = (param1:'val1', param2:'val2', ... paramN:'valN')  //Optional definition of function parameters
      note: parameters values are not quoted so you have to use single quote in the call right now; will add the quoting
            directly in the function calling postgres functions.
