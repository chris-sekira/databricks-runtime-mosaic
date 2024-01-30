# databricksruntime-mosaic:0.3.14-dbr-12.2-lts Docker Image

## Image Details

- **Base Image**: Databricks Runtime 12.2 LTS
- **Additional Libraries**: 
    - databricks-mosaic 0.3.14
    - databricks-mosaic-gdal 3.4.3


## Configuration

You can set the following configuration options on the Spark Cluster before startup

```conf
spark.databricks.labs.mosaic.jar.autoattach true
spark.databricks.labs.mosaic.raster.api GDAL
spark.databricks.labs.mosaic.geometry.api JTS
spark.databricks.labs.mosaic.index.system H3
spark.sql.extensions com.databricks.labs.mosaic.sql.extensions.MosaicSQL
```