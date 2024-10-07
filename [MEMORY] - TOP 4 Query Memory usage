SELECT TOP 4
   qs.memory_grant_kb,
   qs.total_worker_time,
   qs.total_elapsed_time,
   qs.execution_count,
   SUBSTRING(qt.text, (qs.statement_start_offset/2) + 1,
   ((CASE qs.statement_end_offset
       WHEN -1 THEN DATALENGTH(qt.text)
       ELSE qs.statement_end_offset
   END - qs.statement_start_offset)/2) + 1) AS query_text
FROM
   sys.dm_exec_query_stats AS qs
   CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) AS qt
ORDER BY
   qs.memory_grant_kb DESC;



SELECT TOP 4
  qs.granted_query_memory * 8 AS memory_grant_kb, -- granted_query_memory is in pages, each page is 8 KB
  qs.total_worker_time,
  qs.total_elapsed_time,
  qs.execution_count,
  SUBSTRING(qt.text, (qs.statement_start_offset/2) + 1,
  ((CASE qs.statement_end_offset
      WHEN -1 THEN DATALENGTH(qt.text)
      ELSE qs.statement_end_offset
  END - qs.statement_start_offset)/2) + 1) AS query_text
FROM
  sys.dm_exec_query_stats AS qs
  CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) AS qt
ORDER BY
  qs.granted_query_memory DESC;
