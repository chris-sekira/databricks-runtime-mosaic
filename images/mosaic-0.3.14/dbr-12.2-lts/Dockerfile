# syntax=docker/dockerfile:1.4

FROM databricksruntime/standard:12.2-LTS

ARG WITH_MOSAIC=1
ARG WITH_GDAL=1

ARG MOSAIC_VERSION=0.3.14
ARG MOSAIC_GDAL_VERSION=3.4.3

ARG DBR_JARS_DIR=/databricks/jars

ARG DBR_PIP_EXEC=/databricks/python3/bin/pip
ARG DBR_PYTHON_EXEC=/databricks/python3/bin/python


# Common setup things
RUN <<EOF
#!/usr/bin/env bash

set -e

# Upgrade pip in the python venv and install the latest databricks-sdk
${DBR_PYTHON_EXEC} -m pip install --no-cache-dir --upgrade pip
${DBR_PIP_EXEC} install --no-cache-dir --upgrade databricks-sdk

# Create Databricks spark directories
mkdir -p ${DBR_JARS_DIR}

EOF

# Bash script to install the gdal binaries and python bindings for mosaic
RUN <<EOF
#!/usr/bin/env bash

set -e

if [ ${WITH_GDAL} == 1 ]
then

  # Install the custom version of GDAL libs via databricks-mosaic-gdal package
  ${DBR_PIP_EXEC} install --no-cache-dir databricks-mosaic-gdal==${MOSAIC_GDAL_VERSION}

  # Get the resources directory of the databricks-mosaic-gdal package
  GDAL_RESOURCE_DIR="$(${DBR_PIP_EXEC} show databricks-mosaic-gdal | grep 'Location' | cut -d' ' -f2)/databricks-mosaic-gdal/resources"

  # Extract the custom GDAL resources to the container
  tar -xf $GDAL_RESOURCE_DIR/gdal-${MOSAIC_GDAL_VERSION}-filetree.tar.xz --skip-old-files -C /
  tar -xhf $GDAL_RESOURCE_DIR/gdal-${MOSAIC_GDAL_VERSION}-symlinks.tar.xz --skip-old-files -C /
fi
EOF

# Install the databricks-mosaic package
RUN <<EOF
#!/usr/bin/env bash

set -e

if [ ${WITH_MOSAIC} == 1 ]
then

  # Install the databricks-mosaic package
  ${DBR_PIP_EXEC} install --no-cache-dir "databricks-mosaic==${MOSAIC_VERSION}"

  # Remove PySpark since it is injected at runtime
  ${DBR_PIP_EXEC} uninstall -y pyspark

  # Get the mosaic/lib directory with the required .jar files
  MOSAIC_LIB_DIR="$(${DBR_PIP_EXEC} show databricks-mosaic | grep 'Location' | cut -d' ' -f2)/mosaic/lib"

  # Copy all of the .jar files to the databricks/jars directory
  find $MOSAIC_LIB_DIR -type f -name "*.jar" -exec cp "{}" ${DBR_JARS_DIR} ';'

  apt-get clean
  rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
fi
EOF
