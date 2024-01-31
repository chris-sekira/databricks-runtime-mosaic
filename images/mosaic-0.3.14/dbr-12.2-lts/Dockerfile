# syntax=docker/dockerfile:1.4

FROM databricksruntime/standard:12.2-LTS

ARG WITH_MOSAIC=1
ARG WITH_GDAL=1

ARG MOSAIC_VERSION=0.3.14
ARG MOSAIC_GDAL_VERSION=3.4.3
ARG MOSAIC_GDAL_JAR_VERSION=3.4.0

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

  apt-get update
  apt-get install -y unixodbc libcurl3-gnutls libsnappy-dev libopenjp2-7

  TMP_GDAL_ARTIFACTS_DIR="/tmp/gdal_artifacts"
  MOSAIC_REPO_RAW_URL="https://raw.githubusercontent.com/databrickslabs/mosaic/v_${MOSAIC_VERSION}"
  MOSAIC_REPO_GDAL_ARTIFACTS_PATH="modules/python/gdal_package/databricks-mosaic-gdal/resources"

  GDAL_FILETREE_ARTIFACT_NAME="gdal-${MOSAIC_GDAL_VERSION}-filetree.tar.xz"
  GDAL_SYMLINKS_ARTIFACT_NAME="gdal-${MOSAIC_GDAL_VERSION}-symlinks.tar.xz"

  GDAL_FILETREE_ARTIFACT_URL="${MOSAIC_REPO_RAW_URL}/${MOSAIC_REPO_GDAL_ARTIFACTS_PATH}/${GDAL_FILETREE_ARTIFACT_NAME}"
  GDAL_SYMLINKS_ARTIFACT_URL="${MOSAIC_REPO_RAW_URL}/${MOSAIC_REPO_GDAL_ARTIFACTS_PATH}/${GDAL_SYMLINKS_ARTIFACT_NAME}"

  GDAL_FILETREE_ARTIFACT_TMP_PATH="${TMP_GDAL_ARTIFACTS_DIR}/${GDAL_FILETREE_ARTIFACT_NAME}"
  GDAL_SYMLINKS_ARTIFACT_TMP_PATH="${TMP_GDAL_ARTIFACTS_DIR}/${GDAL_SYMLINKS_ARTIFACT_NAME}"

  mkdir -p ${TMP_GDAL_ARTIFACTS_DIR}

  curl -sSL "${GDAL_FILETREE_ARTIFACT_URL}" -o "${GDAL_FILETREE_ARTIFACT_TMP_PATH}"
  curl -sSL "${GDAL_SYMLINKS_ARTIFACT_URL}" -o "${GDAL_SYMLINKS_ARTIFACT_TMP_PATH}"

  tar -xf $GDAL_FILETREE_ARTIFACT_TMP_PATH -C /
  tar -xhf $GDAL_SYMLINKS_ARTIFACT_TMP_PATH -C /

  rm -rf $TMP_GDAL_ARTIFACTS_DIR

  # Install the GDAL .jar file to the common jars directory
  curl -sSL "https://repo1.maven.org/maven2/org/gdal/gdal/${MOSAIC_GDAL_JAR_VERSION}/gdal-${MOSAIC_GDAL_JAR_VERSION}.jar" -o "${DBR_JARS_DIR}/gdal-${MOSAIC_GDAL_JAR_VERSION}.jar"

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