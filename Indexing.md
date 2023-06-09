# Project 1: [Indexing]
- Project topic is **_Indexing_**
- Complete project [Indexing.sql](https://github.com/xkyleann/Databases_SQL_Portfolio/tree/main/Indexing)

## Useful SQL Commands 
  - The **ANALYZE** command collects statistics of value distribution in table columns and stores them in the pg_statistic system catalog. The planner will only be able to determine an optimal execution plan if the ANALYZE command was executed for a given table and its columns significant for the query.
  - The **EXPLAIN** command displays the query execution plan. Using the **_VERBOSE_** option will show more details. The **_ANALYZE_** option automatically executes **_ANALYZE_** and the query itself, letting EXPLAIN display not only the plan but also the actual execution results.
  
## Projects 
- Exercise for Indexing with SQL. 
- Includes 8 exercises including answers.
##### **Task 1: Hash-Based Indexes** 
- Execute a query that displays all orders for the buk1 composition. Check the execution plan and take note of the execution times.
- Add a hash-based index that can accelerate the aforementioned query to the orders table. Take note of the plan and execution plan.
- **_Solution_** for **_Task 1_**, should be like this: 
```SQL
exercise1.txt -- Execute a query that displays all orders for the buk1 composition. Check the execution plan and take note of the execution times.
SELECT * FROM all_orders WHERE compositions_no = 'buk1'; -- compositions_no also refers composition_id
CREATE INDEX compositions_in ON orders USING hash (compositions_no);
EXPLAIN ANALYZE SELECT DESCRIPTION * FROM all_orders WHERE composition_no = 'buk1';
```
##### **Task 2: B-tree indexes**
- Delete the index created in the previous exercise and create a similar one, but using B-trees. Repeat the previous query and take a note of the results.
- Execute a query displaying orders of all compositions with IDs beginning with letters appearing before the letter 'c' in the alphabet. Is the index in use?
- Run another query, displaying the remaining orders (beginning with 'c' and so on). Is the index in use now?
- Try forcing the use of indexes by switching the enable_seqscan parameter:
```SQL
SET ENABLE_SEQSCAN TO OFF;
```
- This will "encourage" PostgreSQL to use indexes – this kind of optimisation may be useful when using mass storage with short seek times, such as SSD drives. Repeat the two queries and compare their execution plans and times.
- **_Solution_** for **_Task 2_**, should be like this: 
```SQL
exercise2.txt -- Delete the index created in the previous exercise and create a similar one, but using B-trees. Repeat the previous query and take a note of the results.
CREATE INDEX compositions_in ON all_orders (compositions_no);
EXPLAIN ANALYZE SELECT DESCRIPTION * FROM WHERE compositions_no = 'buk1';
EXPLAIN ANALYZE SELECT * FROM WHERE compositions_no < 'c' -- Execute a query displaying orders of all compositions with IDs beginning with letters appearing before the letter 'c' in the alphabet. Is the index in use?
EXPLAIN ANALYZE SELECT compositions_no from all_orders where compositions_no ='c' AND paid='f';
EXPLAIN ANALYZE SELECT * FROM all_orders WHERE compositions_no ~ '^c'; -- can be removed 
DROP INDEX compositions_in;
```
##### **Task 3: Indexes and pattern matching**
- Create an index for the remarks column and query the database for all orders with remarks beginning with the "do" string. Is the index in use?
- Delete the index and create a new one, but this time explicitly set its operator class (varchar_pattern_ops):
```SQL
CREATE INDEX orders_remarks_idx ON orders (remarks varchar_pattern_ops);
```
- Repeat the exercise and compare the results.
- **_Solution_** for **_Task 3_**, should be like this: 
```SQL
exercise3.txt -- Create an index for the remarks column and query the database for all orders with remarks beginning with the "do" string. Is the index in use?
CREATE INDEX orders_remarks ON orders (remarks);
EXPLAIN ANALYZE SElECT * FROM orders WHERE remarks LIKE 'do%';
DROP INDEX orders_remarks;
CREATE INDEX orders_remarks_idx ON orders (remarks varchar_pattern_ops);
DROP INDEX orders_remarks;
```
##### **Task 4:  Multi-column indexes**
- Create a multi-column index covering the **_client_id, recipient_id and composition_id_** columns.
- Choose one value existing in these columns and run:
  - a query which joins the constraints for the three columns using the           **_AND_** operator,
  - a similar query, but using **_OR_**
- Compare the execution plans. 
- Now query the database for all orders of the **_buk1_** composition.
- Remove the index and create separate indexes for each of the three columns. Re-run the previous queries and compare the results.
- **_Solution_** for **_Task 4_**, should be like this: 
```SQL
exercise4.txt -- Create a multi-column index covering the client_id, recipient_id and composition_id columns.
CREATE INDEX multi_column_index ON orders (client_id, recipient_id, composition_id);
EXPLAIN ANALYZE SELECT * FROM orders WHERE client_id = 'msowins' AND recipient_id = 1 AND composition_id = 'buk1';
EXPLAIN ANALYZE SELECT * FROM orders WHERE client_id = 'msowins' OR recipient_id = 1 OR composition_id = 'buk1';
EXPLAIN ANALYZE SELECT * FROM orders WHERE composition_id = 'buk1';
DROP INDEX multi_column_index;
CREATE INDEX composition_index ON orders (composition_id);
CREATE INDEX recipient_index ON orders (recipient_id);
CREATE INDEX client_index ON orders (client_id);
EXPLAIN ANALYZE SELECT * FROM orders WHERE client_id = 'msowins' AND recipient_id = 1 AND composition_id = 'buk1';
EXPLAIN ANALYZE SELECT * FROM orders WHERE client_id = 'msowins' OR recipient_id = 1 OR composition_id = 'buk1';
EXPLAIN ANALYZE SELECT * FROM orders WHERE composition_id = 'buk1';
```
##### **Task 5: Indexes and sorting** 
- Run a query which returns all orders sorted by their composition ID. Is the index in use?
- Now delete the index and repeat the query. Compare result, then drop all indexes.
- **_Solution_** for **_Task 5_**, should be like this: 
```SQL
exercise5.txt -- Run a query which returns all orders sorted by their composition ID. Is the index in use?
EXPLAIN ANALYZE SELECT * FROM all_orders ORDER BY composition_id ORDER BY composition_id ASC;
DROP INDEX composition_index;
DROP INDEX recipient_id_idx;
DROP INDEX client_index;
EXPLAIN ANALYZE SELECT  all_orders, composition_id  FROM orders ORDER BY composition_id ASC;
```
##### **Task 6: Partial indexes** 
- Create an index on the client_id column, but only for orders which have been paid. Check it by retrieving all paid orders for a given client. Repeat the query for unpaid orders.
- Now calculate the sum of all unpaid orders. Is the index in use?
- **_Solution_** for **_Task 6_**, should be like this:
```SQL
exercise6.txt -- Create an index on the client_id column, but only for orders which have been paid.
CREATE INDEX  client_index ON orders (client_id) WHERE all.paid ='t';
EXPLAIN ANALYZE SELECT * FROM orders WHERE all.paid  = 't';
EXPLAIN ANALYZE SELECT * FROM orders WHERE client_id='msowins' AND all.paid='t'; 
EXPLAIN ANALYZE SELECT sum(price) FROM orders WHERE all.paid = 'f';
DROP INDEX client_index;
``` 
##### **Task 7: Indexes on expressions** 
- Create an index and write a query which retrieves clients living in a city which begins with a given string (e.g. "krak"), regardless of the case (e.g. "krak"="Krak"="KRAK", etc.).
- Check if the index runs properly.
- **_Solution_** for **_Task 7_**, should be like this:
```SQL
exercise7.txt -- Create an index and write a query which retrieves clients living in a city which begins with a given string (e.g. "krak"), regardless of the case (e.g. "krak"="Krak"="KRAK", etc.).
EXPLAIN ANALYZE SELECT * FROM all_clients WHERE city LIKE 'Krak%';
CREATE INDEX name_in ON clients (client_id);
DROP INDEX name_in;
EXPLAIN ANALYZE SELECT * FROM all_clients WHERE city LIKE 'Krak%';
``` 
##### **Task 7: Indexes on expressions** 
- Add a column called the location of type point to the orders table. Populate this column with random data within the (0,0)–(100,100) range:
```SQL
ALTER TABLE orders ADD COLUMN location point;
UPDATE orders SET location=point(random()*100, random()*100);
``` 
- Write queries which:
  - retrieve all orders within 10 units from the city center (50, 50),
  - retrieve all orders for the north-west quarter of the city.
- **_Solution_** for **_Task 8_**, should be like this:
```SQL
exercise8.txt -- Add a column called location of type point to the orders table. Populate this column with random data within the (0,0)–(100,100) range:
ALTER  TABLE orders ADD  COLUMN location point;
UPDATE all_orders SET location = point ( random ( ) * 100 , random ( ) * 100 ) ;
EXPLAIN ANALYZE SELECT * FROM all_orders ORDER BY location <-> point '(50,50)' LIMIT 10;
CREATE INDEX  location_idx ON all_orders USING gist(location);
EXPLAIN ANALYZE SELECT * FROM all_orders ORDER BY location <-> point '(50,50)' LIMIT 10;
DROP INDEX index_of_location;
ALTER TABLE all_orders DROP COLUMN location;
``` 
