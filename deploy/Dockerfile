# Copyright 2018 Google, Inc. All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM golang:1.17
WORKDIR /src

# This arg is passed by docker buildx & contains the target CPU architecture (e.g., amd64, arm64, etc.)
ARG TARGETARCH

ENV GOARCH=$TARGETARCH
ENV CGO_ENABLED=0
ENV GOBIN=/usr/local/bin

# Add .docker config dir
RUN mkdir -p /kaniko/.docker

COPY . .
RUN \
    --mount=type=cache,target=/root/.cache/go-build \
    --mount=type=cache,target=/go/pkg \
    make GOARCH=$TARGETARCH

# Generate latest ca-certificates

FROM debian:buster-slim AS certs

RUN \
  apt update && \
  apt install -y ca-certificates && \
  cat /etc/ssl/certs/* > /ca-certificates.crt

FROM scratch
COPY --from=0 /src/out/executor /kaniko/executor
COPY --from=certs /ca-certificates.crt /kaniko/ssl/certs/
COPY --from=0 /kaniko/.docker /kaniko/.docker
COPY files/nsswitch.conf /etc/nsswitch.conf
ENV HOME /root
ENV USER root
ENV PATH /usr/local/bin:/kaniko
ENV SSL_CERT_DIR=/kaniko/ssl/certs
ENV DOCKER_CONFIG /kaniko/.docker/
WORKDIR /workspace

ENTRYPOINT ["/kaniko/executor"]
