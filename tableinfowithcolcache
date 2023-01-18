--Table infornmation with columnar cache
SELECT n.nspname as "Schema",
  c.relname as "Name",
  CASE c.relkind WHEN 'r' THEN 'table' WHEN 'v' THEN 'view' WHEN 'm' THEN 'materialized view' WHEN 'i' THEN 'index' WHEN 'S' THEN 'sequence' WHEN 't' THEN 'TOAST table' WHEN 'f' THEN 'foreign table' WHEN 'p' THEN 'partitioned table' WHEN 'I' THEN 'partitioned index' END as "Type",
  pg_catalog.pg_get_userbyid(c.relowner) as "Owner",
  CASE c.relpersistence WHEN 'p' THEN 'permanent' WHEN 't' THEN 'temporary' WHEN 'u' THEN 'unlogged' END as "Persistence",
  pg_catalog.pg_size_pretty(pg_catalog.pg_table_size(c.oid)) as "Size",
  gcol.status as "ColumnarCache",
  round(gcol.size/1024/1024,2) as "ColCacheSizeMB",
  pg_catalog.obj_description(c.oid, 'pg_class') as "Description"
FROM pg_catalog.pg_class c
     LEFT JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
     LEFT JOIN pg_catalog.pg_am am ON am.oid = c.relam 
     LEFT JOIN g_columnar_relations gcol on gcol.schema_name = n.nspname and gcol.relation_name = c.relname
WHERE c.relkind IN ('r','p','')
      AND n.nspname <> 'pg_catalog'
      AND n.nspname !~ '^pg_toast'
      AND n.nspname <> 'information_schema'
  AND pg_catalog.pg_table_is_visible(c.oid)
ORDER BY 1,2;

--Table column information.
SELECT a.attname,
  pg_catalog.format_type(a.atttypid, a.atttypmod),
  (SELECT pg_catalog.pg_get_expr(d.adbin, d.adrelid, true)
   FROM pg_catalog.pg_attrdef d
   WHERE d.adrelid = a.attrelid AND d.adnum = a.attnum AND a.atthasdef),
  a.attnotnull,
  gcols.status as "ColumnarCache",
 round(gcols.size_in_bytes/1024/1024,2) as "ColCacheSizeMB"
FROM pg_catalog.pg_class c
     LEFT JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
     left join pg_catalog.pg_attribute a on a.attrelid = c.oid
     left outer join g_columnar_columns gcols on gcols.schema_name = n.nspname and gcols.relation_name = c.relname
     and gcols.column_name = a.attname
WHERE c.relname OPERATOR(pg_catalog.~) '^(lineitem)$' COLLATE pg_catalog.default
  AND pg_catalog.pg_table_is_visible(c.oid) AND a.attnum > 0 AND NOT a.attisdropped
ORDER BY a.attnum;
