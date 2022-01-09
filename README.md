# Vertica-Commands
# Vertica commands
 General Commands for Vertica

## Get user_session/query_count
    SELECT node_name, total_user_session_count, executed_query_count FROM query_metrics;

## Delete Vectors (A high number of delete vectors adversely affects the performance of your database.)
    SELECT COUNT(*) FROM v_monitor.delete_vectors;

## ROS Containers (If the number of ROS containers in your database grows too large, it can negatively affect performance)
    SELECT node_name, projection_schema, projection_name, SUM(ros_count) AS ros_count FROM v_monitor.projection_storage GROUP BY node_name, projection_schema, projection_name ORDER BY ros_count DESC;

## Monitor the Resource Pools
    SELECT sysdate AS current_time, node_name, pool_name, memory_inuse_kb, general_memory_borrowed_kb, running_query_count FROM resource_pool_status WHERE pool_name IN ('general') ORDER BY 1,2,3;

## Monitor if a query is taking excessive memory resource and causing the cluster to slow down
    SELECT * FROM resource_acquisitions ORDER BY memory_inuse_kb desc limit 10;

## Resource Pool Queue Status
    SELECT * FROM v_monitor.resource_queues;

## Resource Request Rejections 
    SELECT * FROM v_monitor.resource_rejections;

## system_resource_usage 
    SELECT * FROM v_monitor.system_resource_usage ORDER BY end_time DESC;

## Error Messages
    SELECT distinct error_level, message from error_messages where message ilike 'Vertica suggests%';

## Get pools
    SELECT * from resource_pools 

##  Monitor the Storage Space Vertica
    SELECT * FROM v_monitor.storage_usage ORDER BY poll_timestamp DESC;

##  GEt sessions Active
    SELECT user_name, session_id, current_statement, statement_start FROM v_monitor.sessions;

##  Get Active Query
    SELECT node_name, query, query_start, user_name, is_executing FROM v_monitor.query_profiles WHERE is_executing = 't';

##  Monitor the Load Status == Check the loading progress of active and historical queries using the following query:
    SELECT table_name, read_bytes, input_file_size_bytes, accepted_row_count, rejected_row_count, parse_complete_percent, sort_complete_percent FROM load_streams WHERE is_executing = 't' ORDER BY table_name;

##  Get Lock Status
    SELECT locks.lock_mode, locks.lock_scope, substr(locks.transaction_description, 1, 100) AS "left", locks.request_timestamp, locks.grant_timestamp FROM v_monitor.locks;

## Get Recovery Status
    SELECT node_name, recover_epoch, recovery_phase, current_completed, current_total, is_running FROM v_monitor.recovery_status ORDER BY 1;

## Monitor the Rebalance Status : A clean node dependency lists (number of nodes + 1) lines
    SELECT GET_NODE_DEPENDENCIES();

## Monitor the Overall Progress of the Rebalance Operation
    SELECT rebalance_method Rebalance_method, Status, COUNT(*) AS Count FROM ( SELECT rebalance_method, CASE WHEN (separated_percent = 100 AND transferred_percent = 100) THEN 'Completed' WHEN ( separated_percent <>  0 and separated_percent <> 100) OR (transferred_percent <> 0 AND transferred_percent <> 100) THEN 'In Progress' ELSE 'Queued' END AS  Status FROM v_monitor.rebalance_projection_status WHERE is_latest) AS tab GROUP BY 1, 2 ORDER BY 1, 2;

##  Historical Activities
    SELECT user_name, start_timestamp, request_duration_ms, transaction_id, statement_id, substr(request, 0, 1000) as request FROM v_monitor.query_requests WHERE transaction_id > 0 ORDER BY request_duration_ms DESC limit 10;

## Monitor the Memory Usage
    SELECT node_name, transaction_id, statement_id, user_name, start_timestamp, request_duration_ms, memory_acquired_mb, substr(request, 1, 100) AS request FROM v_monitor.query_requests WHERE transaction_id = transaction_id AND statement_id = statement_id;

## Host Resources
    Select * from host_resources;

## View the partition count per node per projection: The default maximum for the ROS_COUNT field is 1024
    SELECT node_name, projection_name, count(partition_key) FROM v_monitor.partitions GROUP BY node_name, projection_name ORDER BY node_name, projection_name;

## Monitor the Segmentation and Data Skew: Segmentation refers to organizing and distributing data across cluster nodes for fast data purges and query performance
##  The projections should have equivalent number of rows per projection per node
    SELECT ps.node_name, ps.projection_schema, ps.projection_name, ps.row_count FROM v_monitor.projection_storage ps INNER JOIN v_catalog.projections p ON ps.projection_schema = p.projection_schema AND ps.projection_name = p.projection_name WHERE p.is_segmented ORDER BY ps.projection_schema, ps.projection_name, ps.node_name;


## Monitor the Load Streams: This is useful for obtaining statistics about how many records got loaded and rejected from the previous load. 
    SELECT schema_name, table_name, load_start, load_duration_ms, is_executing, parse_complete_percent, sort_complete_percent, accepted_row_count, rejected_row_count FROM v_monitor.load_streams;

##  Get DB Size and ROWS
    SELECT t.table_schema,t.table_name AS table_name,  SUM(ps.wos_row_count + ps.ros_row_count) AS row_count, SUM(ps.wos_used_bytes + ps.ros_used_bytes)/ ( 1024 ^3 ) AS byte_countGB FROM tables t JOIN projections p ON t.table_id = p.anchor_table_id JOIN projection_storage ps on p.projection_name = ps.projection_name WHERE (ps.wos_used_bytes + ps.ros_used_bytes) > 500000 GROUP BY t.table_schema,t.table_name ORDER BY t.table_schema,t.table_name,byte_countGB;  

   
##  Get  resource pools
    SELECT name, memorysize, maxmemorysize,priority, maxconcurrency FROM V_CATALOG.RESOURCE_POOLS;


##  Get  Status of All Resource Pools
    SELECT pool_name, memory_size_kb, memory_size_actual_kb, memory_inuse_kb, general_memory_borrowed_kb,running_query_count  FROM V_MONITOR.RESOURCE_POOL_STATUS
   
##  Get Query Resource Acquisitions
    SELECT pool_name, thread_count, open_file_handle_count, memory_inuse_kb, queue_entry_timestamp, acquisition_timestamp ,(acquisition_timestamp-queue_entry_timestamp) AS 'queue wait'  FROM V_MONITOR.RESOURCE_ACQUISITIONS
   
   
##  GET Disk Space
    Select NODE_NAME, STORAGE_USAGE, RANK, THROUGHPUT, LATENCY, DISK_BLOCK_SIZE_BYTES, DISK_SPACE_USED_BLOCKS, DISK_SPACE_USED_MB, DISK_SPACE_FREE_BLOCKS, DISK_SPACE_FREE_MB, DISK_SPACE_FREE_PERCENT from DISK_STORAGE


## GET Host Resource
    Select HOST_NAME, OPEN_FILES_LIMIT, THREADS_LIMIT, CORE_FILE_LIMIT_MAX_SIZE_BYTES, PROCESSOR_COUNT, OPENED_FILE_COUNT, OPENED_SOCKET_COUNT, OPENED_NONFILE_NONSOCKET_COUNT, TOTAL_MEMORY_BYTES, TOTAL_MEMORY_FREE_BYTES,  TOTAL_BUFFER_MEMORY_BYTES, TOTAL_MEMORY_CACHE_BYTES, TOTAL_SWAP_MEMORY_BYTES, TOTAL_SWAP_MEMORY_FREE_BYTES, DISK_SPACE_FREE_MB, DISK_SPACE_USED_MB, DISK_SPACE_TOTAL_MB from HOST_RESOURCES


## GET IO Usage
    Select NODE_NAME, READ_KBYTES_PER_SEC, WRITTEN_KBYTES_PER_SEC from IO_USAGE

## GET Node Status
    SELECT node_name, node_state FROM nodes ORDER BY 1;

## GET Server STatus 
    Select s1.NODE_NAME, s1.NODE_STATE from NODE_STATES s1 WHERE event_timestamp =( SELECT MAX(event_timestamp) FROM NODE_STATES s2 WHERE s1.node_name = s2.node_name)

## Close Session
    SELECT close_user_sessions('u1');
    SELECT CLOSE_ALL_SESSIONS();

## GET  active partitions for a projection
    SELECT DISTINCT partition_key FROM strata  WHERE projection_name ILIKE '%sktest%'    AND schema_name ILIKEe '%public%'

## Querying Existing Storage Policies
    select node_name, projection_name, storage_type, location_label from v_monitor.storage_containers;

## Get a list of all node names on your cluster
    SELECT node_name from DISK_STORAGE;

## Determine the size of the database catalog in memory
    SELECT node_name, MAX (ts) AS ts, MAX(catalog_size_in_MB) AS catlog_size_in_MB FROM (SELECT node_name, TRUNC((dc_allocation_pool_statistics_by_second."time")::TIMESTAMP, 'SS'::VARCHAR(2)) AS ts,  SUM((dc_allocation_pool_statistics_by_second.total_memory_max_value - dc_allocation_pool_statistics_by_second.free_memory_min_value))/(1024*1024)  AS catalog_size_in_MB from dc_allocation_pool_statistics_by_second GROUP BY 1, TRUNC((dc_allocation_pool_statistics_by_second."time")::TIMESTAMP, 'SS'::VARCHAR(2)) ) subquery_1 GROUP BY 1 ORDER BY 1 LIMIT 50;
	
## Returns the top 50 projections that have the maximum ROS containers
    SELECT s.node_name, p.table_schema, s.projection_name, COUNT(DISTINCT s.storage_oid) storage_container_count, COUNT(DISTINCT partition_key) partition_count,  COUNT(r.rosid) ros_file_count FROM storage_containers s LEFT OUTER JOIN PARTITIONS p  ON s.storage_oid = p.ros_id JOIN vs_ros r  ON r.delid = s.storage_oid GROUP BY 1,2,3 ORDER BY 4 DESC LIMIT 50;

## Get Epoch/and fault tolerance
    SELECT current_epoch, ahm_epoch, last_good_epoch, designed_fault_tolerance, current_fault_tolerance, wos_used_bytes, ros_used_bytes FROM system;
