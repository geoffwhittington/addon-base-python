ARG BUILD_FROM=ghcr.io/hassio-addons/base:12.2.4
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Environment variables
ENV \
    PATH="/usr/local/bin:$PATH" \
    GPG_KEY="CFDCA245B1043CF2A5F97865FFE87404168BD847" \
    PYTHON_VERSION="3.10.7" \
    PYTHON_PIP_VERSION="22.2.2" \
    PYTHON_SETUPTOOLS_VERSION="65.3.0"

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Install base system
# hadolint ignore=DL3003,SC2155
RUN \
    apk add --no-cache --virtual .build-dependencies \
        bzip2-dev=1.0.8-r1 \
        dpkg-dev=1.21.8-r0 \
        dpkg=1.21.8-r0 \
        expat-dev=2.4.8-r0 \
        findutils=4.9.0-r0 \
        gcc=11.2.1_git20220219-r2 \
        gdbm-dev=1.23-r0 \
        gnupg=2.2.35-r4 \
        libc-dev=0.7.2-r3 \
        libffi-dev=3.4.2-r1 \
        libnsl-dev=2.0.0-r0 \
        libtirpc-dev=1.3.2-r1 \
        linux-headers=5.16.7-r1 \
        make=4.3-r0 \
        ncurses-dev=6.3_p20220521-r0 \
        openssl-dev=1.1.1q-r0 \
        pax-utils=1.3.4-r0 \
        readline-dev=8.1.2-r0 \
        sqlite-dev=3.38.5-r0 \
        tar=1.34-r0 \
        tcl-dev=8.6.12-r1 \
        tk-dev=8.6.12-r0 \
        tk=8.6.12-r0 \
        util-linux-dev=2.38-r1 \
        xz-dev=5.2.5-r1 \
        xz=5.2.5-r1 \
        zlib-dev=1.2.12-r3 \
    \
    && curl -J -L -o /tmp/python.tar.xz \
        "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz" \
    && curl -J -L -o /tmp/python.tar.xz.asc \
        "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz.asc" \
    \
    && export GNUPGHOME="$(mktemp -d)" \
    && gpg \
        --batch \
        --keyserver hkps://keys.openpgp.org \
        --recv-keys "$GPG_KEY" \
    && gpg \
        --batch \
        --verify /tmp/python.tar.xz.asc /tmp/python.tar.xz \
    && { command -v gpgconf > /dev/null && gpgconf --kill all || :; } \
    \
    && mkdir -p /usr/src/python \
    && tar -xJC /usr/src/python --strip-components=1 -f /tmp/python.tar.xz \
    && cd /usr/src/python \
    \
    && gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
    && ./configure \
        --build="$gnuArch" \
        --enable-loadable-sqlite-extensions \
        --enable-optimizations \
        --enable-option-checking=fatal \
        --enable-shared \
        --with-lto \
        --with-system-expat \
        --with-system-ffi \
        --without-ensurepip \
    \
    && make -j "$(nproc)" \
        EXTRA_CFLAGS="-DTHREAD_STACK_SIZE=0x100000" \
        LDFLAGS="-Wl,--strip-all" \
    && make install \
    \
    && find /usr/local \
        -type f \
        -executable \
        -not \( -name '*tkinter*' \) \
        -exec scanelf \
        --needed \
        --nobanner \
        --format '%n#p' '{}' ';' \
            | tr ',' '\n' \
            | sort -u \
            | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
            | xargs -rt apk add --no-cache --virtual .python-rundeps \
    \
    && cd /usr/local/bin \
    && ln -s idle3 idle \
    && ln -s pydoc3 pydoc \
    && ln -s python3 python \
    && ln -s python3-config python-config \
    \
    && curl -J -L -o /tmp/get-pip.py \
        'https://bootstrap.pypa.io/get-pip.py' \
    \
    && python /tmp/get-pip.py \
        --disable-pip-version-check \
        --no-cache-dir \
        --no-compile \
        "pip==$PYTHON_PIP_VERSION" \
        "setuptools==$PYTHON_SETUPTOOLS_VERSION" \
    \
    && apk del --no-cache --purge .build-dependencies \
    && rm -f -r \
        /usr/src \
        "$GNUPGHOME" \
        /tmp/* \
    \
    && python3 --version \
    && pip3 --version \
    \
    && find /usr -depth \
        \( \
            \( -type d -a \( -name test -o -name tests -o -name idle_test \) \) \
            -o \
            \( -type f -a \( -name '*.pyc' -o -name '*.pyo' -o -name 'libpython*.a' \) \) \
        \) -exec rm -rf '{}' +

# Entrypoint & CMD
ENTRYPOINT ["/init"]

# Build arugments
ARG BUILD_DATE
ARG BUILD_REF
ARG BUILD_VERSION
ARG BUILD_REPOSITORY

# Labels
LABEL \
    io.hass.name="Addon Python base for ${BUILD_ARCH}" \
    io.hass.description="Home Assistant Community Add-on: ${BUILD_ARCH} Python base image" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="base" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Franck Nijhof <frenck@addons.community>" \
    org.opencontainers.image.title="Addon Python base for ${BUILD_ARCH}" \
    org.opencontainers.image.description="Home Assistant Community Add-on: ${BUILD_ARCH} Python base image" \
    org.opencontainers.image.vendor="Home Assistant Community Add-ons" \
    org.opencontainers.image.authors="Franck Nijhof <frenck@addons.community>" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://addons.community" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/master/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
