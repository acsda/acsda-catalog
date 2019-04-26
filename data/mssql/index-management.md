# Data Architecture | Microsoft SQL Server | Index Management

## Reindexing

Reindexing is done to delete-insert indices in all tables with existing indices. This will clean up cluttered data and free up storage which will improve the overall performance of queries.

* to execute reindexing in the `DB_SAMPLE_A` database

```sql
DECLARE	@curr AS CURSOR
    ,	@SchemaName AS NVARCHAR(50)
	,	@TableName AS NVARCHAR(250)
	,	@IndexName AS NVARCHAR(250)
	,	@IndexAvgFragmentation AS FLOAT
	,	@stmt AS VARCHAR(1000)

DECLARE	@IndexedTables AS TABLE (
			SchemaName NVARCHAR(50)
		,	TableName NVARCHAR(250)
		,	IndexName NVARCHAR(250)
		,	IndexAvgFragmentation FLOAT
	)

INSERT INTO @IndexedTables
SELECT	d.name AS SchemaName
	,	c.name AS TableName
	,	b.name AS IndexName
	,	a.avg_fragmentation_in_percent AS IndexAvgFragmentation
FROM sys.dm_db_index_physical_stats(DB_ID('DB_SAMPLE_A'), NULL, NULL, NULL, NULL) AS a  
    JOIN sys.indexes AS b ON a.object_id = b.object_id 
		AND a.index_id = b.index_id
	JOIN sys.objects AS c ON b.object_id = c.object_id
	JOIN sys.schemas AS d ON c.schema_id = d.schema_id
WHERE	b.name IS NOT NULL
	AND	b.name NOT LIKE '%Missing Index%'
	AND	a.avg_fragmentation_in_percent > 5
ORDER BY	d.name
		,	c.name
		,	b.name

BEGIN
	SET @curr = CURSOR FOR SELECT * FROM @IndexedTables
	OPEN @curr

	FETCH NEXT FROM @curr
	INTO @SchemaName, @TableName, @IndexName, @IndexAvgFragmentation

	WHILE @@FETCH_STATUS = 0 BEGIN
		
		--   ------------------ ----------------------------------------
		--  | fragmentation    | corrective statement                   |
		--  |------------------|----------------------------------------|
		--  | > 5% and <= 30%  | ALTER INDEX REORGANIZE                 |
		--  | > 30%            | ALTER INDEX REBUILD WITH (ONLINE = ON) |
		--   ------------------ ----------------------------------------
		-- 	ref: https://docs.microsoft.com/en-us/sql/relational-databases/indexes/reorganize-and-rebuild-indexes?view=sql-server-2014

		SET @stmt = 'ALTER INDEX [' + @IndexName + '] ON [' + @SchemaName + '].[' + @TableName + '] '
		IF @IndexAvgFragmentation <= 30 BEGIN
			SET @stmt = @stmt + 'REORGANIZE;'
		END
		ELSE BEGIN
			SET @stmt = @stmt + 'REBUILD WITH (FILLFACTOR = 80, SORT_IN_TEMPDB = ON, STATISTICS_NORECOMPUTE = ON, ONLINE = ON);'
		END

		PRINT @stmt
		PRINT ' -- Average Fragmentation: ' + CAST(@IndexAvgFragmentation AS NVARCHAR(250))
		EXEC(@stmt)

		FETCH NEXT FROM @curr
		INTO @SchemaName, @TableName, @IndexName, @IndexAvgFragmentation
	END

	CLOSE @curr
	DEALLOCATE @curr
END
```
