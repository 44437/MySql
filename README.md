# MySql@8

---
Database modeling stages:
    
    1- Analyze
    2- Conceptual Design
    3- Logical Design

---
Schema is actually a Database since MySql@5.0.2

so:
```mysql
CREATE DATABASE my_database = CREATE SCHEMA my_database
```
---
Print function in MySql:
```mysql
SELECT 'hello world'

OR

\! echo "hello world"
```
---
Create a new mysql user:
```mysql
CREATE USER IF NOT EXISTS ercan@localhost IDENTIFIED BY "password123";
GRANT ALL PRIVILEGES ON * . * TO ercan@localhost;
GRANT CREATE, DROP, SELECT ON * . * TO ercan@localhost;
# ALL PRIVILEGES, CREATE, DROP, DELETE, INSERT, SELECT, UPDATE
FLUSH PRIVILEGES; # Immediately reflect added privileges.
SHOW GRANTS for ercan@localhost; # or '... for CURRENT_USER();'

GRANT ALL ON db_name.* TO ercan@localhost;  
GRANT ALL ON db_name.table_name TO ercan@localhost;  
GRANT EXECUTE ON db_name.procedure_name TO ercan@localhost;  

# Column specific:
GRANT SELECT (col1), INSERT (col1, col2), UPDATE (col2)
    ON db_name.table_name
    TO ercan@localhost;
```
---
Access a table:
```mysql
USE database_name;
SELECT * FROM table_name;

OR

SELECT * FROM database_name.table_name;
```
---
Create a table:
```mysql
CREATE TABLE table_name(
    id INT PRIMARY KEY AUTO_INCREMENT, # primary key is auto indexed, only one column can auto increment
    name VARCHAR(40) NOT NULL, # 40 character 
    json_column JSON # {"key": dynamic_value }
)
```
---
Get table information:
```mysql
DESCRIBE table_name;
```
---
```mysql
ALTER TABLE ... 
```
- Columns can be added to the table, first or after a certain column.
- Column type can be changed with MODIFY.
- Column can be drop in the table.
- Column name can be changed.
- Table name can be changed. // or RENAME new_table_name TO old_table_name;
---
```mysql
TRUNCATE TABLE ... 
```
- Deletes all values of the table but not the table. First the table is deleted and then recreated.
---
Copy Table:
```mysql
CREATE {TEMPORARY} TABLE IF NOT EXISTS duplicate_table SELECT * FROM original_table;
```
---
```yaml
Table Locking:
    LOCK TABLES table_name [READ | WRITE];
    READ LOCK: Without the read lock released, other sessions cannot write, but they can read.
    there is no locking in their own session, they do this for other sessions.
    WRITE LOCK: Blocks both writing and reading for other sessions.
```
---
```yaml
User Locking:
  Y: Locked
  N: Unlocked
  Locking:
  > CREATE USER IF NOT EXISTS ercan@localhost
    IDENTIFIED BY '123'
    ACCOUNT LOCK;
  > SELECT user, host, account_locked
    FROM mysql.user
    WHERE user = 'ercan' AND host = 'localhost';// returns 'Y' on account_locked column
After trying to login it returns access denied.
  Unlocking:
  > ALTER USER [IF EXISTS] 'ercan'@'localhost' ACCOUNT UNLOCK;
```
---
Insert:
```mysql
INSERT INTO table_name(name) VALUES ('Ercan');
INSERT INTO table_name('name', )
```
---
Update:
```mysql
UPDATE table_name 
    SET column_name = "new value"
    WHERE table_name.id = 123;
```
---
Sub-queries:
```mysql
SELECT * FROM product p WHERE p.name = (SELECT product_name FROM order WHERE order_id = 123 LIMIT 1);
```
---
CHECK constraints:
```mysql
    Name varchar(45) CHECK(Name!="ercan"),
    - Name should not be "ercan" otherwise you cannot add 
    
    Name VARCHAR(30) DEFAULT "ERCAN",
    - If the Name is empty, the default is used.
    
    category_id int,
    FOREIGN KEY (category_id) REFERENCES Category(id) 
        ON DELETE CASCADE 
        ON UPDATE CASCADE
```
---
```mysql
REPLACE INTO table_name SET column1 = value1, column2 = value2;
```
This command is not safe. First deletes the matching rows then replaces the values

---
INSERT IGNORE:
```mysql
INSERT INTO table_name(column_name) VALUES ("ercan"); # returns error because column_name VARCHAR(3)
INSERT IGNORE INTO table_name(column_name) VALUES ("ercan"); # done!
```

or \
if column_name is UNIQUE and we try to adding the same data, the previously added one becomes the update with new.
---
INSERT INTO SELECT:
```mysql
INSERT INTO user_info (name, email)
SELECT name, email
FROM user
WHERE age = 33;
```
---
<h3>MYSQL INDEX<h3/>
PRIMARY and UNIQUEs automatically indexed.
innoDB use the binary search;
```mysql
CREATE INDEX index_name ON employees(lastName, firstName);

we can use query optimizer when lastName in the "where".
-
we can use query optimizer when lastName and firstName in the "where".
-
we can not use query optimizer when only firstName in the "where".
-
we can not use query optimizer when lastName and firstName in the "where",
    if they are connected by "or" instead of "and".

Ex-2:
    ... ON employees(col1, col2, col3);
    query optimizer run:
        col1
        col1, col2
        col1, col2, col3,
    query optimizer not run:
        col2, col3
        col1, col3
```
DROP INDEX:
```mysql
DROP INDEX index_name ON table_name [algorithm_option | lock_option];
```
default algorithm: INPLACE. If INPLACE is not supported, it uses the COPY algorithm.

Clustered and Non-Clustered Index:
<br>
PRIMARY KEY is Clustered index and sorts the rows automatically. One in each table.
All indexes except clustered index are Non-Clustered index.
--- 
```mysql
SELECT IF(11=11, "Equal", "Not equal");
SELECT IFNULL(phone_number, "+905555555555") as FROM user;
```
---
```mysql
WHERE col1 IN (...)      =   WHERE col1 = ANY(...)
WHERE col1 <>ANY (...)   =   WHERE col1 NOT IN (...)
    -SOME = ANY-
    "<>" = "!="   
p.price > ALL(50, 60) # p.price > 50 and p.price > 60
```
---
```mysql
SELECT * FROM user INNER JOIN  user_info ON user.id = user_info.id;
                                    ...  USING(id) WHERE name IS NOT NULL;
RIGHT/LEFT [OUTER] JOIN 
SELF JOIN # To join a table with itself.
EquiJoin -> # Used when combining 3 tables. actually called EquiJoin.
NATURAL JOIN # It joins the tables based on the same column names and their data types

```
---
```mysql
WHERE [NOT] EXISTS ... # returns 1 or 0
Query will run if "EXISTS" return 1, so can run row specific
```
---
GROUP_CONCAT:
```mysql
SELECT id, name,
       GROUP_CONCAT(hobby) as "hobbies" FROM user group by id;
```
---
RLIKE = REGEXP_LIKE \
"_" -> for one char \
"%" -> includes all characters before or after the characters
---
```mysql
SELECT engine FROM information_schema.tables ... 
```
---
You can export table to CSV

---
The LAG() function is used to get value from row that precedes the current row.\
The LEAD() function is used to get value from row that succeeds the current row. 
---
CTE(Common Table Expression) : \
    The result set of a query that exists temporarily and is typically recursive and for use in large query expressions.\
    CTE exists within a transaction and then disappears\
View is a stored SQL query. It doesn't store the output of the query, it just stores the query

```mysql
WITH kids AS (  
SELECT * FROM users WHERE age < 8   
)   
SELECT name, surname, age FROM kids  
WHERE name = 'John' and surname = 'Doe' ORDER BY age;

```
---
```mysql
...
FOREIGN KEY (product_id) REFERENCES Product (id) 
    ON DELETE CASCADE
    ON UPDATE CASCADE
...
```
MySQL contains five different referential options: # default : RESTRICT, If ON DELETE ... and ON UPDATE ... values are not added\
1- CASCADE:
    Used to automatically delete or update matching records from child table when we delete or update rows from parent table\
2- SET NULL:
    It assigns NULL to the child's foreign key column\
3- RESTRICT:
    If the parent row is deleted or updated, the child row is not deleted or updated. Gives an error message. For example "The field .... in the row you are trying to delete is in use in another table". This error will appear if you delete the parent row. If you delete the child row it is attached to, you will not get an error, which is what it should be. \
4- NO ACTION:
    A keyword from standard SQL. In MySQL, equivalent to RESTRICT. It doesn't take any action. That is, even if the parent row is deleted or updated, no action is taken on the child row\
5- SET DEFAULT:
    The MySQL parser recognizes this action. However, the InnoDB and NDB tables both rejected this action

---
UPSERT:
If a record is new, it will trigger an INSERT command. But, if it already exists in the table, then this operation will perform an UPDATE statement
---
Partitioning:
1 - Horizontal Partitioning
2 - Vertical Partitioning
---
SIGNAL RESIGNAL:
Used to return a warning or error message
---
```mysql
SELECT FORMAT(13600.2021, 2, 'de_DE'); 
```
---
Prepared Statement
 * Better than normal statement.
 * Prepared statements are queries that contain the placeholders instead of actual values.
 * It can't be stored in database but we can use it with stored procedure
```mysql
PREPARE state FROM 'SELECT ?+? AS sum';
SET @a = 10;
SET @b = 20;
EXECUTE state USING @a, @b;
```
---
MINUS : first_table / second_table \
INTERSECT : first_table ∩ second_table \
COALESCE :  return the first non-null value
---
```mysql
SHOW PROCESSLIST
```
You can see the work done by users instantly

---
FUNCTIONS
 * Stored in database
 * They can only return one value
 * using: custom_trace(2,4)
 * DETERMINISTIC: if it always gives the same result when given the same values
```mysql
DELIMITER $$ # The delimiter is changed to $ to enable the entire definition to be passed to the server as a single statement, and then restored to ; before invoking the procedure. This enables the ; delimiter used in the procedure body to be passed through to the server rather than being interpreted by mysql itself.
CREATE FUNCTION custom_sum(
a INT,
b INT 
) RETURNS INT
DETERMINISTIC # or NOT DETERMINISTIC 
BEGIN 
   RETURN a+b;
END $$
DELIMITER ;
```
---
CURSORS
 * Reads the lines one by one
```mysql
DELIMITER $$
CREATE PROCEDURE step_by_step()
BEGIN 
    DECLARE orderCursor CURSOR FOR SELECT id, total_price FROM 'order';
    DECLARE id INT;
    DECLARE totalPrice FLOAT;

    OPEN orderCursor;
  
    getOrder: LOOP
        FETCH orderCursor INTO id, totalPrice;
        IF id = 20 OR totalPrice = 333.2 THEN
            ITERATE getOrder; # continue 
        ELSEIF totalPrice > 999.3 THEN
            LEAVE getOrder; # break
        END IF;
    END LOOP getOrder;
    
    CLOSE orderCursor;
END $$
DELIMITER ;
```
---
DECLARE ... HANDLER
```mysql
DELIMITER $$
CREATE PROCEDURE handle_error()
BEGIN 
    DECLARE CONTINUE HANDLER FOR SQLEXCEPTION # CONTINUE: execution continues. EXIT: terminates between "begin" and "end"
        BEGIN 
            # as catch function
        END;
    CALL get_error();
END $$
DELIMITER ;
```
---
STORED PROCEDURES
 * Increases the performance, compiled and stored in database as soon as it is created
 * Secure, the procedure can be allowed without needing permission to access the tables
EXAMPLE - 1 :
```mysql
DROP PROCEDURE IF EXISTS  get_categories; # if you don't use, you can't change the existing procedure
DELIMITER $$
CREATE PROCEDURE  get_categories()
BEGIN 
   SELECT * FROM category;
END $$
DELIMITER ;

# using:
CALL get_categories();
```
EXAMPLE - 2 :
```mysql
DROP PROCEDURE IF EXISTS get_categories;
DELIMITER $$
CREATE PROCEDURE custom_sum(
    a INT, # default is IN. INOUT: 
    IN b INT, # IN : function parameter, only get
    OUT c INT # OUT:  returned value, get and set. INOUT is like this
)
BEGIN 
    SELECT a+b INTO c;
END $$
DELIMITER ;

# using:
SET @total = 0;
CALL custom_sum(3, 3, @total);
SELECT @total; # print 6
```
---
VARIABLES
- <b> User-Defined Variable Assignment: </b> User-defined variables are created locally within a session and exist only within the context of that session
```mysql
SET @price := 10;
--
SET @price = 20;
--
SELECT @price := MAX(product.price)
FROM product;
```
- <b> Parameter and Local Variable Assignment:</b> SET applies to parameters and local variables in the context of the stored object within which they are defined
```mysql
...
CREATE PROCEDURE ...
BEGIN
    DECLARE counter INT DEFAULT 0;
    SET counter = counter + 10; # 0 + 10
END;
...
```
- <b>System Variable Assignment: </b> 
  * GLOBAL VARIABLE
  * LOCAL VARIABLE for user session
```mysql
SET GLOBAL max_connections = 1000;
SET @@GLOBAL.max_connections = 1000;
--
SET SESSION sql_mode = 'TRADITIONAL';
SET LOCAL sql_mode = 'TRADITIONAL';
SET @@SESSION.sql_mode = 'TRADITIONAL';
SET @@LOCAL.sql_mode = 'TRADITIONAL';
SET @@sql_mode = 'TRADITIONAL';
SET sql_mode = 'TRADITIONAL';
# All of the above are the same
```
---
TRIGGER: https://dev.mysql.com/doc/refman/8.0/en/trigger-syntax.html
 * It is run before (BEFORE) or after (AFTER) the INSERT, UPDATE, and DELETE operations.
 * If you drop a table, any triggers for the table are also dropped.
 * There should be no action on the same table in the trigger, otherwise it will be an infinite loop. And triggers should be created carefully to avoid circular loop
 * Within the trigger body, the OLD and NEW keywords enable you to access columns in the rows affected by a trigger. OLD and NEW are MySQL extensions to triggers; they are not case-sensitive.
   - In an INSERT trigger, only NEW.col_name can be used; there is no old row.
   - In a DELETE trigger, only OLD.col_name can be used; there is no new row.
   - In an UPDATE trigger, you can use OLD.col_name to refer to the columns of a row before it is updated and NEW.col_name to refer to the columns of the row after it is updated.
Example:
```mysql
DELIMITER $$
CREATE TRIGGER product_update_stock AFTER INSERT ON order_detail FOR EACH ROW 
BEGIN 
    UPDATE product 
    SET stock_qty = stock_qty - NEW.order_qty
    WHERE product.id = NEW.product_id;
END $$
DELIMITER ;
```
---
EVENTS(Cron Job on MySql)
```mysql
CREATE EVENT optimizer
ON SCHEDULE # AT timestamp is used for a one-time event.
    EVERY 24 HOUR
        STARTS # STARTS '2022-01-01' ENDS '2023-01-01'
        DO OPTIMIZE TABLE 'order'; # That can be used to reclaim unused space
```
---
TRANSACTION
 * It has the ability to undo operations made in it.
 * It is only supported in innoDB(for all standard types) and thus complies with the ACID database design principle. 
   * A tomic : They can not be disintegrated in a distress all operations are undone
   * C onsistency : Database is a whole
   * I solation : Operations are processed sequentially
   * D urability : Relationship between physical resource and Mysql software

```mysql
START TRANSACTION;
# queries
ROLLBACK; # you can undo changes if errors occur
# queries
COMMIT; # to permanently apply operations to tables
```
 * Autocommit is enabled on mysql, to disable autocommit mode:
   * 1- For a single series of statements:
     * SELECT @total:=SUM(salary) FROM employee WHERE type=1; 
   * 2- Can be written between begin-end
   * 3- SET autocommit=0;
   * 4- Add autocommit=0 to my.cnf file
 
 * Transaction Isolation Levels: # default transaction isolation level is Repeatable Read.
   * can solve:     +
   * can not solve: - 

| Isolation Levels \ Concurrency Problems | Dirty Reads | Lost Updates | Non-Repeatable Reads | Phantom Reads |
|:-----------------------------------------|-------------|--------------|:---------------------|:--------------|
| READ UNCOMMITTED                         | -           | -            | -                    | -             |
| READ COMMITTED                           | +           | -            | -                    | -             |
| REPEATABLE READ                          | +           | +            | +                    | -             |
| SERIALIZABLE                             | +           | +            | +                    | +             |

REPEATABLE READ : It takes a copy of the current output of a query and uses it when called again, disregarding the changes of other sessions. This copy job degrades performance.
SERIALIZABLE : In a session, it locks the rows used. In other sessions, these rows can only be read, not added or updated.
 * Dirty Reads : You can read uncommitted variables in another session. Afterwards, that variable may rollback, but the other session continues as it was.
 * Non-Repeatable Reads : Session A changes a data to 30 without committing, B accesses this data and reads 20, then after A commits, when B accesses this data again, this time it reads as 30
 * Lost Updates : Data a is now 0, increased by 1 unit in one session, and increased by 1 unit in the other session at the same time. The scenario where I see 1 when I should normally see 2.
 * Phantom Reads : I get the total number of products, another session deletes one of the products or adds a new one. So that product is phantom :)
```mysql
SET [PERSIST|GLOBAL] TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```
---
FULLTEXT SEARCH
 * Full-text indexes can be created only for VARCHAR, CHAR or TEXT columns. 
 * There are three types of full-text searches :
   * Natural Language Full-Text Searches
   * Boolean Full-Text searches
   * Query expansion searches
   <br>
   Natural Language Full-Text Searches:
     * Default search type
     * The rows returned are automatically sorted with the highest relevance first.
     ```mysql
     SELECT * FROM table_name WHERE MATCH(col1, col2) AGAINST('search terms' IN NATURAL LANGUAGE MODE)
     # Returns lines containing "search", "terms" and "search terms" as case insensitive. By default, the search is case-insensitive
     ```
     * How to retrieve the relevance values explicitly:
     ```mysql
       SELECT id, MATCH(col1,col2) AGAINST ('search terms' IN NATURAL LANGUAGE MODE) AS score FROM table_name;
     ``` 
   <br>  
   Boolean Full-Text searches
   Example: # +, -, "", ~ etc. 
   
   ```mysql
        SELECT * FROM articles WHERE MATCH (title,body) AGAINST ('+MySQL -YourSQL' IN BOOLEAN MODE);
        # Get those that definitely contain the word "MySQL" and those that definitely don't contain the word "YourSQL"
   ```
   <br>

   Query expansion searches 
   [Dev MySql](https://dev.mysql.com/doc/refman/8.0/en/fulltext-query-expansion.html) 
   <br>
   Could be searching for books by Georges Simenon about Maigret, when a user is not sure how to spell “Maigret”. A search for “Megre and the reluctant witnesses” finds only “Maigret and the Reluctant Witnesses” without query expansion. A search with query expansion finds all books with the word “Maigret” on the second pass.
   ```mysql
        SELECT * FROM table_name WHERE MATCH (title,body) AGAINST ('search terms' WITH QUERY EXPANSION);
    ```
---
 SQL Normalisation
 * UNF : Un-normalized Form 
 * 1NF : 1st Normal Form
 * 2NF : 2nd Normal Form
 * 3NF : 3rd Normal Form
 * BCNF : Boyce-Codd(3.5NF)
 * 4NF 
 * 5NF 

1NF:
* Repeating columns cannot be found in the same table.
* Each column can have only one value (don't combine commas and data).

2NF:
* Database must be 1NF
* There must be a column that holds a unique value for each row.
* Any subset of data should not be repeated in more than one row. New tables must be created for such subsets of data.
* Relationships should be defined between main tables and new tables by using foreign keys.

3NF:
* Database must be 2NF
* No non-key column should be dependent on one another or have any transitional functional dependency. In other words, each column must be fully dependent on the key.
  - There is a col1 column and a col2 column; col1=10, col2=20, col3 column as total should not be 10+20=30  
---
