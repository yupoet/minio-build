# 自建 MinIO 社区版镜像（上游 2026-04-25 归档、dl.min.io 全线 410 后的唯一分发途径）
# 构建上下文必须是仓库根（源码在根），见 build/README.md。
FROM golang:1.24-alpine AS build
ENV CGO_ENABLED=0 GOPROXY=https://goproxy.cn,direct
WORKDIR /src
COPY . .
ARG MINIO_VERSION=RELEASE.2025-10-15T17-29-55Z
RUN go build -tags kqueue -trimpath \
    -ldflags "-s -w -X github.com/minio/minio/cmd.Version=${MINIO_VERSION} -X github.com/minio/minio/cmd.CopyrightYear=2025 -X github.com/minio/minio/cmd.ReleaseTag=${MINIO_VERSION}" \
    -o /out/minio .

FROM alpine:3.20
RUN apk add --no-cache ca-certificates
COPY --from=build /out/minio /usr/bin/minio
COPY dockerscripts/docker-entrypoint.sh /usr/bin/docker-entrypoint.sh
EXPOSE 9000 9001
VOLUME ["/data"]
ENTRYPOINT ["/usr/bin/docker-entrypoint.sh"]
CMD ["minio"]
