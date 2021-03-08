################################################
# Dockerfile to build OpenWAF container images #
# Based on centos 7                            #
# https://github.com/titansec/docker-openwaf   #
################################################

#Set the base image to centos
FROM centos:7

#File Author
MAINTAINER Miracle

#Docker Build Arguments
ARG OPENWAF_VERSION="v1.1"
ARG OPENWAF_PREFIX="/opt"
ARG OPENRESTY_PREFIX="/usr/local/openresty"
ARG OPENRESTY_VERSION="1.15.8.2"
ARG CIDR_VERSION="1.2.3"
ARG PCRE_VERSION="8.43"
ARG OPENSSL_VERSION="1.1.1d"
ARG OPENWAF_CONFIG_OPTIONS=" \ 
    --with-pcre-jit --with-ipv6 \ 
    --with-http_stub_status_module \ 
    --with-http_ssl_module \ 
    --with-http_realip_module \ 
    --with-http_sub_module \ 
    --with-http_geoip_module \ 
    --with-http_v2_module \ 
    --with-openssl=${OPENWAF_PREFIX}/openssl-${OPENSSL_VERSION} \ 
    --with-pcre=${OPENWAF_PREFIX}/pcre-${PCRE_VERSION} \ 
    "

RUN yum install gcc gcc-c++ wget -y

#1.Install openrestry related
RUN yum install make perl perl-ExtUtils-Embed readline-devel zlib-devel -y \
    && cd ${OPENWAF_PREFIX} \
    && wget http://www.over-yonder.net/~fullermd/projects/libcidr/libcidr-${CIDR_VERSION}.tar.xz \
    && wget https://ftp.pcre.org/pub/pcre/pcre-${PCRE_VERSION}.tar.gz \
    && wget https://www.openssl.org/source/openssl-${OPENSSL_VERSION}.tar.gz \
    && wget https://openresty.org/download/openresty-${OPENRESTY_VERSION}.tar.gz \
    && tar -xvf libcidr-${CIDR_VERSION}.tar.xz \
    && tar -zxvf pcre-${PCRE_VERSION}.tar.gz \
    && tar -zxvf openssl-${OPENSSL_VERSION}.tar.gz \
    && tar -zxvf openresty-${OPENRESTY_VERSION}.tar.gz \
    && cd ${OPENWAF_PREFIX}/libcidr-${CIDR_VERSION} \
    && make && make install

#2. Install OpenWAF
RUN cd ${OPENWAF_PREFIX} \ 
    && yum install GeoIP-devel git swig -y \ 
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
    && ./configure ${OPENWAF_CONFIG_OPTIONS} \
    && make \
    && make install 

#4. Cleanup
RUN cd ${OPENWAF_PREFIX} \ 
    && rm -rf \ 
        pcre-${PCRE_VERSION} \ 
        libcidr-${CIDR_VERSION} \ 
        openssl-${OPENSSL_VERSION} \ 
        pcre-${PCRE_VERSION}.tar.gz \ 
        openresty-${OPENRESTY_VERSION} \ 
        openssl-${OPENSSL_VERSION}.tar.gz \ 
        openresty-${OPENRESTY_VERSION}.tar.gz

ENV PATH=${OPENRESTY_PREFIX}/nginx/sbin/:${OPENRESTY_PREFIX}/bin/:$PATH

CMD ["openresty", "-c", "/etc/ngx_openwaf.conf", "-g", "daemon off;"]

# Use SIGQUIT instead of default SIGTERM to cleanly drain requests
# See https://github.com/openresty/docker-openresty/blob/master/README.md#tips--pitfalls
STOPSIGNAL SIGQUIT