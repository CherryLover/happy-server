# Happy Server Deployment

This directory contains Docker Compose configuration for deploying Happy Server with all its dependencies.

## Services

- **app**: Happy Server application (from GitHub Container Registry)
- **postgres**: PostgreSQL 16 database
- **redis**: Redis 7 for pub/sub and caching
- **minio**: MinIO S3-compatible object storage
- **minio-init**: One-time MinIO bucket initialization

## Directory Structure

```
deploy/
├── docker-compose.yml    # Main configuration
├── .env                 # Environment variables (create from .env.example)
├── .env.example         # Environment template
└── data/               # Persistent data (auto-created)
    ├── postgres/       # PostgreSQL data
    ├── redis/          # Redis data
    └── minio/          # MinIO/S3 data
```

## Quick Start

### 1. Configure Environment

**IMPORTANT**: You must create a `.env` file from the template:

```bash
cd deploy
cp .env.example .env
```

**Required changes in `.env`**:
- Set `HANDY_MASTER_SECRET` to a secure random value (use `openssl rand -hex 32`)
- For production: Change `POSTGRES_PASSWORD` and `MINIO_ROOT_PASSWORD`

### 2. Start Services

```bash
docker-compose up -d
```

Data will be stored in `./data/` directory (created automatically).

### 3. Verify Services

```bash
# Check all services are running
docker-compose ps

# View logs
docker-compose logs -f app
```

### 4. Access Services

All ports are configurable via `.env` file. Default values:

- **Happy Server API**: http://localhost:3000
- **Metrics**: http://localhost:9090
- **MinIO Console**: http://localhost:9001 (credentials from `.env`)
- **PostgreSQL**: localhost:5432 (credentials from `.env`)
- **Redis**: localhost:6379

## Database Migrations

Run migrations after first deployment:

```bash
docker-compose exec app yarn migrate
```

## Updating

Pull the latest image and restart:

```bash
docker-compose pull app
docker-compose up -d app
```

## Monitoring

View real-time logs:

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f app
```

## Stopping

```bash
# Stop all services
docker-compose down

# Stop and remove all data (WARNING: destroys all data in ./data/)
docker-compose down
rm -rf ./data
```

## Data Management

All persistent data is stored in `./data/` directory:

```bash
# Backup all data
tar -czf happy-backup-$(date +%Y%m%d).tar.gz data/

# Restore from backup
tar -xzf happy-backup-YYYYMMDD.tar.gz

# View disk usage
du -sh data/*
```

**IMPORTANT**: Add `data/` to `.gitignore` to avoid committing sensitive data.

## Production Considerations

1. **Security**:
   - Generate secure secrets: `HANDY_MASTER_SECRET=$(openssl rand -hex 32)`
   - Change all default passwords in `.env`
   - Restrict file permissions: `chmod 600 .env`
   - Enable SSL/TLS for all services
   - Use secrets management (e.g., Docker secrets, Vault)

2. **Persistence**:
   - Regular backups of `./data/` directory
   - Use external managed databases for production
   - Consider using Docker volumes for better isolation

3. **Scaling**:
   - Use separate Redis for production load
   - Consider PostgreSQL connection pooling
   - Use external S3 instead of MinIO

4. **Monitoring**:
   - Set up Prometheus to scrape metrics from port 9090
   - Configure health check endpoints
   - Set up log aggregation
   - Monitor disk usage of `./data/` directory

## Troubleshooting

### App fails to start

Check logs:
```bash
docker-compose logs app
```

Common issues:
- Database not ready: Wait for postgres health check
- Missing migrations: Run `docker-compose exec app yarn migrate`

### Database connection errors

Verify PostgreSQL is healthy:
```bash
docker-compose ps postgres
docker-compose exec postgres pg_isready -U postgres
```

### MinIO access errors

Check MinIO is running and bucket is created:
```bash
docker-compose ps minio minio-init
docker-compose logs minio-init
```

## Environment Variables

See `.env.example` for all available configuration options.

Required:
- `HANDY_MASTER_SECRET`: Secret key for authentication

Optional:
- `METRICS_ENABLED`: Enable Prometheus metrics (default: true)
