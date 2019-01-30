
### Queries

```sql
--------------------------------------
--find economic sectors and number of facilities in each economic sector
--within 400 meters
--of the npl site with site_id = '0901389'
--------------------------------------

SELECT economic_sector, COUNT(economic_sector)
FROM facility as f
LEFT JOIN naics as n 
USING (registry_id)
WHERE f.state = 'CA' 
AND ST_DWithin(ST_Transform(f.geom,26910), (SELECT ST_Transform(geom,26910) FROM npl_final_site WHERE site_id = '0901389'),400)
GROUP BY economic_sector 
ORDER BY 2 DESC;


```

```sql
--------------------------------------
--facilities in california 
--with code_description = 'APPLE ORCHARDS.'
--------------------------------------

SELECT *
FROM facility as f
LEFT JOIN naics as n
USING (registry_id)
WHERE n.code_description = 'APPLE ORCHARDS.'
AND f.state = 'CA';
```

```sql
--------------------------------------
--facilities in Boulder, CO
--with primary_name ilike '%bakery%'
--------------------------------------

SELECT *
FROM facility as f
INNER JOIN environmental_interest as ei
USING(registry_id)
WHERE f.primary_name ilike '%bakery%'
AND f.city = 'BOULDER';
```
