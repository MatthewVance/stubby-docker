FROM debian:stretch as openssl
LABEL maintainer="Matthew Vance"

ENV version_openssl=openssl-1.1.1a \
    sha256_openssl=fc20130f8b7cbd2fb918b2f14e2f429e109c31ddd0fb38fc5d71d9ffed3f9f41 \
    source_openssl=https://www.openssl.org/source/ \
    opgp_openssl=8657ABB260F056B1E5190839D9C4D26D0E604491

WORKDIR /tmp/src

RUN set -e -x && \
    build_deps="build-essential ca-certificates curl dirmngr gnupg libidn2-0-dev libssl-dev" && \
    debian_frontend=noninteractive apt-get update && apt-get install -y --no-install-recommends \
      $build_deps && \
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
      $build_deps && \
    rm -rf \
      /tmp/* \
      /var/tmp/* \
      /var/lib/apt/lists/*

FROM debian:stretch as getdns
LABEL maintainer="Matthew Vance"

ENV version_getdns=getdns-1.5.0 \
    sha256_getdns=577182c3ace919ee70cee5629505581a10dc530bd53fe5c241603ea91c84fa84 \
    source_getdns=https://getdnsapi.net/dist/ \
    opgp_getdns=DC34EE5DB2417BCC151E5100E5F8F8212F77A498

WORKDIR /tmp/src

COPY --from=openssl /opt /opt


RUN set -e -x && \
    build_deps="\
                autoconf \
                build-essential \
                ca-certificates \
                curl \
                dh-autoreconf \
                dirmngr \
                gnupg \
                libssl-dev \
                libtool-bin \
                libyaml-dev \
                make \
                m4 \
               " && \
    debian_frontend=noninteractive apt-get update && apt-get install -y --no-install-recommends \
      $build_deps && \
    curl -L "${source_getdns}${version_getdns}.tar.gz" -o getdns.tar.gz && \
    echo "${sha256_getdns} ./getdns.tar.gz" | sha256sum -c - && \
    curl -L "${source_getdns}${version_getdns}.tar.gz.asc" -o getdns.tar.gz.asc && \
    GNUPGHOME="$(mktemp -d)" && \
    export GNUPGHOME && \
    ( gpg --no-tty --keyserver ipv4.pool.sks-keyservers.net --recv-keys "$opgp_getdns" \
    || gpg --no-tty --keyserver ha.pool.sks-keyservers.net --recv-keys "$opgp_getdns" ) && \
    gpg --batch --verify getdns.tar.gz.asc getdns.tar.gz && \
    tar xzf getdns.tar.gz && \
    cd "$version_getdns" && \
    libtoolize -ci && \
    autoreconf -fi && \
    mkdir build && \
    cd build && \
    ../configure \
        # --with-cc-opt="-O3 -fPIE -fstack-protector-strong -Wformat -Werror=format-security" \
        # --with-ld-opt="-Wl,-Bsymbolic-functions -Wl,-z,relro" \
        --without-libidn \
        --without-libidn2 \
        --enable-stub-only \
        --with-ssl=/opt/openssl && \
    make && \
    make install && \
    apt-get purge -y --auto-remove \
      $build_deps && \
    rm -rf \
      /tmp/* \
      /var/tmp/* \
      /var/lib/apt/lists/*

FROM debian:stretch
LABEL maintainer="Matthew Vance"

ENV version_stubby=0.2.3 \
    sha256_stubby=5fbe10f421f1313b5e2259b0eec96785af0c5b460431cf86bb6450a11cb3f473 \
    source_stubby=https://github.com/getdnsapi/stubby/archive/

EXPOSE 8053/udp

WORKDIR /tmp/src

COPY --from=openssl /opt /opt
COPY --from=getdns /usr/local /usr/local

RUN set -e -x && \
    build_deps="\
                autoconf \
                build-essential \
                ca-certificates \
                curl \
                dh-autoreconf \
                libssl-dev \
                libtool-bin \
                libyaml-dev \
                make \
                m4 \
               " && \
    debian_frontend=noninteractive apt-get update && apt-get install -y --no-install-recommends \
      $build_deps \
      ca-certificates \
      dns-root-data \
      ldnsutils \
      libev4 \
      libevent-core-2.0.5 \
      libidn11 \
      libuv1 \
      libyaml-0-2 && \
    curl -L "${source_stubby}v${version_stubby}.tar.gz" -o "stubby.tar.gz" && \
    echo "${sha256_stubby} stubby.tar.gz" | sha256sum -c - && \
    tar xzf stubby.tar.gz && \
    cd "stubby-${version_stubby}"
RUN set -e -x && pwd && ls -al && \
    cd "stubby-${version_stubby}" && ls -al && \
    libtoolize -ci && \
    autoreconf -fi && \
    mkdir build && \
    cd build && \
    ls -al && \
    ../configure \
        --prefix=/opt/stubby \
        # --with-cc-opt="-O3 -fPIE -fstack-protector-strong -Wformat -Werror=format-security" \
        # --with-ld-opt="-Wl,-Bsymbolic-functions -Wl,-z,relro" \
        CFLAGS="-I/usr/local/include" \
        LDFLAGS="-L/usr/local/lib" && \
    make && \
    make install && \
    groupadd -r stubby && \
    useradd --no-log-init -r -g stubby stubby && \
    apt-get purge -y --auto-remove \
      $build_deps && \
    rm -rf \
      /tmp/* \
      /var/tmp/* \
      /var/lib/apt/lists/*

WORKDIR /opt/stubby

ENV PATH /opt/stubby/bin:$PATH

USER stubby:stubby

COPY stubby.yml /opt/stubby/etc/stubby/stubby.yml

HEALTHCHECK --interval=5s --timeout=3s --start-period=5s CMD drill @127.0.0.1 cloudflare.com || exit 1

CMD ["/opt/stubby/bin/stubby"]
