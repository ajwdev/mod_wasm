# Container image build for ease of testing mod_wasm.
#
# This container image provides Apache2, mod_wasm and a default
# configuration that makes easy to execute custom WebAssembly modules.

ARG IMAGE_REPOSITORY=docker.io

# Build wasm_runtime in release mode
FROM $IMAGE_REPOSITORY/library/rust:1.63.0-slim@sha256:2e68e8832b2982e48bb11730abcea8786bbc1b6a893cde0cbff9b14e0bb7f52f as builder-rs
ARG WASM_RUNTIME_PATH=/usr/src/mod_wasm
WORKDIR $WASM_RUNTIME_PATH
COPY ./wasm_runtime ./
RUN cargo build --release

# Build mod_wasm Apache module
FROM $IMAGE_REPOSITORY/library/httpd:2.4@sha256:5d1b0bd4333abe51e4631564bffb7c56f08f449757615c42d9676c807ccea8bd as builder-apache
ARG WASM_RUNTIME_PATH=/usr/src/mod_wasm
ARG DIST_DIR=$WASM_RUNTIME_PATH/dist
RUN apt-get update && apt-get install apache2-dev build-essential pkg-config libtool libapr1-dev libaprutil1-dev -y
WORKDIR /usr/src/mod_wasm
COPY ./mod_wasm $WASM_RUNTIME_PATH
COPY ./dist $DIST_DIR
COPY --from=builder-rs $WASM_RUNTIME_PATH/target/release/libwasm_runtime.so /usr/lib
COPY --from=builder-rs $WASM_RUNTIME_PATH/src/wasm_runtime.h /usr/include/mod_wasm/
RUN mkdir -p $WASM_RUNTIME_PATH/dist/conf $DIST_DIR/modules
RUN ./build.sh

# Distribute wasm_runtime, mod_wasm and a default configuration
FROM $IMAGE_REPOSITORY/library/httpd:2.4@sha256:5d1b0bd4333abe51e4631564bffb7c56f08f449757615c42d9676c807ccea8bd
LABEL org.opencontainers.image.source https://github.com/vmware-labs/mod_wasm
ARG WASM_RUNTIME_PATH=/usr/src/mod_wasm
ARG DIST_DIR=$WASM_RUNTIME_PATH/dist
RUN mkdir -p /usr/local/apache2/wasm_modules/conf
RUN mkdir -p /usr/local/apache2/wasm_modules/python-cgi
RUN mkdir -p /usr/local/apache2/wasm_modules/python-scripts/uploads
COPY ./examples/wasm_modules/mod_wasm_demo.conf /usr/local/apache2/wasm_modules/mod_wasm_demo.conf
COPY ./examples/wasm_modules/python-wasm/ /usr/local/apache2/wasm_modules/python-wasm/
COPY ./examples/wasm_modules/conf/cgi_prettify.conf /usr/local/apache2/wasm_modules/conf/cgi_prettify.conf
COPY ./examples/wasm_modules/python-scripts/cgi_prettify.py /usr/local/apache2/wasm_modules/python-scripts/cgi_prettify.py
COPY ./examples/wasm_modules/python-scripts/uploads /usr/local/apache2/wasm_modules/python-scripts/uploads
COPY --from=builder-rs $WASM_RUNTIME_PATH/target/release/libwasm_runtime.so /usr/lib
COPY --from=builder-apache $DIST_DIR/modules /usr/local/apache2/modules
COPY --from=builder-apache $DIST_DIR/conf/httpd.conf /usr/local/apache2/conf/
