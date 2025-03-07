# Special Dockerfile to build all Ubuntu targets, the only difference is
# the packages in the base image.
# Set this to the base image to use in each case, so if we want to build for amazonlinux/2.arm64v8
# we would set BASE_BUILDER=amazonlinux-2.arm64v8-base.
ARG BASE_BUILDER
# Lookup the name to use below but should follow the '<distro>-base' convention with slashes replaced.
# Use buildkit to skip unused base images: DOCKER_BUILDKIT=1

# Multiarch support
FROM multiarch/qemu-user-static:x86_64-aarch64 as multiarch-aarch64

# amazonlinux/2 base image
FROM amazonlinux:2 as amazonlinux-2-base

# hadolint ignore=DL3033
RUN yum -y update && \
    yum install -y rpm-build curl ca-certificates gcc gcc-c++ cmake make bash \
                   wget unzip systemd-devel wget flex bison \
                   cyrus-sasl-lib cyrus-sasl-devel openssl openss-libs openssl-devel \
                   postgresql-devel postgresql-libs \
                   cmake3 && \
    yum clean all

# amazonlinux/2.arm64v8 base image
FROM arm64v8/amazonlinux:2 as amazonlinux-2.arm64v8-base

COPY --from=multiarch-aarch64 /usr/bin/qemu-aarch64-static /usr/bin/qemu-aarch64-static

# hadolint ignore=DL3033
RUN yum -y update && \
    yum install -y rpm-build curl ca-certificates gcc gcc-c++ cmake make bash \
                   wget unzip systemd-devel wget flex bison \
                   cyrus-sasl-lib cyrus-sasl-devel openssl openss-libs openssl-devel \
                   postgresql-devel postgresql-libs \
                   cmake3 && \
    yum clean all

# Common build for all distributions now
# hadolint ignore=DL3006
FROM $BASE_BUILDER as builder
ARG FLB_PREFIX
ARG FLB_VERSION
ARG FLB_SRC

ENV FLB_PREFIX=$FLB_PREFIX
ENV FLB_VERSION=$FLB_VERSION
ENV FLB_SRC=$FLB_SRC

ENV FLB_TARBALL http://github.com/fluent/fluent-bit/archive/$FLB_PREFIX$FLB_VERSION.zip
COPY sources/$FLB_SRC /

WORKDIR /tmp
RUN if [ -z "$FLB_SRC" ] ; then wget -q -O "/tmp/fluent-bit-${FLB_VERSION}.zip" ${FLB_TARBALL} && unzip "fluent-bit-$FLB_VERSION.zip" ; else tar zxfv "/$FLB_SRC" ; fi

# CMake configuration variables
# Unused
ARG CFLAGS
ARG CMAKE_INSTALL_PREFIX=/opt/td-agent-bit/
ARG CMAKE_INSTALL_SYSCONFDIR=/etc/
ARG FLB_TD=On
ARG FLB_RELEASE=On
ARG FLB_TRACE=On
ARG FLB_SQLDB=On
ARG FLB_HTTP_SERVER=On
ARG FLB_OUT_KAFKA=On
ARG FLB_OUT_PGSQL=On

WORKDIR /tmp/fluent-bit-$FLB_VERSION/build/
RUN cmake3 -DCMAKE_INSTALL_PREFIX="$CMAKE_INSTALL_PREFIX" \
           -DCMAKE_INSTALL_SYSCONFDIR="$CMAKE_INSTALL_SYSCONFDIR" \
           -DFLB_RELEASE="$FLB_RELEASE" \
           -DFLB_TRACE="$FLB_TRACE" \
           -DFLB_TD="$FLB_TD" \
           -DFLB_SQLDB="$FLB_SQLDB" \
           -DFLB_HTTP_SERVER="$FLB_HTTP_SERVER" \
           -DFLB_OUT_KAFKA="$FLB_OUT_KAFKA" \
           -DFLB_OUT_PGSQL="$FLB_OUT_PGSQL" \
           ../

VOLUME [ "/output" ]
CMD [ "/bin/bash", "-c", "make -j 4 && cpack3 -G RPM && cp *.rpm /output/" ]
