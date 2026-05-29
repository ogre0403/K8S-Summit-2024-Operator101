# Build the manager binary
FROM golang:1.25.10-alpine AS builder

WORKDIR /workspace
# Copy the Go Modules manifests
COPY 03-OPERATOR/operator/go.mod go.mod
COPY 03-OPERATOR/operator/go.sum go.sum
RUN go mod download
RUN go mod tidy

COPY 03-OPERATOR/operator .

# Build
ARG TARGETARCH
RUN CGO_ENABLED=0 GOOS=linux GOARCH=${TARGETARCH} go build -a -o operator main.go web_reconcier.go simple_reconcier.go

# Use distroless as minimal base image to package the manager binary
# Refer to https://github.com/GoogleContainerTools/distroless for more details
FROM gcr.io/distroless/static:nonroot
WORKDIR /
COPY --from=builder /workspace/operator .
USER 65532:65532

ENTRYPOINT ["/operator"]
