# vector_tiles_showcase


## Martin

### Option 1 : load with config file

Create the config file (scan postgresql to infer spatial tables)

```bash
docker run -p 3000:3000 -e DATABASE_URL=postgresql://maverick_test@docker.for.mac.localhost:5432/maverick_test -v $(pwd)/config:/config ghcr.io/maplibre/martin --save-config config.yaml
```

Go to the docker conatiner, and export the file.

```bash
docker run -p 3000:3000 -v $(pwd)/config:/config ghcr.io/maplibre/martin --config /config/config.yaml
```

### Option 2 : load without config file

```bash
docker run -p 3000:3000 -e DATABASE_URL=postgresql://maverick_test@docker.for.mac.localhost:5432/maverick_test -v $(pwd)/config:/config ghcr.io/maplibre/martin --config /config/config.yaml
```

Example : 

- PostgreSQL
    
    Query to merge to layers (communes and cantons)
    
    ```sql
    CREATE OR REPLACE
        FUNCTION admin(z integer, x integer, y integer)
        RETURNS bytea AS $$
    DECLARE
      mvt bytea;
    BEGIN
    	IF z < 10 THEN
    	  SELECT INTO mvt ST_AsMVT(tile, 'admin', 4096, 'geom') FROM (
    		SELECT
    		  ST_AsMVTGeom(
    			  ST_Transform(ST_CurveToLine(geom), 3857),
    			  ST_TileEnvelope(z, x, y),
    			  4096, 64, true) AS geom
    		FROM canton
    		WHERE geom && ST_Transform(ST_TileEnvelope(z, x, y), 4326)
    	  ) as tile WHERE geom IS NOT NULL;
    	  RETURN mvt;
    	  
    	 ELSIF z >= 10  THEN
    	 
    	 SELECT INTO mvt ST_AsMVT(tile, 'admin', 4096, 'geom') FROM (
    		SELECT
    		  ST_AsMVTGeom(
    			  ST_Transform(ST_CurveToLine(geom), 3857),
    			  ST_TileEnvelope(z, x, y),
    			  4096, 64, true) AS geom
    		FROM communes
    		WHERE geom && ST_Transform(ST_TileEnvelope(z, x, y), 4326)
    	  ) as tile WHERE geom IS NOT NULL;
    	  RETURN mvt;
    		
    	 END IF; 
    	 
    END
    $$ LANGUAGE plpgsql IMMUTABLE STRICT PARALLEL SAFE;
    ```
    
- Web app
    
    run the web server : 
    
    ```bash
    python3 -m http.server
    ```