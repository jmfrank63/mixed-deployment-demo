# Stage 1: Build the binary
FROM rust:latest AS builder

# Create app directory
WORKDIR /app

# Copy manifests
COPY server/Cargo.toml ./server/Cargo.toml

RUN mkdir -p server/src && echo "fn main() {}" > server/src/main.rs

# Remove dummy source files
RUN rm -f ./server/src/*.rs

# Copy real source files
COPY . .

# Build the specified workspace
RUN cargo build --release --manifest-path=server/Cargo.toml

# Stage 2: Create a minimal image with OpenSSL 3.0
FROM debian:testing-slim
ARG APP=/usr/src/app

EXPOSE 8443
ENV TZ=Etc/UTC \
    APP_USER=appuser

# Install dependencies and create a user
RUN apt-get update \
    && apt-get install -y ca-certificates tzdata libssl-dev \
    && rm -rf /var/lib/apt/lists/* \
    && useradd $APP_USER \
    && mkdir -p ${APP}

COPY --from=builder /app/target/release/server ${APP}/server
COPY --from=builder /app/server/static ${APP}/static

RUN chown -R $APP_USER:$APP_USER ${APP}

USER $APP_USER
WORKDIR ${APP}

CMD ["./server"]