FROM docker.bintray.io/jfrog/openjdk:8u131

MAINTAINER matank@jfrog.com

USER root

# Set vars
ENV ARTIFACTORY_USER_NAME=artifactory \
    ARTIFACTORY_USER_ID=1030 \
    ARTIFACTORY_HOME=/opt/jfrog/artifactory \
    ARTIFACTORY_DATA=/var/opt/jfrog/artifactory \
    ARTIFACTORY_EXTRA_CONF=/artifactory_extra_conf \
    RECOMMENDED_MAX_OPEN_FILES=32000 \
    MIN_MAX_OPEN_FILES=10000 \
    RECOMMENDED_MAX_OPEN_PROCESSES=1024 \
    POSTGRESQL_VERSION=9.4.1212

# Copy the artifactory zip and entrypoint files
COPY standalone.zip /opt/jfrog/
COPY entrypoint-artifactory.sh /
COPY Dockerfile.artifactory /docker/artifactory-pro/

# Create user, data directory and extract artifactory zip and create needed softlinks
RUN useradd -M -s /usr/sbin/nologin --uid ${ARTIFACTORY_USER_ID} --user-group ${ARTIFACTORY_USER_NAME} && \
    mkdir -p ${ARTIFACTORY_DATA} && \
    chown -R ${ARTIFACTORY_USER_NAME}:${ARTIFACTORY_USER_NAME} ${ARTIFACTORY_DATA} && \
    unzip -q /opt/jfrog/standalone.zip -d /opt/jfrog/ && \
    mv ${ARTIFACTORY_HOME}*/ ${ARTIFACTORY_HOME}/ && \
    rm -f /opt/jfrog/standalone.zip && \
    mv ${ARTIFACTORY_HOME}/etc ${ARTIFACTORY_HOME}/etc.orig/ && \
    rm -rf ${ARTIFACTORY_HOME}/logs && \
    ln -s ${ARTIFACTORY_DATA}/etc ${ARTIFACTORY_HOME}/etc && \
    ln -s ${ARTIFACTORY_DATA}/data ${ARTIFACTORY_HOME}/data && \
    ln -s ${ARTIFACTORY_DATA}/logs ${ARTIFACTORY_HOME}/logs && \
    ln -s ${ARTIFACTORY_DATA}/backup ${ARTIFACTORY_HOME}/backup && \
    ln -s ${ARTIFACTORY_DATA}/access ${ARTIFACTORY_HOME}/access && \
    ln -s ${ARTIFACTORY_DATA}/replicator ${ARTIFACTORY_HOME}/replicator && \
    chown -R ${ARTIFACTORY_USER_NAME}:${ARTIFACTORY_USER_NAME} ${ARTIFACTORY_HOME} && \
    mkdir -p /tmp/plugins && \
    chmod +x ${ARTIFACTORY_HOME}/bin/*.sh ${ARTIFACTORY_HOME}/tomcat/bin/*.sh /entrypoint-artifactory.sh

# Add the PostgreSQL driver
COPY postgresql-${POSTGRESQL_VERSION}.jar ${ARTIFACTORY_HOME}/tomcat/lib/postgresql-${POSTGRESQL_VERSION}.jar

# Add internalUser plugin
COPY plugins/internalUser.groovy /tmp/plugins/internalUser.groovy

# Expose Tomcat's port
EXPOSE 8081

# The user that will run the container and artifactory
USER ${ARTIFACTORY_USER_NAME}

# Default mounts for data and extra config directories
VOLUME ${ARTIFACTORY_DATA}
VOLUME ${ARTIFACTORY_EXTRA_CONF}

# Start the simple standalone mode of Artifactory
ENTRYPOINT ["/entrypoint-artifactory.sh"]