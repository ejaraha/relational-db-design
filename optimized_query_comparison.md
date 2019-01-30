

## Queries Before Optimization



#### <u>Query 1:</u>

#### What is the distance from each FRS apple orchard in California to its nearest casino?

RUN TIME: 44 seconds

```sql

SELECT 
f1.primary_name, 
	(SELECT f2.primary_name --select name of the closest casino
	FROM facility as f2
	WHERE f2.primary_name IN
		(SELECT f.primary_name --from facilities that are casinos
		FROM facility as f
		LEFT JOIN naics as n
		USING (registry_id)
		WHERE n.code_description ilike '%casino%'
		AND f.state = 'CA')
	ORDER BY f2.geom <-> f1.geom LIMIT 1),
		(SELECT (ST_Transform(f2.geom, 26910) <-> ST_Transform(f1.geom, 26910))/1000 as distance_km --select the distance of the closest casino
		FROM facility as f2
		WHERE f2.primary_name IN
			(SELECT f.primary_name --from facilities that are casinos
			FROM facility as f
			LEFT JOIN naics as n
			USING (registry_id)
			WHERE n.code_description ilike '%casino%'
			AND f.state = 'CA')
		ORDER BY f2.geom <-> f1.geom LIMIT 1)
FROM facility as f1		---find distance from facilies
WHERE f1.primary_name IN	--that are apple orchards
	(SELECT f.primary_name
	FROM facility as f
	LEFT JOIN naics as n
	USING (registry_id)
	WHERE n.code_description = 'APPLE ORCHARDS.'
	AND f.state = 'CA');

```

#### <u>Query 2:</u>

#### Which economic sectors, and how many FRS facilities of each economic sector, exist within one mile of a specific NPL site?

RUN TIME: 20 seconds

```sql
SELECT economic_sector, COUNT(economic_sector)
FROM facility as f
LEFT JOIN naics as n --get economic sector
USING (registry_id)
WHERE f.state = 'CA' --only CA geoms from facility ...... getting geoms from npl_final_site ........ for the desired site and distance
AND ST_DWithin(ST_Transform(f.geom,26910), (SELECT ST_Transform(geom,26910) FROM npl_final_site WHERE site_id = '0901389'),1610)
GROUP BY economic_sector --to get count
ORDER BY 2 DESC;

```

#### <u>Query 3:</u>

#### What is the economic sector of the closest FRS facility to each site on the NPL?

RUN TIME:  hours

```sql

SELECT site_id,			
	(SELECT n.economic_sector 
	FROM facility as f
	LEFT JOIN naics as n
	USING(registry_id) -- getting economic sector
	WHERE n.economic_sector IS NOT NULL -- not all facilities in the faciliy table are in the naics table
	ORDER BY f.geom <-> npl.geom ---find closest facility to npl site..index!
	LIMIT 1 ) 
FROM npl_final_site as npl --determines direction of distance measure


```



## Queries After Optimization

#### <u>Query 1:</u>

#### What is the distance from each FRS apple orchard in California to its nearest casino?

ORIGINAL RUN TIME: 44 seconds

*OPTIMIZED RUN TIME: 22 seconds*

```sql
SELECT n1.primary_name as apple_orchard, 
	(SELECT n2.primary_name
	FROM naics_denorm as n2
	WHERE code_description ilike '%casino%'
	AND state = 'CA'
	ORDER BY n1.geom <-> n2.geom 
    LIMIT 1) as casino,
		(SELECT (ST_Transform(n1.geom, 26910) 
                 <-> 
                 ST_Transform(n2.geom, 26910))/1000 as distance_km
		FROM naics_denorm as n2
		WHERE code_description ilike '%casino%'
		AND state = 'CA'
		ORDER BY n1.geom <-> n2.geom 
        LIMIT 1) as distance		
FROM naics_denorm as n1
WHERE code_description = 'APPLE ORCHARDS.'
AND state = 'CA';
```

#### <u>Query 2:</u>

#### Which economic sectors, and how many FRS facilities of each economic sector, exist within one mile of a specific NPL site?

ORIGINAL RUN TIME: 20 seconds

*OPTIMIZED RUN TIME: 6.5 seconds*

```sql
SELECT economic_sector, COUNT(economic_sector)
FROM naics_denorm
WHERE state = 'CA' 
AND ST_DWithin(
    	ST_Transform(geom,26910), 
    		(SELECT ST_Transform(geom,26910) 
         	FROM npl_final_site 
         	WHERE site_id = '0901389'),1610)
GROUP BY economic_sector 
ORDER BY 2 DESC;
```

#### <u>Query 3:</u>

#### What is the economic sector of the closest FRS facility to each site on the NPL?

ORIGINAL RUN TIME: >2 hours

*OPTIMIZED RUN TIME: 8 seconds*

```sql
SELECT site_id,			
	(SELECT n.economic_sector 
	FROM naics_denorm as n
	ORDER BY n.geom <-> npl.geom 
	LIMIT 1 ) as economic_sector_closest_facility
FROM npl_final_site as npl;
```
