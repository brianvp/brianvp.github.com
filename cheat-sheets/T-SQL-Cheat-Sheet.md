---
title: T-SQL Cheat Sheet
layout: page
---

### Determine if object exists


```sql
IF OBJECT_ID('product.Model', 'U') IS NOT NULL
```

### add Check Constraint


```sql
ALTER TABLE dbo.MyTable
    ADD CONSTRAINT CHK_dbo_MyTable_Value
    CHECK (VALUE > 0.00)
```
    
    
### T-SQL Processing Order



1. From
2. Where
3. Group by
4. Having
5. Select
6. Order By


### Window Function


```sql
SELECT PurchaseOrderID, ItemCode, subtotal,
 ROW_NUMBER() OVER ( PARTITION BY PurchaseOrderID ORDER BY ItemCode) AS rownum,
 SUM(subtotal) OVER (PARTITION BY PurchoseOrderID) AS purchaseOrderTotal
FROM pur.PurchaseOrderDetail
```

### Casting


```sql
SELECT CAST('12345' AS NUMERIC(12,2))
```

### Case Expressions


```sql
SELECT Vendor_Code =
        CASE
            WHEN VendorItemCode IS NULL THEN ''
            WHEN LEN(VendorItemCode) > 10 THEN LEFT(VendorItemCode, 10)
            ELSE VendorItemCode
        END    
FROM pur.PurchaseOrderDetail
```

### String Functions


```sql
select LEN('sql server') -- 10
select CHARINDEX('e','sql server') -- 6
select PATINDEX('%serv%', 'sql server') -- 5
select REPLACE('sql server', 'sql', 'cookie') -- cookie server
select REPLICATE('sql', 4) -- sqlsqlsqlsql
select STUFF('sql server', 1, 0, 'Microsoft ') -- Microsoft sql server
```

### Wildcards


```sql
SELECT ManufacturerCode
FROM  product.Model
WHERE ManufacturerCode
LIKE 'PG-42%' --PG-42445-01 PG-42600-02
LIKE '%G-42%'--  PG-42445-01 RG-42900-03
LIKE 'RG-_____-__' -- RG-85000-01 RG-42900-03
LIKE 'RG-[8-9]____-__' --  RG-85000-01, RG-95000-01
LIKE '[O-Z]G%' -- RG, PG, but not AG, FG, etc.  
```

### Date Functions


```sql
select GETDATE() -- 2014-01-17 07:45:59.730
select DATEADD(year, 1, getdate()) --2015-01-17 07:45:59.730
select DATEADD(month, 1, getdate())-- 2014-02-17 07:45:59.730
select DATEADD(day, 1, getdate()) -- 2014-01-18 07:45:59.730

select DATEDIFF(year,  '20130101', '20131024') -- 0
select DATEDIFF(month,  '20130101', '20131024') -- 9
select DATEDIFF(day,  '20130101', '20131024') -- 296

select DATEPART(year, getdate()) -- 2014
select DATEPART(month, getdate()) -- 1
select DATEPART(day, getdate()) -- 17

select YEAR(GETDATE()) -- 2014
select MONTH(GETDATE()) -- 1
select DAY(getdate()) -- 17

select DATENAME(month, getdate()) -- January
select DATENAME(DAY, GETDATE()) -- 17

select ISDATE('20130101') - 1
select ISDATE('20139999') - 0
```

### Metadata Queries


```sql
USE BikeStore
GO

SELECT SCHEMA_NAME(SCHEMA_ID) AS table_schema_name, name AS table_name FROM sys.tables ORDER BY table_schema_name, table_name


SELECT name
FROM sys.columns
WHERE OBJECT_ID = OBJECT_ID('product.Category')


SELECT * FROM INFORMATION_SCHEMA.TABLES
SELECT * FROM INFORMATION_SCHEMA.TABLE_PRIVILEGES -- table specific privileges granted to accounts
SELECT * FROM INFORMATION_SCHEMA.VIEW_TABLE_USAGE -- tables referenced by views

EXEC sys.sp_tables
EXEC sys.sp_help @objname = N'product.Model' -- returns general info about the object


SELECT SERVERPROPERTY('ProductLevel') -- current value is 'SP1'
SELECT SERVERPROPERTY('Edition') -- Standard Edition (64-bit)
SELECT @@VERSION -- Microsoft SQL Server 2008 R2 (SP1) - 10.50.2550.0 (X64)   Jun 11 2012 16:41:53   Copyright (c) Microsoft Corporation  Standard Edition (64-bit) on Windows NT 6.1 <X64> (Build 7601: Service Pack 1) (Hypervisor)
SELECT DATABASEPROPERTYEX('enterprise', 'Collation') -- SQL_Latin1_General_CP1_CI_AS
SELECT OBJECTPROPERTY(OBJECT_ID('product.PartNumber), 'TableHasPrimaryKey') -- 1
```

### CROSS JOIN


```sql
CROSS JOIN
    DECLARE @digits TABLE (digit INT)
    INSERT INTO @digits (digit) values (1),(2), (3)
                    
    SELECT  d2.digit, d1.digit -- returns 9 record result
        FROM @digits d1
            CROSS JOIN @digits d2
```
          
### Derived Tables


```sql
    SELECT SalesYear, SUM(LineTotal) AS TotalSold
    FROM (
            SELECT YEAR(DateSold) AS SalesYear, LineTotal
            FROM sales.SkuSales
         ) SalesByYear
    GROUP BY SalesYear
    ORDER BY TotalSold DESC
```
  
### Common Table Expression's  / CTE's


```sql
    WITH <CTEname> (<column1>, <column2>) -- column list optional
    as
    (
        <subquery>
    )
    <outer query>


    WITH SalesByYearCTE ( SalesYear, LineTotal ) -- list of columns optional
    AS
    (
        SELECT YEAR(DateSold) AS SalesYear, LineTotal 
            FROM sales.SkuSales
    ) -- to add more CTE's, add a comma here
    SELECT SalesYear, SUM(LineTotal) AS TotalSold
    FROM SalesByYearCTE
    GROUP BY SalesYear
    ORDER BY TotalSold DESC
```
	
### Recursive CTE 

 useful for querying tables that are self-referencing

```sql
    declare @Location_Table table
        (
            Location_ID int,
            Location_Name varchar(25),
            Location_Parent int null
        )

        insert into @Location_Table (Location_ID, Location_Name, Location_Parent)
        select 1, 'United States', null union all
        select 2, 'Iowa', 1 union all
        select 3, 'South Dakota', 1 union all
        select 4, 'Minnesota', 1 union all
        select 5, 'Nebraska', 1 union all
        select 6, 'Orange City', 2 union all
        select 7, 'Sioux Center', 2 union all
        select 8, 'Hospers', 2 union all
        select 9, 'Sioux Falls', 3 union all
        select 10, 'Brookings', 3 union all
        select 11, '102 Michigan Ave SW', 6 union all
        select 12, '412 4th St SE', 6 union all
        select 13, 'Utility Room', 11 union all
        select 14, 'Kitchen', 11 union all
        select 15, 'Chest Freezer', 13;


        with Location_CTE as
        (
            -- anchor
            select location_ID, location_Name, Location_Parent
            from @Location_Table where Location_ID = 2 --  iowa
            
            union all
            
            -- recurse
            select lt.Location_ID, lt.Location_Name, lt.Location_Parent
            from Location_CTE lc
                join @Location_Table lt
                on lt.Location_Parent = lc.Location_ID
        )
        select Location_ID, Location_Name,
        (select Location_Name from @Location_Table where Location_ID = Location_CTE.Location_Parent) as location_parent_name
        from Location_CTE;
```

### Correlated Queries

```sql
    -- inside WHERE clause
    SELECT TOP 1000 BillOfMaterialsID, BillOfMaterialsRevisionID, Quantity
    FROM manufacturing.BillOfMaterials bom_outer
    WHERE BillOfMaterialsID =
    (
        SELECT TOP 1 BillOfMaterialsID
        FROM manufacturing.BillOfMaterial  bom_inner
        WHERE bom_inner.BillOfMaterialsRevisionID = bom_outer.BillOfMaterialsRevisionID -- outer reference
        ORDER BY Quantity DESC
    )
    
    -- inside SELECT clause
    SELECT BillOfMaterialsID, BillOfMaterialsRevisionID, MaterialID, Quantity,
        (Amount / (SELECT SUM(Quantity) FROM manufacturing.BillOfMaterials bom_inner WHERE bom_inner.BillOfMaterialsRevisionID  = bom_outer.BillOfMaterialsRevisionID) ) * 100
            AS percent_of_recipe
    FROM manufacturing.BillOfMaterialsl bom_outer
    WHERE BillOfMaterialsRevisionID = 10004
```
    
### EXISTS


```sql
SELECT mfg.Name
FROM product.Manufacturer mfg
WHERE  NOT EXISTS ( SELECT * FROM pur.PurchaseOrder po WHERE po.ManufacturerID = mfg.ManufacturerID)
```

### Views


```sql
CREATE VIEW PUR.ViewManufacturersWithPurchases
AS
    SELECT mfg.Name
	FROM product.Manufacturer mfg
	WHERE  NOT EXISTS ( SELECT * FROM pur.PurchaseOrder po WHERE po.ManufacturerID = mfg.ManufacturerID)
GO

SELECT * FROM PUR.ViewManufacturersWithPurchases
```

### Apply Operator


```sql
-- Query 5 most recent orders for each active product
SELECT a_left.PartNumberID, p_right.DateSold, p_right.LineTotal
FROM product.PartNumber a_left
CROSS APPLY -- replace with OUTER APPLY to include products from left that do not have any batches
(    
        SELECT TOP(5) PartNumberID, DateSold, LineTotal
        FROM sales.SkuSales AS p_inner
        WHERE p_inner.PartNumberID = a_left.PartNumberID
        ORDER BY DateSold DESC
    ) AS p_right
WHERE a_left.StatusID = 1
ORDER BY a_left.PartNumberID, p_right.DateSold DESC
```

### SET Operators



```sql
<query 1> -- column names define output columns
[Set Operation] -- specifying ALL will return duplicates
<query 2>

UNION [ALL] - return rows from both queries - using all will return rows from both queries
                even if they are duplicates

INTERSECT - return rows if they appear in both queries

EXCEPT - return rows if they appear in the 1st query but not the 2nd query
```

### PIVOT


    Pivot = rows-> columns
    
    Pivot Process

    1. Group
    2. Spread
    3. Aggregate
	
```sql
    SELECT ...
    FROM <source_table_or_table_expression>
      PIVOT(<agg_func>(<aggregation_element>)
            FOR <spreading_element>
              IN (<list_of_target_columns>)) AS <result_table_alias>
```

```sql              
    WITH ProductSalesYearCTE
    AS
    (
        SELECT PartNumberID, [1],[2],[3],[4],[5],[6],[7],[8],[9],[10],[11],[12]
        FROM
        (
            SELECT PartnUmberID, LineTotal, MONTH(DateSold) AS SalesMonth
            FROM sales.SkuSales
            WHERE YEAR(DateSold) = 2016
            
        ) AS SalesInYear
        PIVOT(SUM(LineTotal) FOR SalesMonth IN ([1],[2],[3],[4],[5],[6],[7],[8],[9],[10],[11],[12]))  AS P
    )
    SELECT PartNumberID, COALESCE([1], 0) AS January, COALESCE([2], 0) AS February, COALESCE([3], 0) AS March, COALESCE([4], 0) AS April
                 , COALESCE([5], 0)AS May, COALESCE([6], 0) AS June, COALESCE([7], 0) AS July, COALESCE([8], 0) AS August
                 , COALESCE([9], 0) AS September, COALESCE([10], 0) AS October, COALESCE([11], 0) AS November, COALESCE([12], 0) AS December
     FROM ProductSalesYearCTE
    ORDER BY PartNumberID
```
        
### UNPIVOT



    unpivot - columns->rows
```sql    
    SELECT ...
    FROM <source_table_or_table_expression>
      UNPIVOT(<target_col_to_hold_source_col_values>
        FOR <target_col_to_hold_source_col_names> IN(<list_of_source_columns>)) AS
    <result_table_alias>
```

```sql    
	select PartNumberID, SalesMonth, LineTotal
	from sales.ViewProductSalesYear
		unpivot (LineTotal for SalesMonth in (January, February, March, April, May, June, July, August,September,October,November,December) ) as U		
```

        
### Select from VALUES

```sql
    SELECT * FROM
    (
        VALUES (1,2), (2,3), (3,4), (4,5)
    ) AS t (field_1, field2)
    
    SELECT *
    FROM ( VALUES
             (10003, '20090213', 4, 'B'),
             (10004, '20090214', 1, 'A'),
             (10005, '20090213', 1, 'C'),
             (10006, '20090215', 3, 'C') )
    AS O(orderid, orderdate, empid, custid);
```
    
### Grouping Sets


```sql
    SELECT PartNumberID, Store, SUM(LineTotal) AS TotalSold
    FROM sales.SkuSales
    WHERE YEAR(DateSold) = 2013
    group by
        GROUPING sets
        (
            (), -- total 2013 Sales
            (Store), -- total 2013 sales by store
            (store,PartNumberID) -- total 2013 sales by Store + Part Number
        );
        
    -- ROLLUP clause
    -- will return
        -- 1) all Sales
        -- 2) Each Store
        -- 3) Each Store + SKU
        -- 4) Does not return just the SKU because PartNumberID listed after Store in rollup clause
    SELECT PartNumberID, Store, SUM(LineTotal) AS TotalSold
    FROM sales.SkuSales
    WHERE YEAR(DateSold) = 2013
    GROUP BY ROLLUP( Store, PartNumberID); -- Left-right order important!
```
        


### Group by cube

```sql
    - total sales at each store, and for each sku
    -     PartNumberID    Store
        null    null    - total 2013 sales
        <value>    null    - total 2013 sales for sku (any store)
        null    <value>    - total 2013 sales at store
        <value> <value>    - total 2013 sales for sku at store
    
    SELECT PartNumberID, Store, SUM(LineTotal) AS TotalSold
    FROM sales.SkuSales
    WHERE YEAR(DateSold) = 2013
    GROUP BY CUBE(Store, PartNumberID)
    ORDER BY PartNumberID
```
    
### Insert from Select

```sql
        INSERT INTO tempdb.dbo.ModelListing ( ManufacturerCode, Name)
            SELECT  ManufacturerCode, Name
            FROM product.Model
```
            
### Insert from Sproc

```sql
        INSERT INTO tempdb.dbo.ModelListing
        EXEC product.ModelList
```
                
### Select into a new table

```sql
            -- note that ModelID is an identity column, and will be created as such
            SELECT ModelID, ManufacturerCode, Name
            INTO tempdb.dbo.Model
            FROM product.Model
```
            
### Bulk Insert


        test file:  
            1,Brian,2003
            2,Kit,2006
            3,Dean, 2007
            4,Ryan,2010
            

```sql
        USE tempdb;

        if OBJECT_ID('dbo.SSBulk_Insert_Test','U') is not null drop table dbo.SSBulk_Insert_Test;

        create table dbo.SSBulk_Insert_Test
        (
            id int,
            person varchar(25),
            year_started int
        )

        BULK INSERT dbo.SSBulk_Insert_Test FROM 'c:\temp\SSBulk_Insert_Test.txt' -- MUST be releative to the server
          WITH
            (
               DATAFILETYPE    = 'char',
               FIELDTERMINATOR = ',',
               ROWTERMINATOR   = '\n'
            );
```

### Last Identity


```sql
    SCOPE_IDENTITY() -- @@identity is legacy, do not use
    
    -- scope_identity() will be null if no inserts in the current session
    -- do not use the following as replacement for scope_identity()
    select IDENT_CURRENT('product.Model')
```
    
### DELETE


```sql
    SELECT TOP 100 *
    INTO tempdb.dbo.Model
    FROM product.Model

    -- Does not update identity seed -
    DELETE FROM tempdb.dbo.Model -- entire table
    DELETE FROM tempdb.dbo.MOdel WHERE ModelID = 10000
    
    -- resets identity seed
    TRUNCATE TABLE tempdb.dbo.Model
    
    -- delete based on join
    DELETE FROM modelTemp
    FROM tempdb.dbo.Model modelTemp
    JOIN product.Manufacturer mfg
        ON modelTemp.ManufacturerID = mfg.ManufacturerID
    WHERE mfg.MaufacturerID = 10000
```
    
### UPDATE based on join


```sql
    DECLARE @table1 TABLE (date_field DATE, value_field VARCHAR(25))

    INSERT INTO @table1 (date_field, value_field) VALUES
        ('4/1/2012', 'intial value 1'), ('4/2/2012', 'intial value 2'),
        ('4/3/2012', 'intial value 3'), ('4/4/2012', 'intial value 4')
        
    SELECT * FROM @table1

    DECLARE @table2 TABLE (date_field DATE, value_field VARCHAR(25))

    INSERT INTO @table2 (date_field, value_field) VALUES
        ('4/1/2012', 'modified value 1'),('4/2/2012', 'modified value 2'),
        ('4/3/2012', 'modified value 3'),('4/4/2012', 'modified value 4')
            
    UPDATE t1
      SET t1.value_field = t2.value_field
    FROM @table1 t1
      JOIN @table2 AS t2
        ON t1.date_field = t2.date_field
```
        
### MERGE


```sql
    MERGE INTO <destination table> as dest
    using <source table> as source
        on dest.key = source.key
    WHEN MATCHED THEN
        UPDATE SET
            dest.val1 = source.val1,
            dest.val2 = source.val2
    WHEN NOT MATCHED THEN
        INSERT (key, val1, val2)
        VALUES (source.key, source.val1, source.val2)
```
        
### OUTPUT clause


```sql
    DECLARE @temp_table TABLE (ManufacturerCode VARCHAR(50))

    INSERT INTO @temp_table (ManufacturerCode)
    SELECT ManufacturerCode
    FROM product.ManufacturerCode
    WHERE ManufacturerCode LIKE 'PG-%';


    -- same as above, but with optional OUTPUT clause
    INSERT INTO @temp_table (ManufacturerCode)
    OUTPUT inserted.ManufacturerCode -- each displayed field must be preceded by inserted.<field name>
    SELECT ManufacturerCode
    FROM product.ManufacturerCode
    WHERE ManufacturerCode LIKE 'PG-%';

    -- delete
    
    DELETE FROM @temp_table
    OUTPUT deleted.ManufacturerCode
    WHERE ManufacturerCode LIKE 'PG-%'
    
    -- use output clause to view changed values (there is no 'updated' value per se)
    update @temp_table
        set ManufacturerCode = SUBSTRING(ManufacturerCode, 1,4)
    output
        deleted.ManufacturerCode as prior_value,
        inserted.ManufacturerCode as new_value
```
        
### Transactions

```sql
    BEGIN TRAN -- if not specified, each statement runs as an implicit transaction

    -- Statement #1
    -- Statement #2
    ...
    -- Statement N

    COMMIT TRAN -- All statements since BEGIN TRAN committed to database

    ROLLBACK -- ALL statements since BEGIN TRAN are canceled, no data will be changed
```	
	
### Detailed Transaction Template

```sql 
-- http://stackoverflow.com/questions/2073737/nested-stored-procedures-containing-try-catch-rollback-pattern/2074139#2074139
SET XACT_ABORT, NOCOUNT ON

DECLARE @starttrancount INT

BEGIN TRY
         SELECT @starttrancount = @@TRANCOUNT
         
        IF @starttrancount = 0
                BEGIN TRANSACTION
                
                -- Do work, call nested sprocs


        IF @starttrancount = 0 
                COMMIT TRANSACTION
END TRY 
BEGIN CATCH
        IF XACT_STATE() <> 0 AND @starttrancount = 0 
                ROLLBACK TRANSACTION 
                
        DECLARE @error INT, @message VARCHAR(4000)
        
        SET @error = ERROR_NUMBER()
        SET @message = ERROR_MESSAGE()
        
        RAISERROR('<Sproc Name> : %d: %s', 16, 1, @error, @message)
END CATCH

```
    
### Locking



    - Exclusive Locks
        - generated by update operations
        - no other session can update or read an item that has an exclusive lock
    - Shared Locks
        - generated by selects
        
    - Isolation levels affect how sql interacts with shared locks:
        READ UNCOMMITTED - SELECT does not generate shared locks - dirty reads
        READ COMMITTED (default) - SELECT requires shared locks
        REPEATABLE READ - shared lock open for the entire transaction
        SERIALIZABLE - locks range of keys returned to prevent phantom records
        SNAPSHOT - reads previous row reivsion stored in tempdb
        READ COMMITTED SNAPSHOT - Gets the last committed version of the row that was available when the statement started
    - can be set at database level or transaction level:
        SET TRANSACTION ISOLATION LEVEL <name>
        
### Variables


```sql
DECLARE @i AS INT;
SET @i = 10;


DECLARE @i AS INT = 10; -- only works in SS2008 or higher
    
DECLARE @ManufacturerCode VARCHAR(25)
DECLARE @Name VARCHAR(100)

-- select into variables
SELECT @ManufacturerCode = ManufacturerCode,
        @Name = Name
FROM product.Model
WHERE ModelID = 10000

```

### Flow Control


```sql
-- single statements
IF <expression>
    -- will execute if expression is true
ELSE
    -- will execute if expression is false or unknown
    
-- Multiple statements
IF <expression>
    BEGIN
        
    END
ELSE
    BEGIN
        
    END
    
WHILE <expression
    begin
    
    end
    
-- FOR replacement
DECLARE @i AS INT;

SET @i = 1
WHILE @i <= 10
BEGIN
  PRINT @i;
  SET @i = @i + 1;
END;
```

### Cursors


```sql
    DECLARE @ModelID  INT,
            @ManufacturerCode VARCHAR(25)
        
    DECLARE C CURSOR FAST_FORWARD /* read only, forward only */ FOR
    SELECT ModelID, ManufacturerCode
    FROM Product.Model
    ORDER BY ModelID

    OPEN C

    FETCH NEXT FROM C INTO @ModelID, @ManufacturerCode;

    WHILE @@FETCH_STATUS = 0
    BEGIN
    -- do work
    FETCH NEXT FROM C INTO @ModelID, @ManufacturerCode;
    END

    CLOSE C;

    DEALLOCATE C;
```

### Min Key approach    


```sql
    DECLARE @ModelID  INT
    DECLARE @ManufacturerCode VARCHAR(25)

    SET @ModelID = (SELECT MIN(ModelID) FROM product.model)

    WHILE @ModelID IS NOT NULL
    BEGIN
        SELECT @ManufacturerCode = ManufacturerCode
        FROM product.model
        WHERE ModelID = @ModelID
        
        -- do work
        
        SET @ModelID = (SELECT MIN(ModelID) FROM product.model WHERE ModelID > @ModelID)
    END
```

### Temporary Tables


```sql
CREATE TABLE #TempTable
(
    ModelID INT,
    ManufacturerCode VARCHAR(25)
)

CREATE TABLE ##GlobalTempTable
(
    ModelID INT,
    ManufacturerCode VARCHAR(25)
)
```

### Table Variables


```sql
DECLARE @TempTable TABLE
(
    ModelID INT,
    ManufacturerCode VARCHAR(25)
);
```

### Table Types


```sql
IF TYPE_ID('dbo.OrderTotalsByYear') IS NOT NULL
  DROP TYPE dbo.OrderTotalsByYear;
CREATE TYPE dbo.OrderTotalsByYear AS TABLE
(
  orderyear INT NOT NULL PRIMARY KEY,
  qty       INT NOT NULL
);

DECLARE @MyOrderTotalsByYear AS dbo.OrderTotalsByYear;
```

### Dynamic SQL


```sql
    -- EXEC
    DECLARE @sql AS VARCHAR(100);
    SET @sql = 'PRINT ''This message was printed by a dynamic SQL batch.'';';
    EXEC(@sql);
    
    -- sp_execute - supports params
    DECLARE @sql NVARCHAR(MAX);

    SET @sql = N'select * from product.model
    where Name LIKE ''%'' +  @Name + ''%'' AND CategoryId = @CategoryId'

    EXEC sp_executesql
        @stmt = @sql,
        @params = N'@Name as varchar(100), @CategoryId as int',
        @Name = 'John Deere',
        @CategoryId = 2
```

### User Defined Functions



#### Scalar

```sql
        CREATE FUNCTION dbo.fn_age
        (
          @birthdate AS DATETIME,
          @eventdate AS DATETIME
        )
        RETURNS INT -- <-- Defines as scalar
        AS
        BEGIN
          RETURN
            DATEDIFF(year, @birthdate, @eventdate)
             - CASE WHEN 100 * MONTH(@eventdate) + DAY(@eventdate)
                       < 100 * MONTH(@birthdate) + DAY(@birthdate)
                   THEN 1 ELSE 0
              END
        END
        GO
```

#### Table Value

```sql
        ALTER FUNCTION [dbo].[Split_MultiValue_Parameter]
        (     @delimitedString    VARCHAR(MAX)
            ,@delimiter            VARCHAR(1)
        )
        RETURNS @Table TABLE (VALUE VARCHAR(100)) -- <-- Defines as table

        AS
        BEGIN
           DECLARE @tempString VARCHAR(MAX);
           SET @tempString = ISNULL(@delimitedString,'') + @Delimiter;
           WHILE LEN(@tempString) > 0
           BEGIN
              INSERT INTO @Table
              SELECT SUBSTRING(@tempString,1,
                     CHARINDEX(@Delimiter,@tempString)-1);
              
              SET @tempString = RIGHT(@tempString,
                LEN(@tempString)-CHARINDEX(@Delimiter,@tempString)) ;
           END
           RETURN;
        END
```
        
### Stored Procedures


```sql
    CREATE PROCEDURE product.ModelList
    AS
    BEGIN
    SET NOCOUNT ON;

    SELECT ModelId, Name, ManufacturerCode, CategoryId, Description
    FROM product.Model

    END
```

### Triggers



#### DML - Data Modification

```sql
        ALTER TRIGGER  product.trProductPartNumberDateModified ON product.PartNumber FOR UPDATE AS
        DECLARE @PartNumberID INT;
        SELECT @PartNumberID = PartNumberId FROM Inserted;

        UPDATE Product.PartNumber
        SET DateModified = GETDATE()
        WHERE PartNumberID = @PartNumberID;
``` 
 
#### Structural modification

```sql
            CREATE TRIGGER [DDL_Notify]
            ON DATABASE
            FOR DROP_TABLE, ALTER_TABLE,CREATE_TABLE, DROP_FUNCTION, ALTER_FUNCTION, CREATE_FUNCTION,
                DROP_PROCEDURE, ALTER_PROCEDURE, CREATE_PROCEDURE
                
            AS
            
            -- actions
```
            
### Try / Catch

```sql
BEGIN TRY

    TRUNCATE TABLE tempdbo.dbo.DoesNotExist
    -- following statement will not execute
    select top 1 * from product.model

END TRY

BEGIN CATCH
    -- 4701 Cannot find the object "DoesNotExist" because it does not exist or you do not have permissions.
    SELECT ERROR_NUMBER(), ERROR_MESSAGE()

END CATCH
```
    
### Template for basic table + relationships


```sql
CREATE TABLE tempdb.dbo.temp1
(
    Temp1_pk INT IDENTITY(1,1) NOT NULL
    CONSTRAINT PK_dbo_Temp1 PRIMARY KEY CLUSTERED,
    VALUE VARCHAR(25)
)
GO

CREATE TABLE tempdb.dbo.temp2
(
    Temp2_pk INT IDENTITY(1,1) NOT NULL
        CONSTRAINT pk_dbo_temp2 PRIMARY KEY CLUSTERED,
    Temp1_fk INT NOT NULL
        CONSTRAINT FK_dbo_Temp2_dbo_Temp1 FOREIGN KEY (Temp1_fk)
            REFERENCES dbo.temp1(Temp1_pk),
    VALUE VARCHAR(25) NULL
)
GO
```

### Identity Insert


```sql
SET IDENTITY_INSERT product.Model ON

    INSERT INTO product.Model (ModelID, Name)
    VALUES (12345, 'Test')

SET IDENTITY_INSERT product.Model OFF
```
