select ag.name
, ags.primary_replica
, db_name(drs.database_id) as database_name
, rcs.replica_server_name
, drs.synchronization_health_desc
, drs.synchronization_state_desc
, drs.log_send_queue_size
, drs.log_send_rate
, drs.redo_queue_size
, drs.redo_rate
, drs.last_received_time
, drs.last_redone_time
from sys.availability_groups ag
inner join sys.dm_hadr_availability_group_states ags on ags.group_id = ag.group_id
inner join sys.dm_hadr_database_replica_states drs on drs.group_id = ag.group_id
inner join sys.dm_hadr_availability_replica_cluster_states rcs on rcs.replica_id = drs.replica_id
order by drs.redo_queue_size desc
