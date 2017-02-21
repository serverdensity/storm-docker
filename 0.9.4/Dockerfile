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

WORKDIR $DISTRO_NAME

ENV PATH $PATH:/$DISTRO_NAME/bin

COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
