-- UPDATE DE ESTATÍSTICAS NAS TABELAS DO SISTEMA
SELECT 'UPDATE STATISTICS ' + QUOTENAME(S.[name], '[') + '.' + QUOTENAME(O.[name], '[') + ' WITH FULLSCAN;' AS [SQL]
FROM sys.objects AS O JOIN sys.schemas AS S ON O.[schema_id] = S.[schema_id]
WHERE O.[type_desc] = 'SYSTEM_TABLE';
