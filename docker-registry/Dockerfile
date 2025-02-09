# image builder base on golang:1.19.2-alpine3.16
FROM eu.gcr.io/kyma-project/external/golang@sha256:45d14bdb069fd5e1fee50160a3e15752038f0fabf339a5ed7f342b84701ed3d6 as builder

ENV GO111MODULE=auto
ENV DISTRIBUTION_DIR /go/src/github.com/docker/distribution
ENV BUILDTAGS include_oss include_gcs

ARG GOOS=linux
ARG GOARCH=amd64
ARG GOARM=6

RUN set -ex \
    && apk upgrade --no-cache \
    && apk add --no-cache make git file

WORKDIR $DISTRIBUTION_DIR
RUN git clone -b release/2.8 https://github.com/distribution/distribution $DISTRIBUTION_DIR

RUN CGO_ENABLED=0 make PREFIX=/go clean binaries && file ./bin/registry | grep "statically linked"

# result container base on alpine:3.16.2
FROM eu.gcr.io/kyma-project/external/alpine@sha256:1304f174557314a7ed9eddb4eab12fed12cb0cd9809e4c28f29af86979a3c870

RUN set -ex \
    && apk upgrade --no-cache \
    && apk add --no-cache ca-certificates apache2-utils

COPY --from=builder /go/src/github.com/docker/distribution/cmd/registry/config-dev.yml /etc/docker/registry/config.yml
COPY --from=builder /go/src/github.com/docker/distribution/bin/registry /bin/registry

VOLUME ["/var/lib/registry"]
EXPOSE 5000

LABEL source=git@github.com:kyma-project/kyma.git

ENTRYPOINT ["registry"]
CMD ["serve", "/etc/docker/registry/config.yml"]

