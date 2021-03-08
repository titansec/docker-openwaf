################################################
# Dockerfile to build OpenWAF container images #
# Based on alpine 3.8                          #
# https://github.com/titansec/docker-openwaf   #
################################################

#Set the base image to alpine
#ARG RESTY_IMAGE_BASE="alpine"
#ARG RESTY_IMAGE_TAG="3.8"
#FROM ${RESTY_IMAGE_BASE}:${RESTY_IMAGE_TAG}
FROM alpine:3.8

#File Author
MAINTAINER Miracle

# Docker Build Arguments
ARG OPENWAF_VERSION="v1.1"
ARG OPENWAF_PREFIX="/opt"
ARG OPENRESTY_PREFIX="/usr/local/openresty"
ARG OPENRESTY_VERSION="1.15.8.2"
ARG CIDR_VERSION="1.2.3"
ARG PCRE_VERSION="8.43"
ARG OPENSSL_VERSION="1.1.1d"
ARG OPENWAF_J="1"
ARG OPENWAF_CONFIG_OPTIONS=" \ 
    --with-pcre-jit --with-ipv6 \ 
    --with-http_stub_status_module \ 
    --with-http_ssl_module \ 
    --with-http_realip_module \ 
    --with-http_sub_module \ 
    --with-http_geoip_module \ 
    --with-http_v2_module \ 
    --with-pcre=${OPENWAF_PREFIX}/pcre-${PCRE_VERSION} \ 
    "
    
#1.Install openrestry related
RUN apk add --no-cache --virtual .build-deps \
        build-base \
        curl \
        gd-dev \
        geoip-dev \
        libxslt-dev \
        linux-headers \
        make \
        perl-dev \
        readline-dev \
        zlib-dev \
        coreutils \ 
    && apk add --no-cache \
        gd \
        geoip \
        libgcc \
        libxslt \
        zlib \
    && mkdir ${OPENWAF_PREFIX} \
    && cd ${OPENWAF_PREFIX} \
    && curl -fSL http://www.over-yonder.net/~fullermd/projects/libcidr/libcidr-${CIDR_VERSION}.tar.xz -o libcidr-${CIDR_VERSION}.tar.xz \ 
    && curl -fSL https://ftp.pcre.org/pub/pcre/pcre-${PCRE_VERSION}.tar.gz -o pcre-${PCRE_VERSION}.tar.gz \
    && curl -fSL https://www.openssl.org/source/openssl-${OPENSSL_VERSION}.tar.gz -o openssl-${OPENSSL_VERSION}.tar.gz \
    && curl -fSL https://openresty.org/download/openresty-${OPENRESTY_VERSION}.tar.gz -o openresty-${OPENRESTY_VERSION}.tar.gz \
    && tar xvf libcidr-${CIDR_VERSION}.tar.xz \
    && tar xzf pcre-${PCRE_VERSION}.tar.gz \
    && tar xzf openssl-${OPENSSL_VERSION}.tar.gz \
    && tar xzf openresty-${OPENRESTY_VERSION}.tar.gz \
    && cd ${OPENWAF_PREFIX}/libcidr-${CIDR_VERSION} \
    && make && make install

RUN cd ${OPENWAF_PREFIX}/openssl-${OPENSSL_VERSION} \
    && ./config no-async \
    && make \
    && make install
    
#2. Install OpenWAF
RUN cd ${OPENWAF_PREFIX} \
    && apk add --no-cache --virtual .openwaf_build_deps git \
    && apk add --no-cache swig \
    && git clone --branch ${OPENWAF_VERSION} https://github.com/titansec/OpenWAF.git \ 
    && mv ${OPENWAF_PREFIX}/OpenWAF/lib/openresty/ngx_openwaf.conf /etc \ 
    && mv ${OPENWAF_PREFIX}/OpenWAF/lib/openresty/configure ${OPENWAF_PREFIX}/openresty-${OPENRESTY_VERSION} \ 
    && cp -RP ${OPENWAF_PREFIX}/OpenWAF/lib/openresty/* ${OPENWAF_PREFIX}/openresty-${OPENRESTY_VERSION}/bundle/ \ 
    && cd ${OPENWAF_PREFIX}/OpenWAF \ 
    && make clean \
    && make install \
    && ln -s /usr/local/lib/libcidr.so /opt/OpenWAF/lib/resty/libcidr.so
    
#3. Build openresty
RUN cd ${OPENWAF_PREFIX}/openresty-${OPENRESTY_VERSION}/ \	
    && ./configure -j${OPENWAF_J} ${OPENWAF_CONFIG_OPTIONS} \
    && make -j${OPENWAF_J} \
    && make -j${OPENWAF_J} install
    
#4. Cleanup
RUN cd ${OPENWAF_PREFIX} \ 
    && rm -rf \ 
        pcre-${PCRE_VERSION} \ 
        libcidr-${CIDR_VERSION} \ 
        openssl-${OPENSSL_VERSION} \ 
        pcre-${PCRE_VERSION}.tar.gz \ 
        openresty-${OPENRESTY_VERSION} \ 
        openssl-${OPENSSL_VERSION}.tar.gz \ 
        openresty-${OPENRESTY_VERSION}.tar.gz \
        OpenWAF/doc \
        OpenWAF/lib/openresty \
    && cd OpenWAF \
    && rm -rf `ls -Fa | grep '^\.\w'` \
    && rm -f `ls -F | grep -v '/$'` \
    && apk del .build-deps \
    && apk del .openwaf_build_deps
    

# Add additional binaries into PATH for convenience
ENV PATH=${OPENRESTY_PREFIX}/luajit/bin/:${OPENRESTY_PREFIX}/nginx/sbin/:${OPENRESTY_PREFIX}/bin/:$PATH

CMD ["openresty", "-c", "/etc/ngx_openwaf.conf", "-g", "daemon off;"]

# Use SIGQUIT instead of default SIGTERM to cleanly drain requests
# See https://github.com/openresty/docker-openresty/blob/master/README.md#tips--pitfalls
STOPSIGNAL SIGQUIT