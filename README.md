# mysql_sys.memory_by_thread_by_current_bytes-

SELECT 
    `mt`.`THREAD_ID` AS `thread_id`,
    IF((`t`.`NAME` = 'thread/sql/one_connection'),
        CONCAT(`t`.`PROCESSLIST_USER`,
                '@',
                CONVERT( `t`.`PROCESSLIST_HOST` USING UTF8MB4)),
        REPLACE(`t`.`NAME`, 'thread/', '')) AS `user`,
    SUM(`mt`.`CURRENT_COUNT_USED`) AS `current_count_used`,
    FORMAT_BYTES(SUM(`mt`.`CURRENT_NUMBER_OF_BYTES_USED`)) AS `current_allocated`,
    FORMAT_BYTES(IFNULL((SUM(`mt`.`CURRENT_NUMBER_OF_BYTES_USED`) / NULLIF(SUM(`mt`.`CURRENT_COUNT_USED`), 0)),
                    0)) AS `current_avg_alloc`,
    FORMAT_BYTES(MAX(`mt`.`CURRENT_NUMBER_OF_BYTES_USED`)) AS `current_max_alloc`,
    FORMAT_BYTES(SUM(`mt`.`SUM_NUMBER_OF_BYTES_ALLOC`)) AS `total_allocated`
FROM
    (`performance_schema`.`memory_summary_by_thread_by_event_name` `mt`
    JOIN `performance_schema`.`threads` `t` ON ((`mt`.`THREAD_ID` = `t`.`THREAD_ID`)))
GROUP BY `mt`.`THREAD_ID` , IF((`t`.`NAME` = 'thread/sql/one_connection'),
    CONCAT(`t`.`PROCESSLIST_USER`,
            '@',
            CONVERT( `t`.`PROCESSLIST_HOST` USING UTF8MB4)),
    REPLACE(`t`.`NAME`, 'thread/', ''))
ORDER BY SUM(`mt`.`CURRENT_NUMBER_OF_BYTES_USED`) DESC
