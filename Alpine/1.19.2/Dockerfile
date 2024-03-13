ARG IMAGE_BUILD_DATE="2024-03-13" \
  ALPINE_VERSION="3.19.1" \
  OPENSSL_VERSION="3.2.1" \
  LIBEVENT_VERSION="2.1.12" \
  UNBOUND_VERSION="1.19.2" \
  IMAGE_REVISION="1"


FROM alpine:${ALPINE_VERSION} AS openbuildenv

ARG OPENSSL_VERSION

ENV OPENSSL_VERSION=${OPENSSL_VERSION} \
  OPENSSL_SHA256="83c7329fe52c850677d75e5d0b0ca245309b97e8ecbcfdc1dfdc4ab9fac35b39" \
  OPENSSL_DOWNLOAD_URL="https://www.openssl.org/source/openssl" \
  OPENSSL_SIGNER_RSA="EFC0A467D613CB83C7ED6D30D894E2CE8B3D79F5" \
  OPENSSL_PGP="8657ABB260F056B1E5190839D9C4D26D0E604491"

WORKDIR /tmp/src

RUN set -xe; \
  apk --update --no-cache add \
  ca-certificates \
  gnupg \
  curl \
  file && \
  apk --update --no-cache add --virtual .build-deps \
    build-base \
    perl \
    libidn2-dev \
    libevent-dev \
    linux-headers \
    apk-tools && \
    curl -sSL "${OPENSSL_DOWNLOAD_URL}"-"${OPENSSL_VERSION}".tar.gz -o openssl.tar.gz && \
    echo "${OPENSSL_SHA256} ./openssl.tar.gz" | sha256sum -c - && \
    curl -L "${OPENSSL_DOWNLOAD_URL}"-"${OPENSSL_VERSION}".tar.gz.asc -o openssl.tar.gz.asc && \
    GNUPGHOME="$(mktemp -d)" && \
    export GNUPGHOME && \
    gpg --no-tty --keyserver hkps://keys.openpgp.org --recv-keys "${OPENSSL_PGP}" && \
    gpg --recv-keys "${OPENSSL_SIGNER_RSA}" && \
    gpg --batch --verify openssl.tar.gz.asc openssl.tar.gz && \
    tar xzf openssl.tar.gz && \
    cd openssl-"${OPENSSL_VERSION}" && \
    ./Configure \
      no-weak-ssl-ciphers \
      no-ssl2 \
      no-ssl3 \
      no-err \
      no-tests \
      shared \
  # enable-quic \
      -DOPENSSL_NO_HEARTBEATS \
      -fstack-protector-strong \
      --prefix=/usr/local/openssl \
      --openssldir=/usr/local/openssl \
      --libdir=/usr/local/openssl/lib && \
  make && \
  make install_sw && \
  apk del --no-cache .build-deps && \
#  pkill -9 gpg-agent && \
#  pkill -9 dirmngr && \
  rm -rf \
    /usr/share/man \
    /usr/share/docs \
    /tmp/* \
    /var/tmp/* \
    /var/log/* 
   
   FROM openbuildenv AS libeventbuildenv

ARG LIBEVENT_VERSION

ENV LIBEVENT_VERSION=${LIBEVENT_VERSION} \
  LIBEVENT_DOWNLOAD_URL="https://github.com/libevent/libevent/releases/download/release" \
  LIBEVENT_PGP="9E3AC83A27974B84D1B3401DB86086848EF8686D"
  
RUN set -xe; \
  apk --update --no-cache add \
  ca-certificates \
  gnupg \
  curl \
  file && \
  apk --update --no-cache add --virtual .build-deps \
    build-base \
    python3 \
    zlib-dev \
    linux-headers \
    apk-tools && \
    curl -sSL "${LIBEVENT_DOWNLOAD_URL}"-"${LIBEVENT_VERSION}"-stable/libevent-"${LIBEVENT_VERSION}"-stable.tar.gz -o libevent.tar.gz && \
    curl -sSL "${LIBEVENT_DOWNLOAD_URL}"-"${LIBEVENT_VERSION}"-stable/libevent-"${LIBEVENT_VERSION}"-stable.tar.gz.asc -o libevent.tar.gz.asc && \
    GNUPGHOME="$(mktemp -d)" && \
    export GNUPGHOME && \
    gpg --no-tty --keyserver hkps://pgp.mit.edu --recv-keys "${LIBEVENT_PGP}" || \
    gpg --no-tty --keyserver hkps://keyserver.ubuntu.com --recv-keys "${LIBEVENT_PGP}" && \
    gpg --batch --verify libevent.tar.gz.asc libevent.tar.gz && \
    tar xzf libevent.tar.gz && \
    rm libevent.tar.gz && \
    cd libevent-"${LIBEVENT_VERSION}"-stable && \
    env CPPFLAGS='-I/usr/local/openssl/include' \
      LDFLAGS='-L/usr/local/openssl/lib' \
    ./configure \
      --prefix=/usr/local/libevent \
      --disable-static \
      --disable-libevent-regress \
      --disable-samples \
      --disable-debug-mode \
      --enable-gcc-hardening && \
    make && \
    make install && \
    apk del --no-cache .build-deps && \
    ##pkill -9 gpg-agent && \
   # pkill -9 dirmngr && \
    rm -rf \
      /usr/share/man \
      /usr/share/docs \
      /tmp/* \
      /var/tmp/* \
      /var/log/* 

FROM libeventbuildenv AS unbound

ARG UNBOUND_VERSION

ENV UNBOUND_VERSION=${UNBOUND_VERSION} \
  UNBOUND_DOWNLOAD_URL="https://www.nlnetlabs.nl/downloads/unbound/unbound-"${UNBOUND_VERSION}".tar.gz" \
  UNBOUND_PGP="EDFAA3F2CA4E6EB05681AF8E9F6F1C2D7E045F8D" \
  UNBOUND_SHA256="cc560d345734226c1b39e71a769797e7fdde2265cbb77ebce542704bba489e55" \
  INTERNIC_PGP="F0CB1A326BDF3F3EFA3A01FA937BB869E3A238C5"

WORKDIR /tmp/src

RUN set -xe; \
  apk --update --no-cache add \
  ca-certificates \
  gnupg \
  curl \
  file \ 
  binutils && \
  apk --update --no-cache add --virtual .build-deps \
    build-base \
    libsodium-dev \
    linux-headers \
    nghttp2-libs \
    expat-dev \
    protobuf-c-dev \
    apk-tools && \
  curl -sSL "$UNBOUND_DOWNLOAD_URL" -o unbound.tar.gz && \
  curl -sSL "${UNBOUND_DOWNLOAD_URL}".asc -o unbound.tar.gz.asc && \
  echo "${UNBOUND_SHA256} *unbound.tar.gz" | sha256sum -c - && \
  GNUPGHOME="$(mktemp -d)" && \
  export GNUPGHOME && \
  gpg --no-tty --keyserver hkps://keys.openpgp.org --recv-keys "$UNBOUND_PGP" && \
  gpg --batch --verify unbound.tar.gz.asc unbound.tar.gz && \
  tar -xzf unbound.tar.gz && \
  rm unbound.tar.gz && \
  cd unbound-${UNBOUND_VERSION} && \
  addgroup -S _unbound -g 1000 && \
  adduser -S -D -H -h /usr/local/unbound -u 1000 -s /sbin/nologin -G _unbound _unbound && \
  ./configure \
    --prefix=/usr/local/unbound/unbound.d \
    --with-conf-file=/usr/local/unbound/unbound.conf \
    --sysconfdir=/usr/local/unbound/unbound.d \
    --libdir=/usr/local/unbound/unbound.d/lib \
    --mandir=/usr/share/man \
    --libexecdir=/usr/local/unbound/unbound.d/lib \
    --localstatedir=/usr/local/unbound/unbound.d \ 
    --with-chroot-dir=/usr/local/unbound \
    --with-pidfile=/usr/local/unbound/unbound.d/unbound.pid \ 
    --with-run-dir=/usr/local/unbound/unbound.d \ 
    --with-rootkey-file=/usr/local/unbound/iana.d/root.key \
    --with-libevent=/usr/local/libevent \
    --with-ssl=/usr/local/openssl \
    --with-username=_unbound \
    --enable-dnstap \
    --with-pthreads \
    --without-pythonmodule \
    --without-pyunbound \
    --enable-event-api \
    --enable-dnscrypt \
    --enable-tfo-server \
    --enable-tfo-client \
    --enable-pie \
    --enable-relro-now && \
  make && \
  make install && \
  apk del --no-cache .build-deps && \
  mkdir -p \
    "/usr/local/unbound/iana.d/" && \
  curl -sSL https://www.internic.net/domain/named.cache -o /usr/local/unbound/iana.d/root.hints && \
  curl -sSL https://www.internic.net/domain/named.cache.md5 -o /usr/local/unbound/iana.d/root.hints.md5 && \
  curl -sSL https://www.internic.net/domain/named.cache.sig -o /usr/local/unbound/iana.d/root.hints.sig && \
  ROOT_HINTS_MD5=`cat /usr/local/unbound/iana.d/root.hints.md5` && \
  echo "${ROOT_HINTS_MD5} */usr/local/unbound/iana.d/root.hints" | md5sum -c - && \
  curl -sSL https://www.internic.net/domain/root.zone -o /usr/local/unbound/iana.d/root.zone && \
  curl -sSL https://www.internic.net/domain/root.zone.md5 -o /usr/local/unbound/iana.d/root.zone.md5 && \
  curl -sSL https://www.internic.net/domain/root.zone.sig -o /usr/local/unbound/iana.d/root.zone.sig && \
  ROOT_ZONE_MD5=`cat /usr/local/unbound/iana.d/root.zone.md5` && \
  echo "${ROOT_ZONE_MD5} */usr/local/unbound/iana.d/root.zone" | md5sum -c - && \   
  GNUPGHOME="$(mktemp -d)" && \
  export GNUPGHOME && \
  gpg --no-tty --keyserver hkps://keys.openpgp.org --recv-keys "$INTERNIC_PGP" && \
  gpg --verify /usr/local/unbound/iana.d/root.hints.sig /usr/local/unbound/iana.d/root.hints && \
  gpg --verify /usr/local/unbound/iana.d/root.zone.sig /usr/local/unbound/iana.d/root.zone && \
  /usr/local/unbound/sbin/unbound-anchor -v -a /usr/local/unbound/iana.d/root.key || true && \
 # pkill -9 gpg-agent && \
  #pkill -9 dirmngr && \
  rm -rf \
    /usr/share/man \
    /usr/share/docs \
    /tmp/* \
    /var/tmp/* \
    /var/log/* \
    /usr/local/libevent/lib/*.la \
    /usr/local/libevent/bin \
    /usr/local/libevent/include \
    /usr/local/libevent/lib/pkgconfig \
    /usr/local/openssl/misc \
    /usr/local/openssl/private \
    /usr/local/openssl/include \
    /usr/local/openssl/lib/pkgconfig \
    /usr/local/openssl/lib/*.a \
    /usr/local/unbound/unbound.d/share \
    /usr/local/unbound/etc \
    /usr/local/unbound/unbound.conf \
    /usr/local/unbound/iana.d/root.hints.* \
    /usr/local/unbound/iana.d/root.zone.* \
    /usr/local/unbound/unbound.d/lib/pkgconfig \
    /usr/local/unbound/unbound.d/include \
    /usr/local/unbound/unbound.d/lib/*.a \
    /usr/local/unbound/unbound.d/lib/*.la && \
    find /usr/local/libevent/lib/lib* -type f | xargs strip --strip-all  && \
    strip --strip-all /usr/local/openssl/lib/ossl-modules/legacy.so && \
    strip --strip-all /usr/local/openssl/bin/openssl && \
    find /usr/local/openssl/lib/lib* -type f | xargs strip --strip-all && \
    find /usr/local/openssl/lib/engines-3/*.so -type f | xargs strip --strip-all  && \
    find /usr/local/unbound/unbound.d/lib/lib* -type f | xargs strip --strip-all && \
     strip --strip-all /usr/local/unbound/unbound.d/sbin/unbound && \
   strip --strip-all /usr/local/unbound/unbound.d/sbin/unbound-anchor && \
    strip --strip-all /usr/local/unbound/unbound.d/sbin/unbound-checkconf  && \
    strip --strip-all /usr/local/unbound/unbound.d/sbin/unbound-control && \
    strip --strip-all /usr/local/unbound/unbound.d/sbin/unbound-host
	
FROM alpine:latest

ARG UNBOUND_VERSION \
  ALPINE_VERSION \
  OPENSSL_VERSION \
  LIBEVENT_VERSION \
  IMAGE_BUILD_DATE \
  IMAGE_REVISION

ENV BUILD_DATE=${IMAGE_BUILD_DATE} \
  IMAGE_REVISION=${IMAGE_REVISION} \
  UNBOUND_VERSION=${UNBOUND_VERSION} \
  ALPINE_VERSION=${ALPINE_VERSION} \
  OPENSSL_VERSION=${OPENSSL_VERSION} \
  LIBEVENT_VERSION=${LIBEVENT_VERSION}
  	
RUN set -xe; \
  addgroup -S _unbound -g 1000 && \
  adduser -S -D -H -h /etc/unbound -u 1000 -s /sbin/nologin -G _unbound _unbound && \
  apk --update --no-cache add \
    ca-certificates \
    tzdata \
    libsodium \
    nghttp2 \
    protobuf-c \
    expat
	
COPY root/usr/local/unbound/ \
  /usr/local/unbound/

COPY root/*.sh \
    /usr/local/sbin/

COPY --from=unbound /usr/local/openssl/ \
  /usr/local/openssl/
  
COPY --from=unbound /usr/local/libevent/ \
  /usr/local/libevent/
  
COPY --from=unbound /usr/local/unbound/ \
  /usr/local/unbound/

RUN mkdir -p \
  "/usr/local/unbound/conf.d/" \
    "/usr/local/unbound/certs.d/" \
      "/usr/local/unbound/zones.d/" \
        "/usr/local/unbound/log.d/" && \
  touch /usr/local/unbound/log.d/unbound.log && \
  chown -R _unbound:_unbound \
    /usr/local/unbound/ && \
  ln -s /dev/random /dev/urandom /dev/null \
    /usr/local/unbound/unbound.d/ && \
  chown -Rh _unbound:_unbound \
    /usr/local/unbound/unbound.d/random \
      /usr/local/unbound/unbound.d/null \
        /usr/local/unbound/unbound.d/urandom && \
  chmod -R 770 \
    /usr/local/sbin/*.sh && \
  rm -rf \
    /usr/share/man \
    /usr/share/docs \
    /tmp/* \
    /var/tmp/* \
    /var/log/* 
	
ENV PATH=/usr/local/unbound/unbound.d/sbin:"$PATH"

EXPOSE 53/tcp
EXPOSE 53/udp

HEALTHCHECK --interval=1m CMD healthcheck.sh

CMD [ "/usr/local/sbin/unbound.sh" ]
