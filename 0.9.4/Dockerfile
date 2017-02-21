FROM openjdk:8-jre-alpine

# Install required packages
RUN apk add --no-cache \
    bash \
    python \
    su-exec

ENV STORM_USER=storm \
    STORM_CONF_DIR=/conf \
    STORM_DATA_DIR=/data \
    STORM_LOG_DIR=/logs

# Add a user and make dirs
RUN set -x \
    && adduser -D "$STORM_USER" \
    && mkdir -p "$STORM_CONF_DIR" "$STORM_DATA_DIR" "$STORM_LOG_DIR" \
    && chown -R "$STORM_USER:$STORM_USER" "$STORM_CONF_DIR" "$STORM_DATA_DIR" "$STORM_LOG_DIR"

ARG GPG_KEY=ACEFE18DD2322E1E84587A148DE03962E80B8FFD
ARG DISTRO_NAME=apache-storm-0.9.4

# Download Apache Storm, verify its PGP signature, untar and clean up
RUN set -x \
    && apk add --no-cache --virtual .build-deps \
        gnupg \
    && wget -q "http://www.apache.org/dist/storm/$DISTRO_NAME/$DISTRO_NAME.tar.gz" \
    && wget -q "http://www.apache.org/dist/storm/$DISTRO_NAME/$DISTRO_NAME.tar.gz.asc" \
    && export GNUPGHOME="$(mktemp -d)" \
    && gpg --keyserver ha.pool.sks-keyservers.net --recv-key "$GPG_KEY" \
    && gpg --batch --verify "$DISTRO_NAME.tar.gz.asc" "$DISTRO_NAME.tar.gz" \
    && tar -xzf "$DISTRO_NAME.tar.gz" \
    && chown -R "$STORM_USER:$STORM_USER" "$DISTRO_NAME" \
    && rm -r "$GNUPGHOME" "$DISTRO_NAME.tar.gz" "$DISTRO_NAME.tar.gz.asc" \
    && apk del .build-deps

# Download topologies requirements
# Zookeeper
ARG ZOOKEEPER_VERSION=3.4.6

RUN set -x \
    && apk add --no-cache --virtual .build-deps \
        gnupg \
    && ZOO_TEMP="$(mktemp -d)" \
    && wget -q -P "$ZOO_TEMP" "http://www.apache.org/dist/zookeeper/zookeeper-$ZOOKEEPER_VERSION/zookeeper-$ZOOKEEPER_VERSION.tar.gz" \
    && wget -q -P "$ZOO_TEMP" "http://www.apache.org/dist/zookeeper/zookeeper-$ZOOKEEPER_VERSION/zookeeper-$ZOOKEEPER_VERSION.tar.gz.asc" \
    && export GNUPGHOME="$(mktemp -d)" \
    && gpg --keyserver ha.pool.sks-keyservers.net --recv-key 1D3D275F61D8E2B589C19FD093FB0254D2C80E32 \
    && gpg --batch --verify "$ZOO_TEMP/zookeeper-$ZOOKEEPER_VERSION.tar.gz.asc" "$ZOO_TEMP/zookeeper-$ZOOKEEPER_VERSION.tar.gz" \
    && tar -xzf "$ZOO_TEMP/zookeeper-$ZOOKEEPER_VERSION.tar.gz" -C "$ZOO_TEMP" \
    && cp "$ZOO_TEMP/zookeeper-$ZOOKEEPER_VERSION/zookeeper-$ZOOKEEPER_VERSION.jar" "/$DISTRO_NAME/lib/" \
    && chown -R "$STORM_USER:$STORM_USER" "/$DISTRO_NAME/lib" \
    && rm -r "$GNUPGHOME" "$ZOO_TEMP" \
    && apk del .build-deps

# Kafka
ARG KAFKA_VERSION=0.8.2.0

RUN set -x \
    && KAFKA_TEMP="$(mktemp -d)" \
    && wget -q "http://mirror.symnds.com/software/Apache/kafka/$KAFKA_VERSION/kafka_2.10-$KAFKA_VERSION.tgz" -O - | /bin/tar -zxf - "kafka_2.10-$KAFKA_VERSION/libs/scala-library-2.10.4.jar" "kafka_2.10-$KAFKA_VERSION/libs/kafka_2.10-$KAFKA_VERSION.jar" "kafka_2.10-$KAFKA_VERSION/libs/metrics-core-2.2.0.jar" "kafka_2.10-$KAFKA_VERSION/libs/kafka-clients-$KAFKA_VERSION.jar" -C "$KAFKA_TEMP" \
    && cp "$KAFKA_TEMP/kafka_2.10-$KAFKA_VERSION/libs/kafka-clients-$KAFKA_VERSION.jar" "$KAFKA_TEMP/kafka_2.10-$KAFKA_VERSION/libs/metrics-core-2.2.0.jar" "$KAFKA_TEMP/kafka_2.10-$KAFKA_VERSION/libs/scala-library-2.10.4.jar" "/$DISTRO_NAME/lib" \
    && rm -r "$KAFKA_TEMP"

RUN wget -q http://central.maven.org/maven2/org/apache/curator/curator-client/2.5.0/curator-client-2.5.0.jar -O "/$DISTRO_NAME/lib/curator-client-2.5.0.jar"
RUN wget -q http://central.maven.org/maven2/org/apache/curator/curator-framework/2.5.0/curator-framework-2.5.0.jar -O "/$DISTRO_NAME/lib/curator-framework-2.5.0.jar"
RUN wget -q http://central.maven.org/maven2/org/apache/curator/curator-recipes/2.5.0/curator-recipes-2.5.0.jar -O "/$DISTRO_NAME/lib/curator-recipes-2.5.0.jar"
RUN wget -q http://central.maven.org/maven2/com/google/guava/guava/17.0/guava-17.0.jar -O "/$DISTRO_NAME/lib/guava-17.0.jar"
RUN chown -R "$STORM_USER:$STORM_USER" "/$DISTRO_NAME/lib"

WORKDIR $DISTRO_NAME

ENV PATH $PATH:/$DISTRO_NAME/bin

COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
