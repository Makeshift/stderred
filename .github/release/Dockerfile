# syntax=docker/dockerfile:1
# Builds and tests libstderred.so for one platform; the release workflow runs it once per
# platform and libc with buildx and exports the library with --output type=local.
ARG BASE=ubuntu:20.04

FROM ${BASE} AS build
ARG LIBC=gnu
ARG DEBIAN_FRONTEND=noninteractive
# src/CMakeLists.txt needs a newer CMake than older distros ship, so glibc builds take it from
# PyPI. Building on an old release keeps the glibc symbol versions the library needs low.
RUN if [ "$LIBC" = musl ]; then \
      apk add --no-cache cmake make gcc musl-dev; \
    else \
      apt-get -o Acquire::Retries=5 update \
      && apt-get -o Acquire::Retries=5 install -y --no-install-recommends gcc libc6-dev make python3 python3-venv ca-certificates \
      && python3 -m venv /opt/cmake \
      && /opt/cmake/bin/pip install --no-cache-dir --upgrade pip \
      && /opt/cmake/bin/pip install --no-cache-dir cmake \
      && ln -s /opt/cmake/bin/cmake /opt/cmake/bin/ctest /usr/local/bin/; \
    fi
COPY src /src
# No CMAKE_BUILD_TYPE, as upstream: at -O2 glibc's headers inline putc_unlocked and friends in
# the test program, so the tests stop exercising the interposed functions.
RUN cmake -S /src -B /build \
    && cmake --build /build \
    && cd /build && ctest --output-on-failure

FROM scratch
COPY --from=build /build/libstderred.so /lib/libstderred.so
