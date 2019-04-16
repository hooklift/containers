FROM alpine:3.7 as builder

ENV HAPROXY_BASELINE 1.9
ENV HAPROXY_VERSION ${HAPROXY_BASELINE}.6
ENV LIBRESSL_VERSION 2.6.4
ENV OPENSSL_VERSION 1.1.1b
ENV LIBSLZ_VERSION 1.1.0
ENV PCRE_VERSION 10.32

ENV LIBRESSL /tmp/staticssl
ENV PCRE /tmp/staticpcre

RUN apk add --no-cache --update alpine-sdk linux-headers perl

# STATIC LIBRESSL
# WORKDIR /libressl
# RUN curl -s http://ftp.openbsd.org/pub/OpenBSD/LibreSSL/libressl-$LIBRESSL_VERSION.tar.gz | tar xvzf - --strip-components=1
# RUN ./configure  --prefix=$LIBRESSL --enable-shared=no \
# 	&& make install

WORKDIR /openssl
RUN curl -s https://www.openssl.org/source/openssl-${OPENSSL_VERSION}.tar.gz | tar xvzf - --strip-components=1
RUN ./config --prefix=${LIBRESSL} no-shared no-async \
	&& make install_sw

# STATIC PCRE
WORKDIR /pcre
RUN curl -s https://ftp.pcre.org/pub/pcre/pcre2-${PCRE_VERSION}.tar.gz | tar xvzf - --strip-components=1
RUN ./configure --prefix=${PCRE} --enable-shared=no --enable-utf8 --enable-jit \
	&& make install

# STATIC LIBSLZ
WORKDIR /libslz
RUN curl -s -L "http://git.1wt.eu/web?p=libslz.git;a=snapshot;h=v${LIBSLZ_VERSION};sf=tgz" | tar xvzf - --strip-components=1
RUN make static \
	&& make install

# STATIC HAPROXY
# libpcre is compiled statically but haproxy still dynamically links to libc. So,
# to fix it we had to patch the Makefile and remove -Wl, -Bdynamic from the pcre ldflags.
WORKDIR /haproxy
RUN curl -s http://www.haproxy.org/download/${HAPROXY_BASELINE}/src/haproxy-${HAPROXY_VERSION}.tar.gz | tar xvzf - --strip-components=1 \
	&& sed -i 's/-Wl,-Bdynamic//' Makefile

# TODO(c4milo): USE_LUA=1
RUN make TARGET_LDFLAGS=-static TARGET_CFLAGS=-static TARGET=linux2628 ARCH=x86_64 USE_PCRE2_JIT=1 USE_STATIC_PCRE2=1 USE_OPENSSL=1 \
	PCRE_LIB=${PCRE}/lib PCRE_INC=${PCRE}/include PCRE2DIR=${PCRE} \
	SSL_INC=${LIBRESSL}/include SSL_LIB=${LIBRESSL}/lib \
	USE_SLZ=1 SLZ_INC=libslz/src SLZ_LIB=libslz \
	USE_LINUX_SPLICE=1 USE_LINUX_TPROXY=1 \
	USE_LINUX_TPROXY=1 USE_REGPARM=1 \
	USE_THREAD=1 USE_TFO=1

FROM scratch
COPY --from=builder /haproxy/haproxy .
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt
COPY --from=builder /etc/passwd /etc/passwd
COPY --from=builder /sbin/nologin /sbin/nologin
ENTRYPOINT ["./haproxy"]
