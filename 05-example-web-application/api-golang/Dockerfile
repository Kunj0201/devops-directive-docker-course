# Pin specific version for stability
# Use separate stage for building image
# Use debian for easier build utilities
FROM golang:1.19-bullseye AS build-base

WORKDIR /app 

# Copy only files required to install dependencies (better layer caching)
COPY go.mod go.sum ./

# Use cache mount to speed up install of existing dependencies
RUN --mount=type=cache,target=/go/pkg/mod \
  --mount=type=cache,target=/root/.cache/go-build \
  go mod download

FROM build-base AS dev

# Install air for hot reload & delve for debugging
RUN go install github.com/cosmtrek/air@latest && \
  go install github.com/go-delve/delve/cmd/dlv@latest

COPY . .

CMD ["air", "-c", ".air.toml"]

# Use separate stage for production build
# This stage compiles the application and prepares it for deployment
FROM build-base AS build-production

# Add non root user
RUN useradd -u 1001 nonroot

COPY . .

# Compile healthcheck
RUN go build \
  -ldflags="-linkmode external -extldflags -static" \
  -tags netgo \
  -o healthcheck \
  ./healthcheck/healthcheck.go

# Compile application during build rather than at runtime
# Add flags to statically link binary
RUN go build \
  -ldflags="-linkmode external -extldflags -static" \
  -tags netgo \
  -o api-golang

# Use separate stage for deployable image
# Essentially, the top part is an intermediate which is used to build the application
# The final image will only contain the compiled binary and necessary files, reducing size and attack surface
FROM scratch

# Set gin mode
ENV GIN_MODE=release

WORKDIR /

# Copy the passwd file, importing the nonroot user from the build stage
# This is necessary to ensure the nonroot user exists in the final image
COPY --from=build-production /etc/passwd /etc/passwd

# Copy the healthcheck binary from the build stage
COPY --from=build-production /app/healthcheck/healthcheck healthcheck

# Copy the app binary from the build stage
COPY --from=build-production /app/api-golang api-golang

# Use nonroot user
USER nonroot

# Indicate expected port
EXPOSE 8080

CMD ["/api-golang"]
# Dockerfile for Golang API application
# This Dockerfile sets up a Golang environment for the API application