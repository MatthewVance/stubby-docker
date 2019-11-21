FROM debian:buster as openssl
LABEL maintainer="Matthew Vance"

ENV version_openssl=openssl-1.1.1c \
    sha256_openssl=f6fb3079ad15076154eda9413fed42877d668e7069d9b87396d0804fdb3f4c90 \
    source_openssl=https://www.openssl.org/source/ \
    opgp_openssl=7953AC1FBC3DC8B3B292393ED5E9E43F7DF9EE8C

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
    ./config --prefix=/opt/openssl no-weak-ssl-ciphers no-ssl3 no-shared -DOPENSSL_NO_HEARTBEATS -fstack-protector-strong && \
    make depend && \
    make && \
    make install_sw && \
    apt-get purge -y --auto-remove \
      $build_deps && \
    rm -rf \
      /tmp/* \
      /var/tmp/* \
      /var/lib/apt/lists/*

FROM debian:buster as unbound
LABEL maintainer="Matthew Vance"

ENV unbound_version=1.9.1 \
    unbound_sha256=c3c0bf9b86ccba4ca64f93dd4fe7351308ab54293f297a67de5a8914c1dc59c5 \
    unbound_download_url="https://nlnetlabs.nl/downloads/unbound/unbound-1.9.1.tar.gz"

WORKDIR /tmp/src

COPY --from=openssl /opt/openssl /opt/openssl

RUN build_deps="ca-certificates curl gcc libc-dev libevent-dev libexpat1-dev make" && \
    set -x && \
    debian_frontend=noninteractive apt-get update && apt-get install -y --no-install-recommends \
      $build_deps \
      bsdmainutils \
      ldnsutils \
      libevent-2.1-6 \
      libexpat1 && \
    curl -sSL "${unbound_download_url}" -o unbound.tar.gz && \
    echo "${unbound_sha256} *unbound.tar.gz" | sha256sum -c - && \
    tar xzf unbound.tar.gz && \
    rm -f unbound.tar.gz && \
    cd unbound-"${unbound_version}" && \
    groupadd _unbound && \
    useradd -g _unbound -s /etc -d /dev/null _unbound && \
    ./configure \
        --disable-dependency-tracking \
        --prefix=/opt/unbound \
        --with-pthreads \
        --with-username=_unbound \
        --with-ssl=/opt/openssl \
        --with-libevent \
        --enable-event-api && \
    make install && \
    mv /opt/unbound/etc/unbound/unbound.conf /opt/unbound/etc/unbound/unbound.conf.example && \
    apt-get purge -y --auto-remove \
      $build_deps && \
    rm -fr \
      /opt/unbound/share/man \
      /tmp/* \
      /var/tmp/* \
      /var/lib/apt/lists/*

FROM debian:buster
LABEL maintainer="Matthew Vance"

ENV name=unbound \
    unbound_version=1.9.1 \
    version=1.2

ENV summary="${name} is a validating, recursive, and caching DNS resolver." \
    description="${name} is a validating, recursive, and caching DNS resolver."

LABEL summary="${summary}" \
      description="${description}" \
      io.k8s.description="${description}" \
      io.k8s.display-name="Unbound ${unbound_version}" \
      name="mvance/${name}" \
      maintainer="Matthew Vance"

WORKDIR /tmp/src

COPY --from=unbound /opt/ /opt/

RUN set -x && \
    debian_frontend=noninteractive apt-get update && apt-get install -y --no-install-recommends \
      bsdmainutils \
      ldnsutils \
      libevent-2.1-6 \
      libexpat1 && \
    groupadd _unbound && \
    useradd -g _unbound -s /etc -d /dev/null _unbound && \
   rm -fr \
      /tmp/* \
      /var/tmp/* \
      /var/lib/apt/lists/*

COPY a-records.conf /opt/unbound/etc/unbound/
COPY unbound.sh /

RUN chmod +x /unbound.sh

WORKDIR /opt/unbound/

ENV PATH /opt/unbound/sbin:$PATH

EXPOSE 53

HEALTHCHECK --interval=5s --timeout=3s --start-period=5s CMD drill @127.0.0.1 cloudflare.com || exit 1

CMD ["/unbound.sh"]
