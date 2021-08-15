FROM debian:sid

LABEL maintainer "fooinha@gmail.com"

# Build arguments
ARG DEBIAN_REPO_HOST=httpredir.debian.org

# Mirror to my location
RUN echo "deb http://${DEBIAN_REPO_HOST}/debian sid main" > /etc/apt/sources.list
RUN echo "deb-src http://${DEBIAN_REPO_HOST}/debian sid main" >> /etc/apt/sources.list

# Update
RUN DEBIAN_FRONTEND=noninteractive apt-get update || true

# Install build dependencies
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y --fix-missing \
    apt-utils \
    autoconf \
    automake \
    bind9-host \
    build-essential \
    dh-autoreconf \
    cpanminus \
    curl \
    devscripts \
    exuberant-ctags \
    git-core \
    jq \
    llvm \
    libgeoip1 \
    libgeoip-dev \
    libpcre3 \
    libpcre3-dbg \
    libpcre3-dev \
    libperl-dev \
    libmagic-dev \
    libtool \
    lsof \
    make \
    mercurial \
    ngrep \
    procps \
    python \
    telnet \
    tcpflow \
    valgrind \
    vim \
    wget \
    zlib1g \
    zlib1g-dev

# Create build directory
RUN mkdir -p /build

WORKDIR /build

# Get openssl master from git
RUN git clone https://github.com/openssl/openssl

# Build and install openssl
WORKDIR /build/openssl

RUN git checkout OpenSSL_1_1_1 -b patched
COPY patches/openssl.extensions.patch /build/openssl
RUN patch -p1 < openssl.extensions.patch
RUN ./config -d
RUN make
RUN make install

# Clone from nginx
WORKDIR /build
# COPY nginx ./nginx
RUN git clone https://github.com/Xetera/nginx-http2-fingerprint.git nginx

# Patch nginx for fetching ssl client extensions
WORKDIR /build/nginx
COPY patches/nginx.latest.patch /build/nginx
RUN patch -p1 < nginx.latest.patch

# Install files
RUN mkdir -p /usr/local/nginx/conf/

# Install self-signed certificate
RUN LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib /usr/local/bin/openssl req -new -x509 -days 365 -nodes -out /usr/local/nginx/conf/cert.pem -keyout /usr/local/nginx/conf/rsa.key -subj "/C=PT/ST=Lisbon/L=Lisbon/O=Development/CN=foo.local"

RUN echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib' | tee -a /root/.bashrc
RUN echo 'export PATH=$PATH:/usr/local/bin:/usr/local/nginx/sbin' | tee -a /root/.bashrc
RUN echo '' | tee -a /root/.bashrc
RUN echo 'export ASAN_OPTIONS=symbolize=1' | tee -a /root/.bashrc
RUN echo 'export export ASAN_SYMBOLIZER_PATH=/usr/bin/llvm-symbolizer' | tee -a /root/.bashrc
RUN echo '' | tee -a /root/.bashrc

WORKDIR /build
COPY docker/compile.sh /build/compile.sh

COPY config /build/nginx-ssl-ja3/config
COPY src /build/nginx-ssl-ja3/src
COPY patches /build/nginx-ssl-ja3/patches 

RUN ./compile.sh

COPY docker/nginx.conf /usr/local/nginx/conf/nginx.conf
CMD ["/usr/local/nginx/sbin/nginx"]
