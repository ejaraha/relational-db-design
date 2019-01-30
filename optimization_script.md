
### The optimization script ....

1. #### *denormalizes* the naics table  

   - **how:** 

     1. create a copy of the naics table called naics_denorm
     2. append and update fields from the facility table:
        - primary_name
        - state
        - geom

   - **why:**

     the facility table holds the geom field, which stores facility locations. the naics table stores fields that are necessary to query the data. Both  tables are necessary to build interesting spatial queries. Without denormalization, a join would be necessary for most spatial queries. 

2. #### *creates an index* on the geometry column in **naics_denorm** (the denormalized naics table)

   - **how:**

     1. create a spatial index on the geom column in naics_denorm

   - **why:** 

     a spatial index on geom allows significantly faster access to data. this is especially helpful when using KNN operators (as shown in Query 3).



## Optimization Script

```sql
-----------------------------------------------------
--CREATE TABLE (NAICS_DENORM)
--FROM (NAICS and FACILITY) 
-----------------------------------------------------

--create table: naics_denorm
CREATE TABLE naics_denorm AS
(SELECT * FROM naics);

--add primary key
ALTER TABLE naics_denorm
ADD PRIMARY KEY (registry_id, naics_code);

--add columns from facility
ALTER TABLE naics_denorm
ADD COLUMN primary_name varchar,
ADD COLUMN state varchar,
ADD COLUMN geom geometry(point,4269);

--update primary_name, state, and geom fields
UPDATE naics_denorm
SET primary_name = sub.primary_name, state = sub.state, geom = sub.geom
FROM 
	(SELECT primary_name, registry_id, state, geom 
	FROM facility) as sub
WHERE sub.registry_id = naics_denorm.registry_id;

-----------------------------------------------------
--CREATE INDEX
--ON NAICS_DENORM(geom)
-----------------------------------------------------

CREATE INDEX
ON naics_denorm USING gist(geom);
```





