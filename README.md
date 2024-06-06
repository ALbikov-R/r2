SELECT total_count, total_vis, date_col FROM (
  SELECT 
    COUNT(p.ad_id) AS total_count, 
    NULL AS total_vis, 
    DATE(p.created_at) AS date_col
  FROM process p
  WHERE flow_id = 1
  GROUP BY DATE(p.created_at)
  
  UNION
  
  SELECT 
    NULL AS total_count, 
    COUNT(p.ad_id) AS total_vis, 
    DATE(h.ttime) AS date_col
  FROM process p
  INNER JOIN task t ON p.process_id = t.process_id
  INNER JOIN (
    SELECT task_id, MIN(action_time) AS ttime
    FROM history
    WHERE `action` = 'COMPLETED'
    GROUP BY task_id
  ) h ON t.task_id = h.task_id
  WHERE 
    t.stage_id = 1
    AND t.redirect_reason IS NOT NULL
    AND t.redirect_reason LIKE ''
  GROUP BY DATE(h.ttime)
) combined
GROUP BY date_col
ORDER BY date_col;
