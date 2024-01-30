# databricksruntime-mosaic:0.4.0-dbr-13.3-lts Docker Image

## Image Details

- **Base Image**: Databricks Runtime 13.3 LTS
- **Additional Libraries**: databricks-mosaic 0.4.0

## Configuration

You can set the following configuration options on the Spark Cluster before startup

```conf
spark.databricks.labs.mosaic.jar.autoattach true
spark.databricks.labs.mosaic.raster.api GDAL
spark.databricks.labs.mosaic.geometry.api JTS
spark.databricks.labs.mosaic.index.system H3
spark.sql.extensions com.databricks.labs.mosaic.sql.extensions.MosaicSQL
```