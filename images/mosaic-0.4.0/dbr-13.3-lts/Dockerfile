# syntax=docker/dockerfile:1.4

FROM databricksruntime/standard:13.3-LTS

ARG WITH_MOSAIC=1
ARG WITH_GDAL=1
ARG WITH_UBUNTUGIS=0

ARG MOSAIC_VERSION=0.4.0

ARG DBR_JARS_DIR=/databricks/jars
ARG DBR_SPARK_CONF_DIR=/databricks/spark/conf

ARG DBR_PIP_EXEC=/databricks/python3/bin/pip
ARG DBR_PYTHON_EXEC=/databricks/python3/bin/python


# Common setup things
RUN <<EOF
#!/usr/bin/env bash

# Upgrade pip in the python venv and install the latest databricks-sdk
${DBR_PIP_EXEC} install --no-cache-dir --upgrade pip
${DBR_PIP_EXEC} install --no-cache-dir --upgrade databricks-sdk

# Create Databricks spark directories
mkdir -p ${DBR_JARS_DIR} ${DBR_SPARK_CONF_DIR}

EOF

# Bash script to install the gdal binaries and python bindings for mosaic
RUN <<EOF
#!/usr/bin/env bash

set -e

# databricks-mosaic version info
MOSAIC_GITHUB_REPO_RESOURCE_PATH="databrickslabs/mosaic/v_${MOSAIC_VERSION}/resources/gdal/jammy"

if [ ${WITH_GDAL} == 1 ]
then
  # - refresh package info
  apt-add-repository "deb http://archive.ubuntu.com/ubuntu $(lsb_release -sc)-backports main universe multiverse restricted"
  apt-add-repository "deb http://archive.ubuntu.com/ubuntu $(lsb_release -sc)-updates main universe multiverse restricted"
  apt-add-repository "deb http://archive.ubuntu.com/ubuntu $(lsb_release -sc)-security main multiverse restricted universe"
  apt-add-repository "deb http://archive.ubuntu.com/ubuntu $(lsb_release -sc) main multiverse restricted universe"

  if [ ${WITH_UBUNTUGIS} == 1 ]
  then
    add-apt-repository ppa:ubuntugis/ppa
    GDAL_VERSION=3.4.3
  else
    GDAL_VERSION=3.4.1
  fi

  apt-get update

  # - install natives
  apt-get install -y unixodbc libcurl3-gnutls libsnappy-dev libopenjp2-7
  apt-get install -y gdal-bin libgdal-dev python3-numpy python3-gdal

  # - pip install gdal
  ${DBR_PIP_EXEC} install --no-cache-dir gdal==$GDAL_VERSION

  # copy libraries from github
  wget -nv -P /usr/lib -nc https://raw.githubusercontent.com/$MOSAIC_GITHUB_REPO_RESOURCE_PATH/libgdalalljni.so
  wget -nv -P /usr/lib -nc https://raw.githubusercontent.com/$MOSAIC_GITHUB_REPO_RESOURCE_PATH/libgdalalljni.so.30
  wget -nv -P /usr/lib -nc https://raw.githubusercontent.com/$MOSAIC_GITHUB_REPO_RESOURCE_PATH/libgdalalljni.so.30.0.3

  apt-get clean
  rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
fi
EOF

# Install the databricks-mosaic package
RUN <<EOF
#!/usr/bin/env bash

set -e

if [ ${WITH_MOSAIC} == 1 ]
then
  # Install build tools for dependencies (h3 and scikit need to build)
  apt-get update
  apt-get install -y build-essential

  # Install the databricks-mosaic package
  ${DBR_PIP_EXEC} install --no-cache-dir "databricks-mosaic==${MOSAIC_VERSION}"

  # Remove PySpark since it is injected at runtime
  ${DBR_PIP_EXEC} uninstall -y pyspark

  # Get the mosaic/lib directory with the required .jar files
  MOSAIC_LIB_DIR="$(${DBR_PIP_EXEC} show databricks-mosaic | grep 'Location' | cut -d' ' -f2)/mosaic/lib"

  # Copy all of the .jar files to the databricks/jars directory
  find $MOSAIC_LIB_DIR -type f -name "*.jar" -exec cp "{}" ${DBR_JARS_DIR} ';'

  # Add the mosaic configuration to the spark-defaults.conf file
  echo "spark.databricks.labs.mosaic.jar.autoattach true" >> ${DBR_SPARK_CONF_DIR}/spark-defaults.conf
  echo "spark.databricks.labs.mosaic.jar.path ${DBR_JARS_DIR}" >> ${DBR_SPARK_CONF_DIR}/spark-defaults.conf 
  echo "spark.databricks.labs.mosaic.raster.api GDAL" >> ${DBR_SPARK_CONF_DIR}/spark-defaults.conf
  echo "spark.databricks.labs.mosaic.geometry.api JTS" >> ${DBR_SPARK_CONF_DIR}/spark-defaults.conf
  echo "spark.databricks.labs.mosaic.index.system H3" >> ${DBR_SPARK_CONF_DIR}/spark-defaults.conf
  echo "spark.sql.extensions com.databricks.labs.mosaic.sql.extensions.MosaicSQL" >> ${DBR_SPARK_CONF_DIR}/spark-defaults.conf

  apt-get clean
  rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
fi
EOF
