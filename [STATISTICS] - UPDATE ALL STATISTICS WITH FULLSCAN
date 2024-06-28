/* --COUNT STATISTICS
SELECT      count(*)
FROM        sys.objects AS obj
INNER JOIN  sys.stats AS stat ON stat.object_id = obj.object_id
CROSS APPLY sys.dm_db_stats_properties(stat.object_id, stat.stats_id) AS sp
WHERE       obj.is_ms_shipped = 0
and last_updated < getdate() -5
and obj.name not like '%%_TTAT_LOG%' -- Protheus Audit
AND OBJECT_SCHEMA_NAME(obj.object_id) not like '%TOTVSAUDIT%' COLLATE Latin1_General_CI_AI --RM Audit

*/




-- VERSÃO EDITADA


begin TRY
	drop table #tbl_stats;
	drop table #tbl_stats_priority;
end TRY
begin CATCH 
end CATCH

create table #tbl_stats
(
	DBName sysname,
	SchemaName sysname,
	TableName sysname,
	StatName sysname,
	LastUpdated datetime,
	ModificationCounter bigint,
	[% Rows Sampled] int
)

create table #tbl_stats_priority
(
	DBNamePriority sysname,
	SchemaNamePriority sysname,
	TableNamePriority sysname,
	StatNamePriority sysname,
	LastUpdatedPriority datetime,
	ModificationCounterPriority bigint,
	[% Rows Sampled Priority] int
)

 declare @mdcounter int  = 30
--SELECT valor variável para ser utilizada dentro do sp_MSforeachdb
if object_id('tempdb.dbo.##tbmdcounter') is not null drop table ##tbmdcounter
SELECT @mdcounter AS mdcounter INTO ##tbmdcounter

--Consulta estatísticas
insert into #tbl_stats (DBName, SchemaName, TableName, StatName, LastUpdated, ModificationCounter, [% Rows Sampled])
exec sp_MSforeachdb @command1 = 'use [?]
	IF DB_ID() > 4 and DATABASEPROPERTYEX(DB_NAME(), ''status'') = ''ONLINE'' and DATABASEPROPERTYEX(DB_NAME(), ''updateability'') = ''READ_WRITE''
		SELECT   
			DBName = DB_NAME(),   
			SchemaName = OBJECT_SCHEMA_NAME(obj.object_id),
			TableName = obj.name,
			StatName = stat.name,
			LastUpdated = sp.last_updated,
            ModificationCounter = sp.modification_counter,
			[% Rows Sampled] = sp.rows_sampled* 100 / [rows]
		FROM  sys.objects AS obj
		INNER JOIN  sys.stats AS stat ON stat.object_id = obj.object_id
		CROSS APPLY sys.dm_db_stats_properties(stat.object_id, stat.stats_id) AS sp 
		WHERE obj.is_ms_shipped = 0
			AND (sp.modification_counter* 100 / [rows] >= (select mdcounter from ##tbmdcounter) OR sp.rows_sampled* 100 / [rows] <= 70 OR DATEDIFF(DAY, sp.last_updated, GETDATE()) > 1)
			AND obj.name not in (SELECT TableNamePriority COLLATE Latin1_General_CI_AI from _DBACloudControle.[dbo].[UpdateStatistics_Priority]) 
			AND OBJECT_SCHEMA_NAME(obj.object_id) not like ''%TOTVSAUDIT%'' COLLATE Latin1_General_CI_AI
			AND obj.name not like ''%HISTORICO''
			AND obj.name not like ''%LOG''
			order by [rows] asc

'

----Consulta estatísticas Prioridade
insert into #tbl_stats_priority (DBNamePriority, SchemaNamePriority, TableNamePriority, StatNamePriority, LastUpdatedPriority, ModificationCounterPriority, [% Rows Sampled Priority])
exec sp_MSforeachdb @command1 = 'use [?]
	IF DB_ID() > 4 and DATABASEPROPERTYEX(DB_NAME(), ''status'') = ''ONLINE'' and DATABASEPROPERTYEX(DB_NAME(), ''updateability'') = ''READ_WRITE''
		SELECT   
			DBName = DB_NAME(),   
			SchemaName = OBJECT_SCHEMA_NAME(obj.object_id),
			TableName = obj.name,
			StatName = stat.name,
			LastUpdated = sp.last_updated,
            ModificationCounter = sp.modification_counter,
			[% Rows Sampled] = sp.rows_sampled* 100 / [rows]
		FROM  sys.objects AS obj
		INNER JOIN  sys.stats AS stat ON stat.object_id = obj.object_id
		CROSS APPLY sys.dm_db_stats_properties(stat.object_id, stat.stats_id) AS sp 
		WHERE obj.is_ms_shipped = 0
			--AND (sp.rows_sampled* 100 / [rows] <= 70 OR DATEDIFF(DAY, sp.last_updated, GETDATE()) > 2)
			AND (sp.modification_counter* 100 / [rows] >= (select mdcounter from ##tbmdcounter) OR sp.rows_sampled* 100 / [rows] <= 70 OR DATEDIFF(DAY, sp.last_updated, GETDATE()) > 1)
			AND obj.name not like ''%_TTAT_LOG%'' COLLATE Latin1_General_CI_AI
			AND obj.name in (SELECT TableNamePriority COLLATE Latin1_General_CI_AI from _DBACloudControle.[dbo].[UpdateStatistics_Priority])
			AND OBJECT_SCHEMA_NAME(obj.object_id) not like ''%TOTVSAUDIT%'' COLLATE Latin1_General_CI_AI
			AND obj.name not like ''%HISTORICO''
			AND obj.name not like ''%LOG''
			order by [rows] asc

'

declare @DBNamePriority sysname,
		@SchemaNamePriority sysname,
		@TableNamePriority sysname,
		@StatNamePriority sysname,
		@str_UpdateStatsPriority nvarchar(max) = ''

declare cr_UpdateStatsPriority cursor keyset for
select DBNamePriority, SchemaNamePriority, TableNamePriority, StatNamePriority
from #tbl_stats_priority
order by DBNamePriority


open cr_UpdateStatsPriority

fetch first from cr_UpdateStatsPriority into @DBNamePriority, @SchemaNamePriority, @TableNamePriority, @StatNamePriority
while @@FETCH_STATUS = 0

 begin --priority

	set @str_UpdateStatsPriority = 'UPDATE STATISTICS ' + QUOTENAME(@DBNamePriority) + '.' + QUOTENAME(@SchemaNamePriority) + '.' + QUOTENAME(@TableNamePriority) + '(' + '['+@StatNamePriority+']' + ') WITH FULLSCAN,MAXDOP=8'
	exec (@str_UpdateStatsPriority)

	fetch next from cr_UpdateStatsPriority into @DBNamePriority, @SchemaNamePriority, @TableNamePriority, @StatNamePriority
 
 end --while priority


declare @DBName sysname,
		@SchemaName sysname,
		@TableName sysname,
		@StatName sysname,
		@str_UpdateStats nvarchar(max) = ''
		

declare cr_UpdateStats cursor keyset for
select DBName, SchemaName, TableName, StatName
from #tbl_stats
order by DBName


open cr_UpdateStats

fetch first from cr_UpdateStats into @DBName, @SchemaName, @TableName, @StatName
while @@FETCH_STATUS = 0

 begin --normal

	set @str_UpdateStats = 'UPDATE STATISTICS ' + QUOTENAME(@DBName) + '.' + QUOTENAME(@SchemaName) + '.' + QUOTENAME(@TableName) + '(' + '['+@StatName+']' + ') WITH FULLSCAN,MAXDOP=8'
	exec (@str_UpdateStats)

	fetch next from cr_UpdateStats into @DBName, @SchemaName, @TableName, @StatName
 
 end --while normal

  IF (SELECT CURSOR_STATUS('global','cr_UpdateStats')) >= -1
	BEGIN
	CLOSE cr_UpdateStats
	DEALLOCATE cr_UpdateStats
  END

  IF (SELECT CURSOR_STATUS('global','cr_UpdateStatsPriority')) >= -1
	BEGIN
	CLOSE cr_UpdateStatsPriority
	DEALLOCATE cr_UpdateStatsPriority
  END
PRINT @mdcounter



/*
/*
drop table #tbl_stats
drop table #tbl_stats_priority

*/
create table #tbl_stats
(
	DBName sysname,
	SchemaName sysname,
	TableName sysname,
	StatName sysname,
	LastUpdated datetime,
	ModificationCounter bigint,
	[% Rows Sampled] int
)

create table #tbl_stats_priority
(
	DBNamePriority sysname,
	SchemaNamePriority sysname,
	TableNamePriority sysname,
	StatNamePriority sysname,
	LastUpdatedPriority datetime,
	ModificationCounterPriority bigint,
	[% Rows Sampled Priority] int
)

 declare @mdcounter int  = 30
--SELECT valor variável para ser utilizada dentro do sp_MSforeachdb
if object_id('tempdb.dbo.##tbmdcounter') is not null drop table ##tbmdcounter
SELECT @mdcounter AS mdcounter INTO ##tbmdcounter

--Consulta estatísticas
insert into #tbl_stats (DBName, SchemaName, TableName, StatName, LastUpdated, ModificationCounter, [% Rows Sampled])
exec sp_MSforeachdb @command1 = 'use [?]
	IF DB_ID() > 4 and DATABASEPROPERTYEX(DB_NAME(), ''status'') = ''ONLINE'' and DATABASEPROPERTYEX(DB_NAME(), ''updateability'') = ''READ_WRITE''
		SELECT   
			DBName = DB_NAME(),   
			SchemaName = OBJECT_SCHEMA_NAME(obj.object_id),
			TableName = obj.name,
			StatName = stat.name,
			LastUpdated = sp.last_updated,
            ModificationCounter = sp.modification_counter,
			[% Rows Sampled] = sp.rows_sampled* 100 / [rows]
		FROM  sys.objects AS obj
		INNER JOIN  sys.stats AS stat ON stat.object_id = obj.object_id
		CROSS APPLY sys.dm_db_stats_properties(stat.object_id, stat.stats_id) AS sp 
		WHERE obj.is_ms_shipped = 0
			AND (sp.modification_counter* 100 / [rows] >= (select mdcounter from ##tbmdcounter) OR sp.rows_sampled* 100 / [rows] <= 70 OR DATEDIFF(DAY, sp.last_updated, GETDATE()) > 1)
			AND obj.name not in (SELECT TableNamePriority COLLATE Latin1_General_CI_AI from _DBACloudControle.[dbo].[UpdateStatistics_Priority]) 
			AND OBJECT_SCHEMA_NAME(obj.object_id) not like ''%TOTVSAUDIT%'' COLLATE Latin1_General_CI_AI
			AND obj.name not like ''%HISTORICO''
			AND obj.name not like ''%LOG''
			order by [rows] asc

'

----Consulta estatísticas Prioridade
insert into #tbl_stats_priority (DBNamePriority, SchemaNamePriority, TableNamePriority, StatNamePriority, LastUpdatedPriority, ModificationCounterPriority, [% Rows Sampled Priority])
exec sp_MSforeachdb @command1 = 'use [?]
	IF DB_ID() > 4 and DATABASEPROPERTYEX(DB_NAME(), ''status'') = ''ONLINE'' and DATABASEPROPERTYEX(DB_NAME(), ''updateability'') = ''READ_WRITE''
		SELECT   
			DBName = DB_NAME(),   
			SchemaName = OBJECT_SCHEMA_NAME(obj.object_id),
			TableName = obj.name,
			StatName = stat.name,
			LastUpdated = sp.last_updated,
            ModificationCounter = sp.modification_counter,
			[% Rows Sampled] = sp.rows_sampled* 100 / [rows]
		FROM  sys.objects AS obj
		INNER JOIN  sys.stats AS stat ON stat.object_id = obj.object_id
		CROSS APPLY sys.dm_db_stats_properties(stat.object_id, stat.stats_id) AS sp 
		WHERE obj.is_ms_shipped = 0
			--AND (sp.rows_sampled* 100 / [rows] <= 70 OR DATEDIFF(DAY, sp.last_updated, GETDATE()) > 2)
			AND (sp.modification_counter* 100 / [rows] >= (select mdcounter from ##tbmdcounter) OR sp.rows_sampled* 100 / [rows] <= 70 OR DATEDIFF(DAY, sp.last_updated, GETDATE()) > 1)
			AND obj.name not like ''%_TTAT_LOG%'' COLLATE Latin1_General_CI_AI
			AND obj.name in (SELECT TableNamePriority COLLATE Latin1_General_CI_AI from _DBACloudControle.[dbo].[UpdateStatistics_Priority])
			AND OBJECT_SCHEMA_NAME(obj.object_id) not like ''%TOTVSAUDIT%'' COLLATE Latin1_General_CI_AI
			AND obj.name not like ''%HISTORICO''
			AND obj.name not like ''%LOG''
			order by [rows] asc

'



SELECT 'UPDATE STATISTICS '+TableName+'('+StatName+')'+ ' WITH FULLSCAN,MAXDOP=8' from #tbl_stats

SELECT distinct (TableName),'UPDATE STATISTICS '+TableName+' WITH FULLSCAN,MAXDOP=8' from #tbl_stats


*/
