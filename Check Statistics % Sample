--VERSION 1
SELECT      OBJECT_SCHEMA_NAME(obj.object_id) SchemaName, obj.name TableName, 
            stat.name, modification_counter, 
            [rows], rows_sampled, rows_sampled* 100 / [rows] AS [% Rows Sampled],
            last_updated
FROM        sys.objects AS obj
INNER JOIN  sys.stats AS stat ON stat.object_id = obj.object_id
CROSS APPLY sys.dm_db_stats_properties(stat.object_id, stat.stats_id) AS sp
WHERE       obj.is_ms_shipped = 0
ORDER BY    modification_counter DESC


--VERSION 2
SELECT stats.name AS StatisticsName,
       OBJECT_SCHEMA_NAME(stats.object_id) AS SchemaName,
       OBJECT_NAME(stats.object_id) AS TableName,
       last_updated AS LastUpdated,
       [rows] AS [Rows],
       rows_sampled,
       steps,
       modification_counter AS ModCounter, --persisted_sample_percent PersistedSamplePercent,
(rows_sampled * 100)/ROWS AS SamplePercent
FROM sys.stats
INNER JOIN sys.stats_columns sc ON stats.stats_id = sc.stats_id
AND stats.object_id = sc.object_id
INNER JOIN sys.all_columns ac ON ac.column_id = sc.column_id
AND ac.object_id = sc.object_id CROSS APPLY sys.dm_db_stats_properties(stats.object_id, stats.stats_id) shr
WHERE OBJECT_SCHEMA_NAME(stats.object_id) <> 'sys'
