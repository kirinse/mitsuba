####################################################################################################
## Builder
####################################################################################################
FROM rust:1.58-alpine3.15 AS builder

ENV SQLX_OFFLINE="true"

RUN apk add --no-cache \
    ca-certificates \
    musl-dev \
    git \
    g++

WORKDIR /mitsuba

COPY . .

RUN cargo build --target x86_64-unknown-linux-musl --release --all-features

####################################################################################################
## Final image
####################################################################################################
FROM alpine:3.15

RUN apk add --no-cache \
    ca-certificates \
    postgresql-client \
    git \
    tini

WORKDIR /mitsuba

ENV SQLX_OFFLINE="true"

COPY --from=builder /mitsuba/target/x86_64-unknown-linux-musl/release/mitsuba /mitsuba/mitsuba
COPY --from=builder /mitsuba/log4rs.yml /mitsuba/log4rs.yml

# Creat persistent data directory
RUN mkdir -p /data

# Add an unprivileged user and set directory permissions
RUN adduser --disabled-password --gecos "" --no-create-home mitsuba \
    && chown -R mitsuba:mitsuba /mitsuba \
    && chown -R mitsuba:mitsuba /data

ENTRYPOINT ["/sbin/tini", "--"]

USER mitsuba

CMD ["./mitsuba", "start"]

VOLUME /data

EXPOSE 8080
EXPOSE 9000

STOPSIGNAL SIGTERM

# Image metadata
LABEL org.opencontainers.image.title=Mitsuba
LABEL org.opencontainers.image.description="Lightweight 4chan board archive software (like Foolfuuka) written in Rust"
LABEL org.opencontainers.image.vendor="reasv"
LABEL org.opencontainers.image.licenses="AGPL-3.0-or-later"
LABEL org.opencontainers.image.source="https://github.com/reasv/mitsuba"
