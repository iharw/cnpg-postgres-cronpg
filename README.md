# CNPG custom PostgreSQL image

This folder contains a simple GitHub-ready setup for building and using a custom CloudNativePG PostgreSQL image with:

- `pg_cron`
- `pg_stat_kcache`

The base image already includes common extensions such as `pgaudit`, `pg_stat_statements`, and `auto_explain`, so this image only adds what is missing.

## Layout

```text
cnpg/
├── custom-image/
│   ├── .dockerignore
│   └── Dockerfile
└── examples/
    ├── app-user-secret.yaml
    └── cluster-custom-image.yaml
```

## Build locally

```bash
cd custom-image
docker build -t docker.io/harwops/cnpg-postgres-cronpg:17 .
docker push docker.io/harwops/cnpg-postgres-cronpg:17
```

Optional verification:

```bash
docker run --rm docker.io/harwops/cnpg-postgres-cronpg:17 \
  ls /usr/lib/postgresql/17/lib/ | grep -E 'pg_cron|pg_stat_kcache'
```

## Docker Hub image

The image has already been pushed to Docker Hub as:

```text
docker.io/harwops/cnpg-postgres-cronpg:17
```

Push command used:

```bash
sudo docker push harwops/cnpg-postgres-cronpg:17
```

## Use in a CNPG cluster

Update the image reference in [cluster-custom-image.yaml](examples/cluster-custom-image.yaml) and apply your secrets first.

```bash
kubectl apply -f examples/app-user-secret.yaml
kubectl apply -f examples/cluster-custom-image.yaml
```

The sample manifest sets:

- `imageName` to `docker.io/harwops/cnpg-postgres-cronpg:17`
- `shared_preload_libraries` to `pg_cron,pg_stat_kcache`
- `cron.database_name` to `app`

## Enable the extensions

After the cluster is ready, create the extensions in the target database:

```bash
kubectl cnpg psql pg-cluster -- dbname=app
```

```sql
CREATE EXTENSION IF NOT EXISTS pg_cron;
CREATE EXTENSION IF NOT EXISTS pg_stat_kcache;
```

Example cron job:

```sql
SELECT cron.schedule('nightly-vacuum', '0 2 * * *', 'VACUUM');
```

## Notes

- Keep `USER postgres` in the Dockerfile because CNPG expects that runtime user.
- `shared_preload_libraries` should only include libraries you manage yourself. CNPG will handle its own managed entries separately.
- If you keep the Docker Hub repository private, create and reference `imagePullSecrets` in the cluster manifest.
