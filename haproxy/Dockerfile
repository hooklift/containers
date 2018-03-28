FROM alpine:3.7

ENV HAPROXY_VERSION 1.8.5
ENV LIBRESSL_VERSION 2.7.1
ENV LIBSLZ_VERSION 1.0.0
ENV PCRE_VERSION 8.41

ENV LIBRESSL /tmp/staticssl
ENV PCRE /tmp/staticpcre

WORKDIR /haproxy
RUN apk add --update alpine-sdk linux-headers

# STATIC LIBRESSL
RUN mkdir libressl && cd libressl && \
	curl -s http://ftp.openbsd.org/pub/OpenBSD/LibreSSL/libressl-$LIBRESSL_VERSION.tar.gz | tar xvzf - --strip-components=1
RUN cd libressl && \
	./configure  --prefix=$LIBRESSL --enable-shared=no && \
	make && make install

# STATIC PCRE
RUN mkdir pcre && cd pcre && \
	curl -s http://ftp.cs.stanford.edu/mirrors/exim/pcre/pcre-$PCRE_VERSION.tar.gz | tar xvzf - --strip-components=1
RUN cd pcre && \
	./configure --prefix=$PCRE --enable-shared=no --enable-utf8 --enable-jit && \
	make && make install

# STATIC LIBSLZ
RUN mkdir libslz && cd libslz && \
	curl -s -L https://github.com/haproxy/libslz/archive/v$LIBSLZ_VERSION.tar.gz | tar xvzf - --strip-components=1
RUN cd libslz && \
	make static

# STATIC HAPROXY
# libpcre is compiled statically but haproxy still dynamically links to libc. So,
# to fix it we had to patch the Makefile and remove -Wl, -Bdynamic from the pcre ldflags.
RUN curl -s http://www.haproxy.org/download/1.8/src/haproxy-$HAPROXY_VERSION.tar.gz | tar xvzf - --strip-components=1 && \
	sed -i 's/-Wl,-Bdynamic//' Makefile

RUN make TARGET_LDFLAGS=-static TARGET_CFLAGS=-static TARGET=linux2628 ARCH=x86_64 USE_PCRE_JIT=1 USE_STATIC_PCRE=1 USE_OPENSSL=1 \
	PCRE_LIB=$PCRE/lib PCRE_INC=$PCRE/include  \
	SSL_INC=$LIBRESSL/include SSL_LIB=$LIBRESSL/lib \
	USE_SLZ=1 SLZ_INC=libslz/src SLZ_LIB=libslz \
	USE_LINUX_SPLICE=1 USE_LINUX_TPROXY=1

CMD ["ldd", "/haproxy/haproxy"]
