# Build the Go Binary
FROM golang:1.18 as build_sales-api
ENV CGO_ENABLED 0
ARG BUILD_REF

# Create the service directory and the copy the module files first and then
# download the dependencies. If this doesn't change, we won't need to do this
# again in future builds.
# RUN mkdir /fourth
# COPY go.* /fourth/
# WORKDIR /fourth
# RUN go mod download

# Copy the source code into the container.
COPY . /fourth

# Build the admin binary.
WORKDIR /fourth/app/tooling/sales-admin
RUN go build -ldflags "-X main.build=${BUILD_REF}"

# Build the service binary.
WORKDIR /fourth/app/services/sales-api
RUN go build -ldflags "-X main.build=${BUILD_REF}"


# Run the Go Binary in Alpine.
FROM alpine:3.17
ARG BUILD_DATE
ARG BUILD_REF
RUN addgroup -g 1000 -S sales && \
    adduser -u 1000 -h /fourth -G sales -S sales
COPY --from=build_sales-api --chown=sales:sales /fourth/zarf/keys/. /fourth/zarf/keys/.
COPY --from=build_sales-api --chown=sales:sales /fourth/app/tooling/sales-admin/sales-admin /fourth/sales-admin
COPY --from=build_sales-api --chown=sales:sales /fourth/app/services/sales-api/sales-api /fourth/sales-api
WORKDIR /fourth
USER sales
CMD ["./sales-api"]

LABEL org.opencontainers.image.created="${BUILD_DATE}" \
	  org.opencontainers.image.title="sales-api" \
	  org.opencontainers.image.authors="Dmitry Ovchinnikov <dmitry.v.ovchinnikov@gmail.com>" \
	  org.opencontainers.image.source="https://github.com/dmitryovchinnikov/third/app/services/sales-api" \
	  org.opencontainers.image.revision="${BUILD_REF}" \
	  org.opencontainers.image.vendor="Dmitry Ovchinnikov"
