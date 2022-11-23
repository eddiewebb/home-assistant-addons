ARG BUILD_FROM=ghcr.io/hassio-addons/base:12.2.7
FROM $BUILD_FROM
# Setup base
ARG BUILD_ARCH=amd64

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Add env
ENV TERM="xterm-256color"


# DEPENDENCIES TO REMOVE AFTER
RUN  set -x \
  \
  &&  apk add --no-cache --virtual .build-dependencies \
        bsd-compat-headers \
        build-base \
        cmake \
        json-c-dev \
        libffi-dev \
        perl-dev \
        perl-utils \
        libuv-dev \
        libxml2-dev \
        openssl-dev \
        python3-dev \
        zlib-dev 

# Core Utilities    
    RUN apk add --no-cache \
        git \
        json-c \
        libltdl \
        libuv \
        libxml2 \
        openssh \
        openssl \
        mutagen \
        pwgen \
        py3-pip \
        python3 \
        tmux \
        vim \
        wget \
        zip \
        zsh 

# Experimental tools (rippers)
COPY rootfs/etc/apk /etc/apk

RUN apk add libdvdcss \
      flac \
      lame \
  handbrake \
      abcde \
      cdparanoia 

# Web Terminal
RUN sed -i -e "s#bin/ash#bin/zsh#" /etc/passwd \
    \
    && git clone --branch "v4.3.2" --depth=1 \
        https://github.com/warmcat/libwebsockets.git /tmp/libwebsockets \
    \
    && mkdir -p /tmp/libwebsockets/build \
    && cd /tmp/libwebsockets/build \
    && cmake .. \
        -DCMAKE_BUILD_TYPE=MinSizeRel \
        -DCMAKE_INSTALL_PREFIX=/usr \
        -DCMAKE_VERBOSE_MAKEFILE=TRUE \
        -DLWS_IPV6=ON \
        -DLWS_STATIC_PIC=ON \
        -DLWS_UNIX_SOCK=OFF \
        -DLWS_WITH_LIBUV=ON \
        -DLWS_WITH_SHARED=ON \
        -DLWS_WITHOUT_TESTAPPS=ON \
    && make \
    && make install \
    \
    && git clone --branch main --single-branch \
        https://github.com/tsl0922/ttyd.git /tmp/ttyd \
    && git -C /tmp/ttyd checkout "1.7.2" \
    \
    && mkdir -p /tmp/ttyd/build \
    && cd /tmp/ttyd/build \
    && cmake .. \
        -DCMAKE_BUILD_TYPE=MinSizeRel \
        -DCMAKE_INSTALL_PREFIX=/usr \
        -DCMAKE_VERBOSE_MAKEFILE=TRUE \
    && make \
    && make install

RUN pip3 install -U \
    eyed3 \
    pylast==2.4.0 
  
## MP3 Disc ID snd CDDB lookup  
RUN curl -SLo /tmp/libdiscid.tar.gz http://ftp.musicbrainz.org/pub/musicbrainz/libdiscid/libdiscid-0.6.2.tar.gz \
  && cd /tmp \
  && tar xvf libdiscid.tar.gz \
  && cd libdiscid-0.6.2 \
  && ./configure \
  && make \
  && make install 
RUN perl -MCPAN -e "CPAN::Shell->notest('install', 'IO::Socket::SSL', '>= 2.009')" \
  &&  perl -MCPAN -e "CPAN::Shell->notest('install', 'XML::LibXML')" \
  &&  perl -MCPAN -e "CPAN::Shell->notest('install', 'YAML')" \
  &&  perl -MCPAN -e "CPAN::Shell->notest('install', 'WebService::MusicBrainz')" \
  &&  perl -MCPAN -e "CPAN::Shell->notest('install', 'MusicBrainz::DiscID')" \
  &&  perl -MCPAN -e "CPAN::Shell->notest('install', 'MusicBrainz')" 



# Remove depencenies
RUN apk del .build-dependencies

COPY rootfs /  
RUN \
    chmod a+x /configure.sh \
 && chmod a+x -R /etc/services.d

COPY abcde.conf /etc/abcde.conf
