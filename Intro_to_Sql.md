# Introduction to SQL by Jon Flanders #

This course focuses on standards-based SQL and perspectives. Check out ANSI and ISO standards.

Databases are used to hold and manage lots of data, and for many Relational Databases and Relational Database Management Systems, SQL is the means to manage and query it. There are many kinds of databases though, and different means to manage and query them. There are also Object-Based and Document-Based databases, and there are other query languages out there.

Relational in this case means there is data and there are relationships between different sets of data. With SQL databases, there are Tables, Columns, and Rows. A Table is named, and is composed of Columns. Columns have names and constraints on the kind of data it can hold. A row is data that is across all columns, and there is one point of data held by each column. Data in rows can be queried.

Tables have Keys, and each table should have one column that can uniquely identify a row, known as a Primary Key. No two rows in a table can share the same Primary Key. Tables may also have Foreign Keys, where a Foreign Key is a value that is equal to a value that is a Primary Key in another table. The idea is Foreign Keys can be easily "expand" the amount of data is associated with a record. Sometimes it's not clear what data value in a set of data should be the key, so you may just create and assign some unique identifier, like an incrementing integer.

Suppose you have a record that contains a first name, last name, and email address. Now things change where you want to store multiple emails for a given person. **Don't just add another column to your person record!** Remove email column from person, give person a unique id, create a new table for emails with columns unique id, person key as a foreign key, and email.

The process of designing a database like this is called *Normalization*. 

A SQL Statement is an actionable set of words that makes the database do something for you, like create data structures, add new data and remove and alter data. In ANSI Standard SQL, a valid SQL statement **does** have a semi-colon at the end of each statement. SQL is not case-sensitive. Convention is to uppercase SQL key words and lowercase user words. Comments can be used, `--` is for a single line, ` /* … */` is for multi-line.

Commands refer to the first key word in a SQL Statement. Think `SELECT`, `INSERT`, `DELETE`, `CREATE`, `DROP`. What comes after each command depends on the command. In `SELECT`, you list columns, literal values, or aggregated values, and is then followed by a `FROM` clause that contains a number (0..N) of table names. 

Naming things can be a point of contention, there isn't one agreed upon way to name things in a database. What's more important is consistency. For this course, the author uses the following conventions

  * Table names are singular, so `email` is used instead of `emails`
  * Column names are not used more than once in one database, meaning there will not be two tables with a column that matches in name.

Names are scoped though. When writing SQL, it's best practice to use the "full" name of the thing you're referring to: i.e. *use multi-part naming*.

So this isn't ANSI SQL, but ANSI SQL doesn't define how to create databases. Most systems have though

```
  CREATE DATABASE Contacts;
  USE DATABASE Contacts;

  CREATE TABLE Person (...);
```

Each column has a Data Type it can hold, and this is defined on table or column creation. Common types include characters, varying amount of characters, binary values, smallint, integer, bigint, boolean, data, time, timestamps.

There are lots of RDMS's out there, almost all of them use ANSI SQL, but extend it with vendor-specific features, so some queries you might see might not be the same in a different RDMS. The author will be using MySQL for examples, but notes there is an ANSI compliance mode he will use.


## The `SELECT` Command ##

The `SELECT` Command is the primary means to query your data to answer specific questions you will have. Some questions you will be able to answer can be like "Who are all my contacts?", "How many contacts do I have?", "Which one of my contacts is the oldest?"

`SELECT` only requires a select list to be valid, but if you want more than displaying variables, constants, or calculated values of such things, you'll be listing columns from a table. You then need a `FROM` clause. When listing tables in your `FROM` clause, you can alias the table name to save some typing.

To constrain the data returned, you can add clauses after the `FROM` clause. You can also add `DISTINCT` before the select list, which is called a result set qualifier.

### `WHERE` and operators used in `WHERE`s ###

After `FROM`, you can add `WHERE` clauses, where there are a set of expressions that evaluate to true or false. Any particular rows that are true through the `WHERE` clause are returned. You can use any of these simple operators in your `WHERE`, and you can use these on non-numeric data types also.

  * `=` - equivalency
  * `<>` - not equal to
  * `>` - greater than, `<` - less than
  * `>=`, `<=` - greater than or equal, less than or equal

For example, the less than, greater than operators use positions in the alphabet when operating on strings, or varchars, etc. So ` 'ABC' < 'DEF' ` is true, since D comes after A in the alphabet.

You can include more expressions in the `WHERE` clause by using `AND` and `OR`.  Note that `AND` has precedence over `OR`, remember your boolean algebra. Also, the `IN` operator evaluates as `OR`s one after the other. However, you should also wrap your expressions in parenthesis as best practice.

There are other operators available also that rely more on knowledge of set theory. `BETWEEN` specifies an inclusive range of values, like `p.age BETWEEN 13 AND 17`.

`LIKE` is for use with strings, and does pattern matching. You use '%' for matching any character for any number of characters.

`IN` says your value must be in a specific list of values. Values can be numeric, strings, dates, or can be the result set of a subquery. If the data is equal to any of the values in the list, the record is included.

`IS` is used when you want to work with `NULL` values. `NULL` does not work with `=` or `<>`, you use `IS NULL` or `IS NOT NULL`.


## Result "shaping" ##

"Shaping" a result set means changing how the data set looks or is organized without having a set difference of data. If you can shape the data with your query before your app consumes the data, it's a good idea to do so.

`ORDER BY` lets you sort your result set. It comes after your `WHERE` clause if there is one. You specify one or more columns from the result set, and it sorts the set one column after the other. The default is to sort by ASCending values, but you can specify `DESC` for descending values (large to small, end of alpha to start of alpha, more recent time followed by less recent time as you scroll down your set).

There are some special functions that operate on result sets. The author refers to these as "set functions". `COUNT` takes a column (there are other things it can take, but consider that to be more advanced), and returns the number of rows that have values in that column. Note that `NULL` values are not included, however `COUNT(*)` will include null values, which basically means how many rows are there. Note that it doesn't work to pass in a subquery into a `COUNT` call.

`MAX` and `MIN` return the greatest and lowest value of the column. Nulls are not considered when calculating theses. `AVG` returns the average value, only for numeric columns and does not include Nulls. `SUM` returns the sum total for a numeric column, and nulls are not included. If you are `SELECT`ing a set function, give it a column alias since it won't have a name. 

You can pass in result set qualifiers (`DISTINCT` and `ALL`, though `ALL` is default and almost always implied) along with a column when you are using a set function. Note that it's almost always the case that you can not `SELECT` a set function and a plain column as part of a select list, it doesn't really make sense to do that. `GROUP BY`s are an exception to this.

`GROUP BY` lets you break up result sets into subsets. You specify a column(s) in your group by, and that column acts as your "key" to the subsets. You can then do something like

```
  SELECT p.last_name, COUNT(p.first_name)
  FROM test.person p
  GROUP BY p.last_name
```

What is the count of every unique first name in contacts?

```
  SELECT p.first_name, COUNT(DISTINCT p.first_name)
  FROM test.person p
  GROUP BY p.first_name
``` 

Suppose you wanted to filter result sets after you have grouped by. You would want to use `HAVING`. `HAVING` is very similar to `WHERE`.


## Joins ##

Joins connect tables together to expand the amount of data you can display and query from. A join needs to know how to relate the rows of multiple tables, so to do that, you often tell it the columns to connect on. Most of the time, you'll want to join on primary keys.

Beware of `CROSS` joins, which may occur if write something like `FROM test.a, test.b /* end of query */`. There will be an implicit `CROSS` join, which means the set to filter from will be the Cartesian product of the two tables.

`INNER JOIN` is the most typical join you'll see, where it finds the rows in table a that have a key value that matches a key value in table b, and the rows are connected into one. `INNER JOIN` takes an `ON` clause, which needs a boolean expression to determine whether to match the rows. Rows are only joined if there is a successful match.

Outer joins work differently. Outer joins will return rows even if there isn't a match, but where there isn't a match, columns from the unmatched table will hold `NULL`.

`FULL OUTER JOIN` will return at least all the rows in both tables, and works like both a `LEFT OUTER JOIN` and `RIGHT OUTER JOIN`. This is useful for showing all information you might have, even if it's incomplete or missing. Note that the author showed that MySQL errored when using `FULL OUTER JOIN` and that it needs left outer joins or right outer joins.

`LEFT OUTER JOIN` will return at least all rows in the "left hand side" of the table, or the table that is declared before the words `LEFT OUTER JOIN` in your query. If there is a match on the join to the table declared after `LEFT OUTER JOIN`, the columns from the "right hand table" are returned with their values. If there is no match on the join, all the columns from the "right hand table" will be filled with `NULL`.

`RIGHT OUTER JOIN` behaves like LEFT OUTER JOINS, except all the rows from the "right hand side" table, the table declared after the words `RIGHT OUTER JOIN`, will be returned, and columns from the table declared before the words `RIGHT OUTER JOIN` will return their values if matched, else return nulls.

Sometimes, it can be useful to join a table to itself, like for describing managerial relationships in an "employees" table. You simply follow the same syntax described above and give your left and right hand side table declarations different aliases.


## Insert, Update, and Delete ##

`INSERT INTO` is the means to add new records to a table. You can only insert into one table per `INSERT` statement. After you declare the table, list the columns you are inserting data into *always*. After your list, you use `VALUES` to then list the data records you wish to insert. Each record is a parens-wrapped list of data where the position of each data is inserted into the matching position column listed after the table. You can insert multiple rows into one table with one insert statement.

Note that you can also `INSERT INTO … SELECT` if you would like to insert data from one table to another, or insert while transforming data, etc.

`UPDATE` let's you change existing data using a syntax like `UPDATE table SET /* list of assignment statements */`. You *almost always* need to qualify this with a `WHERE` clause, otherwise you'll change every record in the table. There are more variations on syntax and qualifiers you can use with `UPDATE`, but those are more advanced, and may vary with RDBS.

`DELETE FROM` deletes records from the database. `DELETE FROM` without qualifiers deletes all rows from a table! Can only specify one table per `DELETE`. This can be very dangerous, so it may be helpful to issue commands to start/end a *transaction* while making a delete.  Transaction syntax may vary from DBS, but ANSI syntax for starting a transaction is like

```
  START TRANSACTION;
  -- Issue some commands, like a DELETE
  -- Before you commit, you can verify your results
  COMMIT;
  -- Or recover
  -- ROLLBACK;
```

Transactions in SQL have qualities that are described by an acronym, ACID.

* Atomic - all of it happens, or none of it does
* Consistent - database is left in a consistent state
* Isolated - changes don't leak out(?)
* Durable - Transaction is stored and recoverable in case of failure beyond the DBS

Note that if you are attempting to delete a row that has contains a column that is a foreign key in another table, the rows in that another table with that foreign key need to be deleted also. Otherwise, the delete may not occur, or the row with the foreign key will get orphaned.


## Creating Tables ##

SQL has a subset of the language designed for creating your schema for holding your data, like for creating databases and tables, and adding columns. This is typically called DDL - Data Definition Language.

This is not ANSI SQL, but you can typically do

```
  CREATE DATABASE YourDb;
  USE DATABASE YourDb;
```

and this will create your database and set your current working database to YourDb.

Creating tables is done with `CREATE TABLE` followed by your tablename, then a list of columns. Each column item starts with a column name, followed by data type, and constraints.

For example, one constraint is on allowing or not allowing `NULL` into the column. Allowing `NULL` is default behavior, but to not allow null values, you follow the data type in the column definition with `NOT NULL`.

Another constraint is `PRIMARY KEY`. `NULL` values can not be inserted into a primary key column. You can use `PRIMARY KEY` multiple times in a table definition to create a compound primary key.

If you'd like to have multiple constraints, you can move your declarations to the end and use the `CONSTRAINT` clause, where you name your constraint, list your constraints, then specify the column that set applies to.

If you wanted to add a foreign key constraint, it could look like `CONSTRAINT FOREIGN KEY (local_column) REFERENCES other_table (other_table_id)`

`ALTER TABLE` lets you change existing tables' columns or constraints. It may be useful if you're scripting DB creation, like in the case of adding all your constraints last to prevent conflicts during script runtime. Altering a table will fail if there is data in the table that would conflict with the alter statement.

`DROP TABLE` removes the table you specific, and all the data it holds. **BEWARE.** Dropping a table will fail if there are constraint that would conflict with the table being gone, i.e. foreign key constraints.

Adding a column looks like

```
  ALTER TABLE local_table ADD new_column INTEGER;
  -- or to add multiple columns
  ALTER TABLE local_table
          ADD (first_column INTEGER,
              second_column VARCHAR(128));
  -- To change an existing column, use MODIFY instead of ADD
  -- To remove a column, use DROP instead of ADD
```
