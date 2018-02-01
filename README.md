SQL Notes
==========

### Select top 3 without TOP/LIMIT/ROWNUM

Top 3 Quantity From OrdersDetails:
https://www.w3schools.com/sql/trysql.asp?filename=trysql_select_all

```sql
SELECT Quantity 
FROM OrderDetails a
where (select count(*) from OrderDetails b where b.Quantity >= a.Quantity) <= 3
order by Quantity desc
```
or 

```sql
SELECT Quantity, (select count(*) from OrderDetails b where b.Quantity >= a.Quantity) as rnk 
FROM OrderDetails a
where (select count(*) from OrderDetails b where b.Quantity >= a.Quantity) <= 3
order by Quantity desc
```
Note: 
  - COUNT acts on WHERE b.Quantity >= a.Quantity which is also aggregation condition 
  - Table self-join to be as a condition
  - Using select to create a new column
  - The order of b.Quantity > a.Quantity
  - Pay attention to '>=' or '>'
  - No need to use “ORDER BY” when using b.Quantity >= a.Quantity

### Second Largest 

Using **LIMIT**

```sql
SELECT number
FROM (
SELECT number FROM Table
ORDER BY number desc limit 2
) ORDER BY number limit 1
```

or Using **TOP**

```sql
SELECT TOP 1 salary   
FROM (  
SELECT TOP 2 salary  
FROM employee_table  
ORDER BY salary DESC) AS emp  
ORDER BY salary ASC;
```

or Using **LIMIT** and **OFFSET**
```sql
SELECT OrderID, Quantity FROM OrderDetails
Order By Quantity desc
Limit 1 Offset 1
```

Note:
  - **LIMIT/TOP/OFFSET** are not always available; they are not basis SQL commands. 

Using **max()** and **not in ()**

```sql
select max(Quantity) from OrderDetails
where Quantity not in (select max(Quantity) from OrderDetails)
```

Notes:
  - **column IN (1,2,3)** / **column NOT IN (select id from table)**

Using **self join**

```sql
select Quantity from OrderDetails a
where (select count(*) from OrderDetails b where b.Quantity > a.Quantity) = 1
```

or 
```sql
SELECT OrderID, Quantity FROM OrderDetails a
where (select count(*) from OrderDetails b where b.Quantity >= a.Quantity) = 2
```

### Maximun in each group

Using **rank() over()**

```sql
RANK() OVER (PARTITION BY column_ID ORDER BY column_ID) AS rank_id
```

Maximun Quantity in each of the *OrderID*:
https://www.w3schools.com/sql/trysql.asp?filename=trysql_select_all

```sql
SELECT OrderDetailID, OrderID, max(Quantity) FROM OrderDetails
group by OrderID
```

Note: 
  - **MAX** acts on each of the group, similarly as **MIN**, **SUM**, etc. 
  
### Between Date

*OrderID* between *'1996-09-02'* and *'1996-09-06'*:
https://www.w3schools.com/sql/trysql.asp?filename=trysql_select_all

```sql
SELECT OrderID, OrderDate FROM Orders
WHERE OrderDate BETWEEN '1996-09-02' AND '1996-09-06'
```

Note:
  - **BETWEEN ... AND ...** is inclusive.

or 

```sql
SELECT OrderID, OrderDate FROM Orders
WHERE OrderDate >= '1996-09-02' AND OrderDate <= '1996-09-06'
```

### Left Join

Left Join **Orders** on **Shippers**
https://www.w3schools.com/sql/trysql.asp?filename=trysql_select_all

```sql
SELECT a.OrderID, a.ShipperID, b.ShipperName FROM Orders a
Left join Shippers b
on a.ShipperID = b.ShipperID
```

Note:
  - **left join** fills all the records in “left table” with matches on “right table”
  

Left Join **Shippers** on **Orders**
https://www.w3schools.com/sql/trysql.asp?filename=trysql_select_all

```sql
SELECT a.ShipperID, ShipperName, OrderID, OrderDate
FROM Shippers a
LEFT JOIN Orders b
ON a.ShipperID = b.ShipperID
```

Note:
  - If there are multiply copies in the ‘right table’ matching the ‘left table’. The ‘left table’ matched item will duplicate to have multiply copies.
  - There is no difference between ‘left join’ and ‘join’ if there is no ‘missing’ on the join condition element(s) in the two tables. The following three queries return the same results (order may be different).
  - ```sql
	SELECT a.ShipperName, CustomerID, EmployeeID, OrderID FROM Shippers a
	join Orders b
	on a.ShipperID = b.ShipperID
    ```
  - ```sql
	SELECT a.ShipperName, CustomerID, EmployeeID, OrderID FROM Shippers a
	Left join Orders b
	on a.ShipperID = b.ShipperID
    ```
  - ```sql
	SELECT b.ShipperName, CustomerID, EmployeeID, OrderID FROM Orders a
	Left join Shippers b
	on a.ShipperID = b.ShipperID
    ```

### The trick of **Left Join** and **IS NULL**

找出A table里的user_id且该user_id不在table B里

```sql
select A.id
from A 
left join B on A.id = B.id
where B.id IS NULL;
```
Note:

  - Use **IS NULL** instead of **=NULL**

### String Match

Write an SQL query to find names of employee start with 'A'?

```sql
SELECT * FROM Employees WHERE EmpName like 'A%'
```

or end with 'A'
```sql
SELECT * FROM Employees WHERE EmpName like '%A'
```

or 'A' in the second position
```sql
SELECT * FROM Employees WHERE EmpName like '_A%'
```

Note:
  - '%' is used as a wild card with 'like'
  - '_' is used as one letter with 'like'
  
### Regular Expression

SQL regular expression very similar to python regular expression.

One example, find *LastName* begins with capital D:
```sql
SELECT * FROM Employees
where LastName regexp '^D'
```
**This one does not work in w3schools**

### Where vs Having

Select the *EmployeeID* with the number of orders larger than 10 from **Orders**
https://www.w3schools.com/sql/trysql.asp?filename=trysql_select_all

```sql
SELECT EmployeeID, count(OrderID) FROM Orders
group by EmployeeID
having count(OrderID) > 10
```

Note:
  - “WHERE” vs “HAVING”: The WHERE clause cannot be used to restrict groups. The HAVING clause should be used.
  - “HAVING” need to be used after ‘GROUP BY’
  
#### 找出EmployeeID和所有对应的OrderID，EmployeeID有多于25的OrderID
https://www.w3schools.com/sql/trysql.asp?filename=trysql_select_all

```sql
SELECT a.EmployeeID, a.OrderID, b.cnt 
FROM Orders a
left join (
select EmployeeID, count(OrderID) as cnt
from Orders 
group by EmployeeID
) b
on a.EmployeeID = b.EmployeeID
where b.cnt > 25
Order by a.EmployeeID
```
or,

```sql
Select a.EmployeeID, a.OrderID From Orders a
Where a.EmployeeID in (
  Select b.EmployeeID From Orders b
  Group by b.EmployeeID
  Having count(b.OrderID) > 25
)
Order by a.EmployeeID, a.OrderID
```
  
### Most common elements in a column

The most frequent *Quantity* in **OrderDetails**
https://www.w3schools.com/sql/trysql.asp?filename=trysql_select_all

```sql
select Quantity, max(cnt) from (
SELECT Quantity, count(Quantity) as cnt FROM [OrderDetails]
group by Quantity) a
```

Second most frequent *Quantity* in **OrderDetails**
```sql
select Quantity, max(cnt) from (
SELECT Quantity, count(Quantity) as cnt FROM [OrderDetails]
group by Quantity) a
where cnt not in (
select max(cnt) from (
SELECT Quantity, count(Quantity) as cnt FROM [OrderDetails]
group by Quantity)
)
```
or
```sql
select Quantity, cnt from (
SELECT Quantity, count(Quantity) as cnt FROM [OrderDetails]
group by Quantity) a
order by cnt desc
limit 1 offset 1
```

### Accumulative Sum 

Get the first *OrderDetailID* from which the accumulative number of orders larger than 80 from **OrderDetails**
https://www.w3schools.com/sql/trysql.asp?filename=trysql_select_all

```sql
select min(OrderDetailID), accsum from (
SELECT OrderDetailID, (select sum(Quantity) from OrderDetails b where b.OrderDetailID <= a.OrderDetailID) as accsum FROM OrderDetails a
)
where accsum > 80
```

Note:
  - Using self join
  - Using sum()
  - Pay attention to *<* or *<=* depending on question
  
### Count in range

Get the number of OrderDetailID with Quantity in 1-5, 6-10, 11-15, ... from OrderDetails
https://www.w3schools.com/sql/trysql.asp?filename=trysql_select_all

```sql
Select (grp+1)*5 as Range, count(OrderDetailID), sum(Quantity) from
(SELECT OrderDetailID, Quantity, ((Quantity - 1) / 5) as grp FROM [OrderDetails])
group by grp
```

Note: 
  - GROUP BY does not work on newly formed names, instead use the whole formula
  - Floor(Amount/50) or Ceil(Amount/50)
  
### Join After Group By

Compute average order / clicks for each visitor?

**click_data**

| visitor | dt | impression | clicks | test_variant |
| ------- | -- | ---------- | ------ | ------------ |
| a | 6/1 | 100 | 1 | control |
| b | 6/1 | 20 | 5 | control |
| ... | ... | ... | ... | ... |
| aaa | 6/1 | 200 | 1 | treatment |
| bbb | 6/1 | 40 | 5 | treatment |

**purchase_data**

| visitor | dt | orders | $ | test_variant |
| ------- | -- | ------ | - | ------------ |
|     aa    |   6/2   |     1    |    100   |   control      |
|     bb    |   6/3   |     2    |    500   |   control      |
|    ...    |   ...   |    ...   |    ...   |    ...         |
|    aaaa   |   6/5   |     3    |   1000   |   treatment    | 
|    bbbb   |   6/10  |     1    |   200    |   treatment    |

```sql
select a.visitor, s_orders/sum(a.clicks) as each_avg
from click_data a
left join (
select visitor, sum(orders) as s_orders
from purchase_data b
group by visitor
) b
on a.visitor = b.visitor
group by a.visitor, s_orders
``` 

Note:
  - **sum() / sum()**, get one **sum()** in aggregation join sub query and then do another **sum()** in the main **select**

### Case ... When ...

```sql
select top 100 date, 
count(*) as count_all,
sum (
  case 
    when content not like '%some condition%' then 1
	when content like '%some condition%' then -1
    else 0
  end
) as count_condition
from sections
group by date
order by date
```

Note:
  - Use ‘Case When … Then …’ when need to count two different conditions from one table
  
### cast( VAR as float)

给一个表有四个column：time, user_id, app_id, event ('imp' or 'click') 。大致意思：当一个user在玩一个app时，有一定几率会弹出一个窗口让user 添加信用卡（event "imp"），如果这个user按了 yes，那就是一个 click的event如何来判定这个弹窗口的效果？（click through rate），如何看哪个app click through rate最高，如何知道这个table的信息是否正确 （比如user 按了一次 yes，结果自动生成多次 click event）

```sql
select app_id, avg(rate) from (
select user_id, app_id, cast(count(case when event = 'click' then 1 else 0 end) as float) / cast(count(case when event = 'imp' then 1 else 0 end) as float) as rate
from table) sub
group by app_id
```

### Write an SQL query that makes recommendations using the pages that your friends liked. Assume you have two tables: a two-column table of users and their friends, and a two-column table of users and the pages they liked. It should not recommend pages you already like.

```sql 
select f.userid, l.pageid
from friends f
JOIN likes l ON l.userid = f.friendid
LEFT JOIN likes r ON (r.userid = f.userid AND r.pageid = l.pageid)
where r.pageid IS NULL;
```

Note: 

  - The trick of using **IS NULL** 



