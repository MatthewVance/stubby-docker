FROM debian:stretch as builder
LABEL maintainer="Matthew Vance"

ENV version_openssl=openssl-1.1.1a \
    sha256_openssl=fc20130f8b7cbd2fb918b2f14e2f429e109c31ddd0fb38fc5d71d9ffed3f9f41 \
    source_openssl=https://www.openssl.org/source/ \
    opgp_openssl=8657ABB260F056B1E5190839D9C4D26D0E604491

WORKDIR /tmp/src

RUN set -e -x && \
    build_deps='build-essential ca-certificates curl dirmngr gnupg libidn2-0-dev libssl-dev' && \
    debian_frontend=noninteractive apt-get update && apt-get install -y --no-install-recommends \
      "$build_deps" && \
    curl -L "${source_openssl}${version_openssl}.tar.gz" -o openssl.tar.gz && \
    echo "${sha256_openssl} ./openssl.tar.gz" | sha256sum -c - && \
    curl -L "${source_openssl}${version_openssl}.tar.gz.asc" -o openssl.tar.gz.asc && \
    GNUPGHOME="$(mktemp -d)" && \
    export GNUPGHOME && \
    ( gpg --no-tty --keyserver ipv4.pool.sks-keyservers.net --recv-keys "$opgp_openssl" \
    || gpg --no-tty --keyserver ha.pool.sks-keyservers.net --recv-keys "$opgp_openssl" ) && \
    gpg --batch --verify openssl.tar.gz.asc openssl.tar.gz && \
    tar xzf openssl.tar.gz && \
    cd "$version_openssl" && \
    ./config --prefix=/opt/openssl no-weak-ssl-ciphers no-ssl3 no-shared enable-ec_nistp_64_gcc_128 -DOPENSSL_NO_HEARTBEATS -fstack-protector-strong && \
    make depend && \
    make && \
    make install_sw && \
    apt-get purge -y --auto-remove \
      "$build_deps" && \
    rm -rf \
      /tmp/* \
      /var/tmp/* \
      /var/lib/apt/lists/*

FROM debian:stretch
LABEL maintainer="Matthew Vance"

EXPOSE 8053/udp

WORKDIR /tmp/src

COPY --from=builder /opt/openssl /opt/openssl

RUN set -e -x && \
    build_deps='autoconf build-essential dh-autoreconf git libssl-dev libtool-bin libyaml-dev make m4' && \
    debian_frontend=noninteractive apt-get update && apt-get install -y --no-install-recommends \
      "$build_deps" \
      ca-certificates \
      dns-root-data \
      libev4 \
      libevent-core-2.0.5 \
      libidn11 \
      libuv1 \
      libyaml-0-2 && \
    git clone https://github.com/getdnsapi/getdns.git --branch develop && \
    cd getdns && \
    git submodule update --init && \
    libtoolize -ci && \
    autoreconf -fi && \
    mkdir build && \
    cd build && \
    ../configure --prefix=/opt/stubby --without-libidn --without-libidn2 --enable-stub-only --with-ssl=/opt/openssl --with-stubby && \
    make && \
    make install && \
    groupadd -r stubby && \
    useradd --no-log-init -r -g stubby stubby && \
    apt-get purge -y --auto-remove \
      "$build_deps" && \
    rm -rf \
      /tmp/* \
      /var/tmp/* \
      /var/lib/apt/lists/*

WORKDIR /opt/stubby

ENV PATH /opt/stubby/bin:$PATH

USER stubby:stubby

COPY stubby.yml /opt/stubby/etc/stubby/stubby.yml

CMD ["/opt/stubby/bin/stubby"]
