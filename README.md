# r2
SELECT 
    a.date,
    a.total_ad,
    COALESCE(v.total_vis, 0) AS total_vis
FROM 
    (SELECT 
        COUNT(ad_id) AS total_ad, 
        DATE(created_at) AS date
     FROM 
        process 
     WHERE 
        flow_id = 1
     GROUP BY 
        DATE(created_at)
    ) a
LEFT JOIN 
    (SELECT 
        COUNT(p.ad_id) AS total_vis, 
        DATE(h.ttime) AS date
     FROM 
        process p
     INNER JOIN 
        task t ON p.process_id = t.process_id
     INNER JOIN 
        (SELECT 
            task_id, 
            MIN(action_time) AS ttime
         FROM 
            history
         WHERE 
            `action` = 'COMPLETED'
         GROUP BY 
            task_id
        ) h ON t.task_id = h.task_id
     WHERE 
        t.stage_id = 1
        AND t.redirect_reason IS NOT NULL
        AND t.redirect_reason LIKE ('')
     GROUP BY 
        DATE(h.ttime)
    ) v ON a.date = v.date

UNION ALL

SELECT 
    v.date,
    COALESCE(a.total_ad, 0) AS total_ad,
    v.total_vis
FROM 
    (SELECT 
        COUNT(p.ad_id) AS total_vis, 
        DATE(h.ttime) AS date
     FROM 
        process p
     INNER JOIN 
        task t ON p.process_id = t.process_id
     INNER JOIN 
        (SELECT 
            task_id, 
            MIN(action_time) AS ttime
         FROM 
            history
         WHERE 
            `action` = 'COMPLETED'
         GROUP BY 
            task_id
        ) h ON t.task_id = h.task_id
     WHERE 
        t.stage_id = 1
        AND t.redirect_reason IS NOT NULL
        AND t.redirect_reason LIKE ('')
     GROUP BY 
        DATE(h.ttime)
    ) v
LEFT JOIN 
    (SELECT 
        COUNT(ad_id) AS total_ad, 
        DATE(created_at) AS date
     FROM 
        process 
     WHERE 
        flow_id = 1
     GROUP BY 
        DATE(created_at)
    ) a ON a.date = v.date
WHERE 
    a.date IS NULL
ORDER BY 
    date;
