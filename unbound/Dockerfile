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

FROM debian:stretch
LABEL maintainer="Matthew Vance"

ENV name=unbound \
    unbound_version=1.8.3 \
    unbound_sha256=2b692b8311edfad41e7d0380aac34576060d4176add81dc5db419c79b2a4cecc \
    unbound_download_url=https://nlnetlabs.nl/downloads/unbound/unbound-1.8.3.tar.gz \
    version=1

ENV summary="${name} is a validating, recursive, and caching DNS resolver." \
    description="${name} is a validating, recursive, and caching DNS resolver."

LABEL summary="${summary}" \
      description="${description}" \
      io.k8s.description="${description}" \
      io.k8s.display-name="Unbound ${unbound_version}" \
      name="mvance/${name}" \
      maintainer="Matthew Vance"

EXPOSE 53

WORKDIR /tmp/src

COPY --from=openssl /opt/openssl /opt/openssl

RUN build_deps="ca-certificates curl gcc libc-dev libevent-dev libexpat1-dev make" && \
    set -x && \
    debian_frontend=noninteractive apt-get update && apt-get install -y --no-install-recommends \
      $build_deps \
      bsdmainutils \
      ldnsutils \
      libevent-2.0 \
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

COPY a-records.conf /opt/unbound/etc/unbound/
COPY unbound.sh /

RUN chmod +x /unbound.sh

WORKDIR /opt/unbound/

ENV PATH /opt/unbound/sbin:$PATH

HEALTHCHECK --interval=5s --timeout=3s --start-period=5s CMD drill @127.0.0.1 cloudflare.com || exit 1

CMD ["/unbound.sh"]
