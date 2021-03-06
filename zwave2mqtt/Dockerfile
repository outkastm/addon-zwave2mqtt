ARG BUILD_FROM=hassioaddons/base:7.2.0
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Setup base
# hadolint ignore=DL3003
RUN \
    apk add --no-cache --virtual .build-dependencies \
        build-base=0.5-r1 \
        git=2.24.3-r0 \
        linux-headers=4.19.36-r0 \
        npm=12.15.0-r1 \
        python2-dev~=2.7 \
        yarn=1.19.2-r0 \
        coreutils=8.31-r0 \
    \
    && apk add --no-cache \
        eudev=3.2.9-r1 \
        libusb=1.0.23-r0 \
        lua-resty-http=0.15-r0 \
        nginx-mod-http-lua=1.16.1-r6 \
        nginx=1.16.1-r6 \
        nodejs=12.15.0-r1 \
        openssl=1.1.1g-r0 \
    \
    && OPENZWAVE_VER=1.6.1123 \
    && mkdir /tmp/openzwavelib \
    && curl -J -L -o /tmp/openzwavelib.tar.gz \
        "http://old.openzwave.com/downloads/openzwave-${OPENZWAVE_VER}.tar.gz" \
    && tar zxvf \
        /tmp/openzwavelib.tar.gz \
        --strip 1 -C /tmp/openzwavelib \
    && cd /tmp/openzwavelib \
    && echo "start build openzavelib" \
    && make \
    && echo "start install openzavelib" \
    && make install PREFIX=/usr \
    && echo "copy database" \
    && rm -f -r /etc/openzwave \
    && mkdir -p /etc/openzwave \
    && mv /tmp/openzwavelib/config/* /etc/openzwave \
    \
    && echo "download zwave2mqtt" \
    && curl -J -L -o /tmp/zwave2mqtt.tar.gz \
        "https://github.com/OpenZWave/Zwave2Mqtt/archive/v3.2.2.tar.gz" \
    && tar zxvf \
        /tmp/zwave2mqtt.tar.gz \
        --strip 1 -C /opt \
    \
    && cd /opt \
    && npm config set unsafe-perm \
    && npm install \
    && npm run build \
    && npm prune --production \
    \
    && yarn global add modclean \
    && modclean \
        --path /opt \
        --no-progress \
        --keep-empty \
        --run \
    && yarn global remove modclean \
    && yarn cache clean \
    \
    && apk del --no-cache --purge .build-dependencies \
    \
    && rm -f -r \
        /etc/nginx \
        /root/.node-gyp \
        /root/.npm \
        /tmp/.[!.]* \
        /tmp/* \
        /usr/lib/node_modules \
        /usr/local/share/.cache

# Copy root filesystem
COPY rootfs /

# Build arguments
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_REF
ARG BUILD_VERSION

# Labels
LABEL \
    io.hass.name="Z-Wave to MQTT dev" \
    io.hass.description="Fully configurable Z-Wave to MQTT gateway and control panel" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Irinel Porcariu <irinel.porcariu@gmail.com>" \
    org.opencontainers.image.title="Z-Wave to MQTT" \
    org.opencontainers.image.description="Fully configurable Z-Wave to MQTT gateway and control panel" \
    org.opencontainers.image.vendor="Home Assistant Community Add-ons" \
    org.opencontainers.image.authors="Irinel Porcariu <irinel.porcariu@gmail.com>" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="" \
    org.opencontainers.image.source="https://github.com/outkastm/my-hassio-addons" \
    org.opencontainers.image.documentation="https://github.com/outkastm/my-hassio-addons/blob/master/zwave2mqtt-okm/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
	