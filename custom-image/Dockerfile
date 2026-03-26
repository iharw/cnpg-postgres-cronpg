FROM ghcr.io/cloudnative-pg/postgresql:17

USER root

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        postgresql-17-cron \
        postgresql-17-pg-stat-kcache \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# CNPG images are expected to run as the postgres user.
USER postgres
