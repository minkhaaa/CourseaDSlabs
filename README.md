# CourseraDSlabs
```
        WITH FilteredComponents AS (
            SELECT parent_id
            FROM "ENG_WW_GQ_RMA_DM"."RMAFast"."AT_TASKS"
            WHERE src:TaskLevel IN ('MSB_SITE_EFA')
        ),
        OrderedTasks AS (
            SELECT
                parent_id, 
                src:ApplicationUrl AS URL,
                src:atTaskType AS atTaskType, 
                src,
                LAST_MODIFICATION_DATE
            FROM 
                "ENG_WW_GQ_RMA_DM"."RMAFast"."AT_TASKS"
            WHERE parent_id IN (SELECT parent_id FROM FilteredComponents) AND 
            src:enable = 'True'            
        )
        SELECT 
            components.id as primary_key,
            components.tracking_number as component_ID,
            TO_TIMESTAMP(CAST(COMPONENTS.SRC:faRequestDate::variant['$date'] AS BIGINT) / 1000) AS fa_Request_Date,
            COMPONENTS.SRC:designId as DID,
            COMPONENTS.SRC:qasiLotID as QASI,
            COMPONENTS.SRC:failRegister as fail_char,
            COMPONENTS.SRC:asmFailBin as fail_bin,
            COMPONENTS.SRC:priority as Priority,
            COMPONENTS.src:lotLocation as Lot_Location,
            COMPONENTS.SRC:status as Status, 
            at_details.SRC:ticketstatus as Ticket_Status,
            at_details.SRC:RequestorName as Requestor,
            LISTAGG(OrderedTasks.atTaskType, '-> ') WITHIN GROUP (ORDER BY OrderedTasks.SRC:startDate::variant['$date']) AS Analysis_Chain,
            LISTAGG(OrderedTasks.src, '--end--') WITHIN GROUP (ORDER BY OrderedTasks.SRC:startDate::variant['$date']) AS task_srcs
        FROM 
            "ENG_WW_GQ_RMA_DM"."RMAFast"."AT_COMPONENTS" components
        JOIN 
            OrderedTasks ON OrderedTasks.parent_id = components.id
        JOIN 
            "ENG_WW_GQ_RMA_DM"."RMAFast"."AT_DETAILS" at_details ON components.parent_id = at_details.id
        WHERE components.SRC:status != 'Completed' AND
            at_details.SRC:ticketstatus NOT IN('Completed', 'None')
        GROUP BY 
            primary_key, 
            component_ID, 
            fa_Request_Date,
            DID,
            QASI,
            fail_char,
            fail_bin,
            Priority,
            Lot_Location,
            Status,
            Ticket_Status,
            Requestor
        ORDER BY 
            FA_REQUEST_DATE DESC;

```
