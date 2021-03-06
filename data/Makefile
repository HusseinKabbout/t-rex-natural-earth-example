DBNAME = naturalearth4

createdb:
	psql postgres -c "DROP DATABASE IF EXISTS $(DBNAME)"
	psql postgres -c "CREATE DATABASE $(DBNAME)"
	psql $(DBNAME) -c "CREATE EXTENSION postgis"

# GPKG dump

gpkgdump:
	ogr2ogr -f GPKG naturalearth4.gpkg PG:dbname=$(DBNAME)

gpkgdump_mvtbench:
	ogr2ogr -f GPKG mvtbench.gpkg PG:dbname=$(DBNAME) ne_10m_admin_0_countries ne_10m_admin_0_country_points ne_10m_admin_0_boundary_lines_land ne_10m_admin_1_states_provinces_lines ne_10m_geographic_lines ne_50m_geographic_lines

gpkgrestore:
	ogr2ogr -f PostgreSQL PG:dbname=$(DBNAME) naturalearth4.gpkg

# Raw data import

data: packages/natural_earth_vector.sqlite vector-tiles-sample-master/countries.tm2source/files/b56c4047-10m-admin-0-country-points.shp

natural_earth_vector.sqlite.zip:
	wget http://naciscdn.org/naturalearth/packages/natural_earth_vector.sqlite.zip

packages/natural_earth_vector.sqlite: natural_earth_vector.sqlite.zip
	unzip $< $@
	touch $@

vector-tiles-sample.zip:
	wget -O vector-tiles-sample.zip https://github.com/klokantech/vector-tiles-sample/archive/master.zip

vector-tiles-sample-master/countries.tm2source/files/b56c4047-10m-admin-0-country-points.shp: vector-tiles-sample.zip
	unzip $< $@
	touch $@

LAYERS = ne_110m_admin_0_boundary_lines_land \
	     ne_110m_admin_0_countries \
	     ne_110m_admin_1_states_provinces_lines \
	     ne_110m_coastline \
	     ne_110m_geography_marine_polys \
	     ne_110m_geography_regions_polys \
	     ne_110m_rivers_lake_centerlines \
	     ne_110m_lakes \
	     ne_110m_glaciated_areas \
	     ne_110m_land \
	     ne_110m_ocean \
	     ne_50m_admin_0_boundary_lines_land \
	     ne_50m_admin_0_boundary_lines_disputed_areas \
	     ne_50m_admin_0_boundary_lines_maritime_indicator \
	     ne_50m_admin_0_countries \
	     ne_50m_admin_0_map_subunits \
	     ne_50m_admin_1_states_provinces_lakes \
	     ne_50m_admin_1_states_provinces_lines \
	     ne_50m_geographic_lines \
	     ne_50m_coastline \
	     ne_50m_antarctic_ice_shelves_lines \
	     ne_50m_antarctic_ice_shelves_polys \
	     ne_50m_geography_marine_polys \
	     ne_50m_geography_regions_polys \
	     ne_50m_rivers_lake_centerlines_scale_rank \
	     ne_50m_rivers_lake_centerlines \
	     ne_50m_lakes \
	     ne_50m_glaciated_areas \
	     ne_50m_land \
	     ne_50m_ocean \
	     ne_10m_admin_0_boundary_lines_land \
	     ne_10m_admin_0_boundary_lines_disputed_areas \
	     ne_10m_parks_and_protected_lands_area \
	     ne_10m_parks_and_protected_lands_line \
	     ne_10m_admin_0_boundary_lines_map_units \
	     ne_10m_admin_0_boundary_lines_maritime_indicator \
	     ne_10m_admin_0_countries \
	     ne_10m_admin_0_map_subunits \
	     ne_10m_admin_1_states_provinces_lines \
	     ne_10m_roads \
	     ne_10m_urban_areas \
	     ne_10m_geographic_lines \
	     ne_10m_coastline \
	     ne_10m_antarctic_ice_shelves_lines \
	     ne_10m_antarctic_ice_shelves_polys \
	     ne_10m_geography_marine_polys \
	     ne_10m_geography_regions_polys \
	     ne_10m_rivers_north_america \
	     ne_10m_rivers_europe \
	     ne_10m_rivers_lake_centerlines_scale_rank \
	     ne_10m_rivers_lake_centerlines \
	     ne_10m_playas \
	     ne_10m_reefs \
	     ne_10m_lakes_historic \
	     ne_10m_lakes_north_america \
	     ne_10m_lakes_europe \
	     ne_10m_lakes \
	     ne_10m_glaciated_areas \
	     ne_10m_land \
	     ne_10m_minor_islands \
	     ne_10m_ocean

POINT_LAYERS = ne_10m_admin_0_label_points \
			ne_10m_admin_1_label_points \
			ne_50m_geography_regions_elevation_points \
			ne_10m_geography_regions_elevation_points \
			ne_10m_geography_regions_points \
			ne_10m_parks_and_protected_lands_point \
			ne_110m_populated_places \
			ne_50m_populated_places \
			ne_10m_populated_places

# Difference to  tegola-osm/natural_earth.sh:
# ne_10m_parks_and_protected_lands.shp -> ne_10m_parks_and_protected_lands_area

import: packages/natural_earth_vector.sqlite
	OGR_ENABLE_PARTIAL_REPROJECTION=true ogr2ogr -f PostgreSQL PG:dbname=$(DBNAME) -t_srs EPSG:3857 -nlt PROMOTE_TO_MULTI $< $(LAYERS)
	OGR_ENABLE_PARTIAL_REPROJECTION=true ogr2ogr -f PostgreSQL PG:dbname=$(DBNAME) -t_srs EPSG:3857 $< $(POINT_LAYERS)

import_points: vector-tiles-sample-master/countries.tm2source/files/b56c4047-10m-admin-0-country-points.shp
	ogr2ogr -f PostgreSQL PG:dbname=$(DBNAME) -t_srs EPSG:3857 -nln ne_10m_admin_0_country_points $<

optimize:
	psql $(DBNAME) -c "CREATE TABLE ne_10m_land_subdivided AS SELECT ST_Subdivide(wkb_geometry)::geometry(Polygon,3857) AS wkb_geometry, featurecla, min_zoom FROM ne_10m_land"
	psql $(DBNAME) --quiet -c "CREATE INDEX ON ne_10m_land_subdivided USING gist(wkb_geometry)"
	psql $(DBNAME) -c "CREATE TABLE ne_50m_ocean_subdivided AS SELECT ST_Subdivide(wkb_geometry)::geometry(Polygon,3857) AS wkb_geometry, featurecla, min_zoom FROM ne_50m_ocean"
	psql $(DBNAME) --quiet -c "CREATE INDEX ON ne_50m_ocean_subdivided USING gist(wkb_geometry)"
	psql $(DBNAME) -c "CREATE TABLE ne_10m_ocean_subdivided AS SELECT ST_Subdivide(wkb_geometry)::geometry(Polygon,3857) AS wkb_geometry, featurecla, min_zoom FROM ne_10m_ocean"
	psql $(DBNAME) --quiet -c "CREATE INDEX ON ne_10m_ocean_subdivided USING gist(wkb_geometry)"
	psql $(DBNAME)  -c "VACUUM ANALYZE"
