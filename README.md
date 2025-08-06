### :star: Important SQL Queries

1. Count of Employee in each Department.
2. Nth Largest Salary of Employee.
3. Highest Salary in each Department.
4. Organization Hierarchy.
5. Deleting Duplicate Records from a Table.
6. Convert Rows into Coloumns by using Pivot.
7. Pagination in SQL.

### ****************************************************

### 1. Count of Employee in each Department.

```sql

create table Employee(EmpId int primary key, EmpName varchar(50), DeptId int, Salary numeric(18,2), Manager int);

create table Department(DeptId int primary key, DeptName varchar(50))

```
-------------------------------------------------------------------------

```sql

insert into Employee values (3, 'Anjali', 2, 45000, 5);
insert into Employee values (4, 'Ram', 3, 30000, 1);
insert into Employee values (5, 'Dinesh', 1, 45000, 7);
insert into Employee values (6, 'Ramesh', 2, 35000, 5);
insert into Employee values (7, 'Arjun', 3, 25000, 5);
insert into Employee values (8, 'Sanjay', 1, 32000, 1);
insert into Employee values (9, 'Kunal', 2, 40000, 3);
insert into Employee values (10, 'Raju', 3, 37000, 7);

```
-------------------------------------------------------------------------

```sql
select * from Employee
```
-------------------------------------------------------------------------

```sql

select d.DeptName as DeptName, EmpCount
from(
select DeptId, count(*) EmpCount from Employee group by DeptId
) emp join Department d on emp.DeptId=d.DeptId

```
-------------------------------------------------------------------------

### 2. Nth Largest Salary of Employee.

```sql

;WITH cte AS
(
select DENSE_RANK() over (order by Salary desc) AS RowNumber, EmpId, EmpName, Salary, DeptId from Employee
)
select top (1) cte.EmpId, cte.EmpName, cte.Salary, d.DeptName from CTE inner join Department d 
on cte.DeptId=d.DeptId 
WHERE RowNumber=5

```
-------------------------------------------------------------------------

### 3. Highest Salary in each Department.

```sql

;WITH cte AS
(
select DENSE_RANK() over (partition by DeptId order by Salary desc) AS RowNumber, EmpId, EmpName, Salary, DeptId from Employee
)
select cte.EmpId, cte.EmpName, cte.Salary, d.DeptName from CTE inner join Department d 
on cte.DeptId=d.DeptId 
WHERE RowNumber=1

```
-------------------------------------------------------------------------

### 4. Organization Hierarchy.

```sql

select e.EmpId, e.EmpName, m.EmpName as Manager from Employee e
inner join Employee m on e.Manager=m.EmpId 

```
-------------------------------------------------------------------------

### 5. Deleting Duplicate Records from a Table.

```sql

CREATE TABLE Person (Pid INT, Pname VARCHAR(50))

-------------------------------------------------------------------------

insert into Person values (1,'Amit')
insert into Person values (2,'Rohit')
insert into Person values (3,'Mohit')
insert into Person values (4,'Anjali')
insert into Person values (1,'Amit')
insert into Person values (2,'Rohit')
insert into Person values (1,'Amit')
insert into Person values (2,'Rohit')

-------------------------------------------------------------------------

select * from Person;

-------------------------------------------------------------------------

;with cte as
(
 select ROW_NUMBER() over (partition by Pid order by Pid) as RowNumber, Pid, Pname from Person
)
delete from cte where RowNumber>1

```
-------------------------------------------------------------------------

### 6. Convert Rows into Coloumns by using Pivot.

```sql

create table Products (id int, name varchar(50), devicetype varchar(50))

-------------------------------------------------------------------------

insert into Products values(1, 'Apple', 'TV')
insert into Products values(2, 'Apple', 'Tablet')
insert into Products values(3, 'Apple', 'SmartPhone')
insert into Products values(4, 'Samsung', 'TV')
insert into Products values(5, 'Samsung', 'Tablet')
insert into Products values(6, 'Samsung', 'SmartPhone')
insert into Products values(7, 'LG', 'TV')
insert into Products values(8, 'LG', 'Refrigenator')
insert into Products values(9, 'LG', 'SmartPhone')
insert into Products values(10, 'Sony', 'TV')
insert into Products values(11, 'Sony', 'Camera')

-------------------------------------------------------------------------
select * from Products

-------------------------------------------------------------------------
select Name, DeviceType1, DeviceType2, DeviceType3 from
(
 select Name, DeviceType, 'DeviceType'+Cast(row_number() over (partition by name order by name) as varchar(50)) as DeviceNumber from Products
) result
pivot
(
max(DeviceType)
for DeviceNumber in (DeviceType1, DeviceType2, DeviceType3)
) pvt

```
-------------------------------------------------------------------------

### 7. Pagination in SQL Stored Procedure.

```sql
select * from Employee

-------------------------------------------------------------------------

--exec GetEmployeeData 2, 5

CREATE PROCEDURE GetEmployeeData
    @PageNumber INT,
    @PageSize INT,
    @OrderByColumn NVARCHAR(128) = 'EmpName', -- Default column to order by
    @SortOrder NVARCHAR(4) = 'ASC' -- Default sort order
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @OffsetRows INT = (@PageNumber - 1) * @PageSize;

    -- Dynamic SQL for sorting
    DECLARE @SQL NVARCHAR(MAX);
    SET @SQL = N'
        SELECT
            *
        FROM
            Employee
        ORDER BY
            ' + QUOTENAME(@OrderByColumn) + ' ' + @SortOrder + '
        OFFSET @OffsetRows ROWS
        FETCH NEXT @PageSize ROWS ONLY;
    ';

    EXEC sp_executesql @SQL,
        N'@OffsetRows INT, @PageSize INT',
        @OffsetRows = @OffsetRows,
        @PageSize = @PageSize;

    -- Optional: Get total record count for UI pagination
    SELECT COUNT(*) AS TotalRecords FROM Employee;

END;

```
-------------------------------------------------------------------------
