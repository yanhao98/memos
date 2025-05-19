# ---- Build Stage ----
ARG TARGETOS
ARG TARGETARCH
# TARGETPLATFORM is a built-in build-arg provided by buildx (e.g., linux/amd64, linux/arm64)
# We can parse TARGETOS and TARGETARCH from it if needed, or use it directly if the Go version supports it.
# For simplicity, we extract GOOS and GOARCH.
ARG GOOS
ARG GOARCH

FROM --platform=${BUILDPLATFORM} golang:1.24-alpine AS backend
# FROM golang:1.24-alpine AS backend

WORKDIR /backend-build

COPY go.mod go.sum ./
RUN go mod download

COPY . .

# Please build frontend first, so that the static files are available.
# Refer to `pnpm release` in package.json for the build command.
# buildx will set TARGETOS and TARGETARCH automatically
RUN --mount=type=cache,target=/go/pkg/mod \
    --mount=type=cache,target=/root/.cache/go-build \
    GOOS=${TARGETOS} GOARCH=${TARGETARCH} CGO_ENABLED=0 go build -ldflags="-s -w" -o memos ./bin/memos/main.go

# ---- Runtime Stage ----
# Make workspace with above generated files.
FROM alpine:latest AS monolithic
ARG TARGETARCH # To select the correct base image if needed, though alpine is multi-arch

WORKDIR /usr/local/memos

RUN apk add --no-cache tzdata
ENV TZ="UTC"

COPY --from=backend /backend-build/memos /usr/local/memos/
COPY ./scripts/entrypoint.sh /usr/local/memos/

EXPOSE 5230

# Directory to store the data, which can be referenced as the mounting point.
RUN mkdir -p /var/opt/memos
VOLUME /var/opt/memos

ENV MEMOS_MODE="prod"
ENV MEMOS_PORT="5230"

ENTRYPOINT ["./entrypoint.sh", "./memos"]
