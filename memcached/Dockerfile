FROM alpine:3.7 as builder

ENV MEMCACHED_VERSION 1.5.11
ENV LIBEVENT_VERSION 2.1.8-stable

RUN apk add --no-cache --update \
	alpine-sdk \
	autoconf \
	automake

WORKDIR /libevent
RUN curl -Ls https://github.com/libevent/libevent/releases/download/release-${LIBEVENT_VERSION}/libevent-${LIBEVENT_VERSION}.tar.gz | tar xvzf - --strip-components=1
RUN ./configure --disable-shared \
	&& make install

WORKDIR /memcached
RUN curl -Ls https://github.com/memcached/memcached/archive/${MEMCACHED_VERSION}.tar.gz | tar xvzf - --strip-components=1
RUN ./autogen.sh \
	&& ./configure \
	&& sed -i 's/LDFLAGS = /LDFLAGS = -static/' Makefile \
	&& make

FROM scratch
COPY --from=builder /memcached/memcached .
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt
COPY --from=builder /etc/passwd /etc/passwd
COPY --from=builder /sbin/nologin /sbin/nologin

ENTRYPOINT ["./memcached", "-u", "nobody"]
