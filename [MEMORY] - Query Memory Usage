SELECT session_id, requested_memory_kb / 1024 as RequestedMemMb, 
granted_memory_kb / 1024 as GrantedMemMb, text
FROM sys.dm_exec_query_memory_grants qmg
CROSS APPLY sys.dm_exec_sql_text(sql_handle)


SELECT 
  session_id
  ,requested_memory_kb
  ,granted_memory_kb
  ,used_memory_kb
  ,queue_id
  ,wait_order
  ,wait_time_ms
  ,is_next_candidate
  ,pool_id
  ,text
  ,query_plan
FROM sys.dm_exec_query_memory_grants
  CROSS APPLY sys.dm_exec_sql_text(sql_handle)
  CROSS APPLY sys.dm_exec_query_plan(plan_handle)
