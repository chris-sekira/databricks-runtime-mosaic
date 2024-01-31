ARG DB_RUNTIME_STANDARD_TAG=12.2-LTS
FROM databricksruntime/standard:${DB_RUNTIME_STANDARD_TAG} AS prebuild

# RUN export DEBIAN_FRONTEND=noninteractive \
#     && apt-get update \
#     && apt-get upgrade -y \
#     && apt-get install -y gnupg ca-certificates curl \
#     && curl -s https://repos.azul.com/azul-repo.key | gpg --dearmor -o /usr/share/keyrings/azul.gpg \
#     && echo "deb [signed-by=/usr/share/keyrings/azul.gpg] https://repos.azul.com/zulu/deb stable main" | tee /etc/apt/sources.list.d/zulu.list \
#     && apt-get clean \
#     && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# RUN export DEBIAN_FRONTEND=noninteractive \
#     && apt-get remove -y --autoremove openjdk-8-jdk openjdk-8-jre \
#     && apt-get update \
#     && apt-get install -y zulu8-ca-jdk zulu8-ca-jre ca-certificates-java \
#     && /var/lib/dpkg/info/zulu8-ca-jre.postinst configure \
#     && /var/lib/dpkg/info/zulu8-ca-jdk.postinst configure \
#     && /var/lib/dpkg/info/ca-certificates-java.postinst configure \
#     && apt-get clean \
#     && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Install build dependencies
RUN export DEBIAN_FRONTEND=noninteractive \
    && apt-get update \
    && apt-get install -y \
        build-essential software-properties-common \
        python3-dev python3-numpy python3-setuptools \
        libpq-dev libjpeg-dev libpng-dev \
        libtiff-dev libgeos-dev libproj-dev \
        libxml2-dev libexpat-dev libxerces-c-dev \
        libnetcdf-dev libpoppler-dev libspatialite-dev \
        libhdf4-alt-dev libhdf5-serial-dev \
        libfreexl-dev unixodbc-dev \
        libwebp-dev libepsilon-dev liblcms2-2 \
        libpcre3-dev libcrypto++-dev libdap-dev libfyba-dev \
        libkml-dev libmysqlclient-dev libogdi-dev \
        libcfitsio-dev \
        libsnappy-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

FROM prebuild as build

ARG TMP_GDAL_BUILD_DIR=/tmp/gdal_install
ARG CUSTOM_GDAL_INSTALL_DIR=/opt/gdal_custom

# Download and build GDAL
ENV GDAL_VERSION 3.4.3
RUN export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which javac)))) \
    && mkdir -p ${TMP_GDAL_BUILD_DIR} ${CUSTOM_GDAL_INSTALL_DIR} \
    && curl -sSL https://download.osgeo.org/gdal/${GDAL_VERSION}/gdal-${GDAL_VERSION}.tar.gz -o ${TMP_GDAL_BUILD_DIR}/gdal-${GDAL_VERSION}.tar.gz \
    && tar -xzf ${TMP_GDAL_BUILD_DIR}/gdal-${GDAL_VERSION}.tar.gz -C ${TMP_GDAL_BUILD_DIR} \
    && cd ${TMP_GDAL_BUILD_DIR}/gdal-${GDAL_VERSION} \
    && ./configure --prefix=${CUSTOM_GDAL_INSTALL_DIR} --with-python --with-java=$JAVA_HOME \
    && make -j$(nproc) \
    && make install -j$(nproc)

# # Clean up
# RUN cd .. && \
#     rm -rf gdal-${GDAL_VERSION} && \
#     rm gdal-${GDAL_VERSION}.tar.gz