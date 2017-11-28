# [Greater Vancouver Regional District Planning Department Land Use Maps](https://open.library.ubc.ca/collections/gvrdmaps)

Proof of concept of a georeferenced web application using data from the Greater Vancouver Regional District Planning collection from UBC Open Collections. This collection contains maps of the metro Vancouver region. There are two index maps that serve as a reference to localize the detailed sections:

- [Index Map : Subdivision and Land Use Maps](https://open.library.ubc.ca/collections/gvrdmaps/items/1.0135075)
- [Index - Land Use Series](https://open.library.ubc.ca/collections/gvrdmaps/items/1.0135076)

## Interactive Index Map

![gvrd_maps.gif](https://github.com/carolamigo/ubc_gvrd_maps/blob/master/gvrd_maps.gif)

### How to do it

- Install QGIS (http://www.qgis.org/en/site/).

- Georeference the Index Map image following the instructions on this link: http://www.qgistutorials.com/en/docs/georeferencing_basics.html.

- Create a vector layer with an attribute “map_id”, following the instructions here: http://docs.qgis.org/2.18/en/docs/training_manual/create_vector_data/create_new_vector.html#basic-fa-the-layer-creation-dialog

- Draw polygons digitizing the tiles information on the Index Map (red lines). On the vector layer, toggle the edit on on the top bar (pencil icon) and click on the “Add Feature” button. Use “control” to close the polygon. Use the attribute map_id to identify the polygons according to the tile number.

- Toggle the edit off on the pencil icon. Left-click on the vector layer name on the left sidebar and save as GeoJSON, CRS (EPSG:4346, WGS 84). Name the file gvrd_partial.

- Open the GeoJSON file gvrd_partial in Open Refine, parsing data as line based text files. 

- Rename Column 1 to “feature”. We want now to extract the map_id from each line. Add a new column named “map_id” based on this column, using the following expression:

`value.replace(/[^\d]/, " ")`

- Trim leading and trailing whitespaces of the “map_id” column by selecting “Edit cells” > “Common transforms” > “Trim leading and trailing whitespaces”.

- On the “map_id” column, select “Edit cells” > “Split multi-valued cells”. The separator is “ ” (empty space) and select maximum of two columns. Delete the second column from the split, keeping just the first one with the map ids. Rename it to map_id.

- Delete the number “1” value from the first cell, since it is not a map id. This spreadsheet is ready to use.

- Download collection metadata using the Open Collections Research API. A php script to batch download is provided at OC API Documentation page > Download Collection Data. This script returns a folder containing one RDF file per collection item (or XML, JSON, any format preferred). We are going to use N-triples because the file is cleaner (no headers or footers), what makes the merging easier later. Edit the script following the instructions on the documentation page and run it using the command:

`$ php collection_downloader.php --cid gvrdmaps --fmt ntriples`

- Merge the files using the Unix cat command:

`$ cat * > merged_filename`

- Convert merged file obtained to a tabular format. Import project in Open Refine using the RDF/N3 files option. No character encoding selection is needed.

- This dataset has more than one set of maps, so we need to filter just the ones that belong to the index map we are working with. Facet column “http://purl.org/dc/terms/publisher” by text. Filter only "Lower Mainland Regional Planning Board of B.C." values.

- To extract just the map numbers, create a new column named “map_id” based on the column “http://purl.org/dc/terms/identifier”, using the expression:

`value.replace(/[^\d]/, " ")`

- Check the results by faceting the column. Just unique values should show up, no numbers missing.

- To reconcile the values of the geojson spreadsheet with the current spreadsheet, create a new column named “geojson” based on the column “map_id” using the expression:

`forEach(cell.cross("gvrd_partial", "map_id"),r,forNonBlank(r.cells["feature"].value,v,v,"")).join("|")`

- Build IIIF image urls. Create a new column named “iiif_url” based on the column “http://www.europeana.eu/schemas/edm/isShownAt”, using the expression:

`"http://iiif.library.ubc.ca/image/cdm.gvrdmaps."+value.substring(10,19).replace(".","-")+".0000" + "/full/300,300/0/default.jpg"`

- Remove double quotes and @en from title. Create a new column named “title” based on the column “http://purl.org/dc/terms/title”, using the expression:

`value.replace("\"", " ").replace("@en"," ")`

- Add all values to geojson string. Split column geojson by using separator “,“, maximum 3 columns. Transform split column 2 cells using the following expression:

`value.replace(" }", "") + " , \"subject\": \"" + cells["subject"].value + "\" , \"title\": " + cells["title"].value + " , " + "\"purl\": \"" + cells["iiif_url"].value + "\" }"`

- Merge the three columns back together by creating a new column named geojson_code using the following expression:

`cells["geojson 1"].value + " , " + cells["geojson 2"].value + " , " + cells["geojson 3"].value`

- Check the results by faceting the column. Any error on the syntax will prevent the application of working.

- Export the geojson_code column back to the original geojson text file. Export > Custom tabular exporter. Select only the geojson_code column, do not output column headers or empty rows. On the download tab, select “Excel” and click download. Open the excel file, remove the comma from the end of the last line, copy the column, and paste it in the appropriate position in the original geojson text file, replacing the old code. Save it as gvrd_full.geojson. This file can be debugged with the help of a geojson debugger such as https://jsonlint.com/.

- Set up leaflet in your machine, following the instructions here: http://leafletjs.com/download.html (Building Leaflet from the Source).

- Use the code provided on file [index.html](https://github.com/carolamigo/ubc_gvrd_maps/blob/master/index.html) to run the application.

## References:

- https://maptimeboston.github.io/leaflet-intro/
- http://joshuafrazier.info/leaflet-basics/
- https://geekswithlatitude.readme.io/docs/leaflet-map-with-geojson-popups
- http://leafletjs.com/examples/quick-start/
- https://github.com/Leaflet/Leaflet/blob/master/debug/map/image-overlay.html
- http://leafletjs.com/reference-1.2.0.html#imageoverlay
- http://www.qgistutorials.com/en/#
- http://leafletjs.com/examples/geojson/

