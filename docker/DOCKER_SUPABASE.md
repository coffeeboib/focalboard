# Running Focalboard with Supabase in Docker

This guide explains how to run Focalboard in Docker using Supabase as your database.

## Quick Start

### 1. Create Environment File

In the `docker` directory, create a `.env` file:

```bash
cd docker
cp .env.example .env
```

### 2. Edit `.env` with Your Supabase Credentials

Replace `[YOUR-PASSWORD]` with your actual Supabase database password:

```bash
FOCALBOARD_DBTYPE=postgres
FOCALBOARD_DBCONFIG=postgresql://postgres.oziujimcqzqwolsedanj:your-actual-password@aws-1-ap-south-1.pooler.supabase.com:6543/postgres
```

### 3. Run with Docker Compose

```bash
docker-compose up -d
```

Focalboard will be available at `http://localhost` (port 80)

## How It Works

The Docker setup now uses environment variables for database configuration:

1. **`docker/server_config.json`**: Default config uses postgres with empty dbconfig (will be overridden by env vars)
2. **`docker/docker-compose.yml`**: Passes `FOCALBOARD_DBTYPE` and `FOCALBOARD_DBCONFIG` from `.env` file
3. **Docker `.env` file**: Contains your Supabase connection string

## Alternative: Build and Run Manually

If you prefer not to use docker-compose:

```bash
# Build the image
docker build -f docker/Dockerfile -t focalboard:supabase .

# Run with environment variables
docker run -d \
  -p 8000:8000 \
  -v focalboard-data:/opt/focalboard/data \
  -e FOCALBOARD_DBTYPE=postgres \
  -e FOCALBOARD_DBCONFIG="postgresql://postgres.oziujimcqzqwolsedanj:YOUR-PASSWORD@aws-1-ap-south-1.pooler.supabase.com:6543/postgres" \
  --name focalboard \
  focalboard:supabase
```

## Verify It's Working

1. Check the container logs:
   ```bash
   docker logs focalboard
   ```

2. Look for:
   - `connectDatabase dbType=postgres`
   - No database connection errors
   - `Server.Start` message

3. Open `http://localhost` in your browser

4. Check Supabase Dashboard → Table Editor to see the Focalboard tables

## Troubleshooting

### Connection Issues

If you see connection errors:

1. **Check your `.env` file**: Ensure password is correct
2. **SSL Note**: The connection pooler uses port 6543 and doesn't require `sslmode=require` parameter
3. **Network**: Ensure your Docker container can access external networks

### Environment Variables Not Working

If environment variables aren't being picked up:

1. Make sure `.env` file is in the `docker` directory
2. Restart docker-compose: `docker-compose down && docker-compose up -d`
3. Check environment variables are set: `docker exec focalboard env | grep FOCALBOARD`

### Port Conflicts

If port 80 is already in use, edit `docker-compose.yml`:

```yaml
ports:
  - 8000:8000  # Change from 80:8000
```

## Using with Local PostgreSQL

If you want to use a local PostgreSQL container instead of Supabase, see `docker-compose-db-nginx.yml` for an example setup with a local database.

## Data Persistence

Your data is stored in the Docker volume `fbdata`. To backup:

```bash
# List volumes
docker volume ls

# Backup the volume
docker run --rm -v fbdata:/data -v $(pwd):/backup alpine tar czf /backup/focalboard-backup.tar.gz -C /data .
```

## Next Steps

- See [SUPABASE_SETUP.md](../SUPABASE_SETUP.md) for detailed Supabase configuration
- Check [docker/README.md](README.md) for general Docker usage
