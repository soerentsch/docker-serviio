# Serviio docker
#
# Run with: docker run --rm --name serviio -d -p 23423:23423/tcp -p 23424:23424/tcp -p 8895:8895/tcp -p 1900:1900/udp -v /etc/localtime:/etc/localtime:ro soerentsch/serviio
ARG ALPINE_VERSION=3.18.0

FROM alpine:${ALPINE_VERSION}

ARG BUILD_DATE
ARG BUILD_VCS_REF

ARG SERVIIO_VERSION=2.3

LABEL \
	org.label-schema.build-date="${BUILD_DATE}" \
	org.label-schema.description="DLNA Serviio Container" \
	org.label-schema.name="DLNA Serviio Container" \
	org.label-schema.schema-version="1.0" \
	org.label-schema.url="https://hub.docker.com/r/soerentsch/serviio/" \
	org.label-schema.vcs-ref="${BUILD_VCS_REF}" \
	org.label-schema.vcs-url="https://github.dev/soerentsch/docker-serviio/" \
	org.label-schema.vendor="[soerentsch] Soeren <soerentsch@gmail.com>" \
	org.label-schema.version="${SERVIIO_VERSION}" \
	maintainer="[soerentsch] Soeren <soerentsch@gmail.com>"

LABEL \
	org.opencontainers.image.created="${BUILD_DATE}" \
	org.opencontainers.image.description="DLNA Serviio Container" \
	org.opencontainers.image.title="DLNA Serviio Container" \
	org.opencontainers.image.url="https://hub.docker.com/r/soerentsch/serviio/" \
	org.opencontainers.image.revision="${BUILD_VCS_REF}" \
	org.opencontainers.image.source="https://github.dev/soerentsch/docker-serviio/" \
	org.opencontainers.image.vendor="[soerentsch] Soeren <soerentsch@gmail.com>" \
	org.opencontainers.image.version="${SERVIIO_VERSION}" \
	org.opencontainers.image.authors="[soerentsch] Soeren <soerentsch@gmail.com>"

# Prepare APK CDNs
RUN set -ex \
	&& echo "https://dl-cdn.alpinelinux.org/alpine/edge/testing" >> /etc/apk/repositories \
	&& echo "https://dl-cdn.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories \
	&& echo "https://dl-cdn.alpinelinux.org/alpine/edge/main" >> /etc/apk/repositories \
	&& apk update && apk upgrade \
	&& apk add --no-cache --update \
		ffmpeg \
		jasper \
	&& apk add --no-cache --update --virtual=.build-dependencies \
		autoconf \
		bash \
		build-base \
		clang \
		g++ \ 
		git \
		jasper-dev \
		lcms2-dev

# Clone OpenJDK RISC-V repository
RUN git clone --depth 1 --single-branch --branch master https://github.com/openjdk/riscv-port.git

# Set environment variables for OpenJDK build
ENV JAVA_HOME=/opt/openjdk
ENV PATH="$JAVA_HOME/bin:${PATH}"

# Build OpenJDK
WORKDIR /app/riscv-port
RUN set -ex \
	&& bash configure --with-target-bits=64 \
    	&& make images

### Create new WORKDIR and get all ingredients		
RUN set -ex \
	&& DIR=$(mktemp -d) && cd ${DIR} \
	&& wget https://raw.githubusercontent.com/soerentsch/dcraw/master/dcraw.c \
	&& wget https://download.serviio.org/releases/serviio-${SERVIIO_VERSION}-linux.tar.gz && tar xvf serviio-${SERVIIO_VERSION}-linux.tar.gz \
### Build dcraw	
	&& cd ${DIR} \
	&& gcc -o dcraw -O4 dcraw.c -lm -ljasper -ljpeg -llcms2 \
	&& cp dcraw /usr/bin/dcraw \
	&& chmod +x /usr/bin/dcraw \
### Install Serviio	
	&& cd ${DIR} \
	&& mkdir -p /opt/serviio \
	&& mkdir -p /media/serviio \
	&& mv ./serviio-${SERVIIO_VERSION}/* /opt/serviio \
	&& chmod +x /opt/serviio/bin/serviio.sh \
	&& mkdir -p /opt/serviio/log \
	&& touch /opt/serviio/log/serviio.log \
### Cleanup	
	&& rm -rf ${DIR} \
	&& apk del --purge .build-dependencies \
	&& rm -rf /var/cache/apk/*

VOLUME ["/opt/serviio/config", "/opt/serviio/library",  "/opt/serviio/log", "/opt/serviio/plugins", "/media/serviio"]

# DLNA
EXPOSE 1900/udp
# Serviio Content Delivery
EXPOSE 8895/tcp
# HTTP/1.1 /console /rest
EXPOSE 23423/tcp 
# HTTP/1.1 /cds /mediabrowser
EXPOSE 23424/tcp
# HTTPS/1.1 /console /rest
EXPOSE 23523/tcp
# HTTPS/1.1 /cds /mediabrowser
EXPOSE 23524/tcp

HEALTHCHECK NONE

CMD tail -f /opt/serviio/log/serviio.log & /opt/serviio/bin/serviio.sh
