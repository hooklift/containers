FROM debian:stretch-slim

ENV MCROUTER_VERSION master
# ENV MCROUTER_VERSION v0.38.0-release
ENV FOLLY_VERSION v2018.10.08.00
ENV WANGLE_VERSION v2018.10.08.00
ENV FIZZ_VERSION v2018.10.08.00

ENV LC_ALL=C.UTF-8
ENV LANG=C.UTF-8

RUN apt-get update -y
RUN DEBIAN_FRONTEND=noninteractive \
	apt-get install -y \
	build-essential \
	binutils-dev \
	autoconf \
	automake \
	pkg-config \
	python-dev \
	libdouble-conversion-dev \
	libgflags-dev \
	libjemalloc-dev \
	libgoogle-glog-dev \
	cmake \
	libboost-all-dev \
	libboost-context-dev \
	libevent-dev \
	libssl1.0-dev \
	libtool \
	zlib1g-dev \
	libsnappy-dev \
	liblzma-dev \
	libgtest-dev \
	libsodium-dev \
	libunwind-dev \
	curl \
	git \
	ragel \
	&& rm -rf /var/lib/apt/lists/*

WORKDIR /folly
RUN curl -Ls https://github.com/facebook/folly/archive/${FOLLY_VERSION}.tar.gz | tar xvzf - --strip-components=1
RUN cmake configure . \
	&& make \
	&& make install

WORKDIR /fizz
RUN curl -Ls https://github.com/facebookincubator/fizz/archive/${FIZZ_VERSION}.tar.gz | tar xvzf - --strip-components=1
RUN cmake configure fizz \
	&& make \
	&& make install

WORKDIR /wangle
RUN curl -Ls https://github.com/facebook/wangle/archive/${WANGLE_VERSION}.tar.gz | tar xvzf - --strip-components=1
RUN cmake configure wangle \
	&& make \
	&& make install

WORKDIR /mcrouter
RUN curl -Ls https://github.com/facebook/mcrouter/archive/${MCROUTER_VERSION}.tar.gz | tar xvzf - --strip-components=1
RUN cd mcrouter \
	&& autoreconf --install \
	&& ./configure \
	&& make VERBOSE=1

FROM debian:stretch-slim
ENV LC_ALL=C.UTF-8
ENV LANG=C.UTF-8

RUN apt-get update -y
RUN DEBIAN_FRONTEND=noninteractive \
	apt-get install -y \
	libssl1.0.2 \
	libevent-2.0-5 \
	libgflags2v5 \
	libgoogle-glog0v5 \
	libboost-context1.62.0 \
	libboost-filesystem1.62.0 \
	libboost-program-options1.62.0 \
	libboost-system1.62.0 \
	libboost-regex1.62.0 \
	libboost-thread1.62.0 \
	libdouble-conversion1

COPY --from=0 /mcrouter/mcrouter/mcrouter .

ENTRYPOINT ["./mcrouter"]
