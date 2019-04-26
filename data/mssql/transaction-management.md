# Data Architecture | Microsoft SQL Server | Transaction Management

## Transactions

A transaction is used to ensure that all the statements within are run and executed successfully before fully commiting the changes to the database. In case a problem arises during runtime, all changes done since the transaction started are rolled back.

* to enclose SQL statements inside a transaction clause

```sql
BEGIN TRY
	BEGIN TRANSACTION

	/* do work */
	
	COMMIT TRANSACTION
END TRY
BEGIN CATCH
	ROLLBACK TRANSACTION
	PRINT(ERROR_MESSAGE()) 
END CATCH
```

## Object Existence

Checking when an object exists is essential to allow iterative runs of any SQL statements without the risk of data duplication or unintended actions.

### Table

* to check if `TBL_SAMPLE_A` table under `MY` schema exists:

```sql
IF OBJECT_ID('MY.TBL_SAMPLE_A', 'U') IS NOT NULL BEGIN

    /* do work */
	
END
```

## Using Cursors

A cursor is used to loop through the rows of a table.

* to iterate through the `Id` and `Name` columns of `TBL_SAMPLE_A` table of `MY` schema

```sql
DECLARE @curr AS CURSOR
DECLARE @id AS INT
DECLARE @name AS VARCHAR(50)

BEGIN
	SET @curr = CURSOR FOR
	SELECT [Id], [Name]
	FROM [MY].[TBL_SAMPLE_A]
	
	OPEN @curr
	FETCH NEXT FROM @curr
	INTO @id, @name
	
	WHILE @@FETCH_STATUS = 0 BEGIN
		
		/* do work */
		
		FETCH NEXT FROM @curr
		INTO @id, @name
	END
	
	CLOSE @curr
	DEALLOCATE @curr
END
```
