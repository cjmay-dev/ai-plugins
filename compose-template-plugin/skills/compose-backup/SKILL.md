# Configure Backups with stack-back

Configure and manage automated backups using stack-back. This skill helps set up backup schedules, configure backup targets, and restore from backups.

## What This Skill Does

This skill guides backup configuration and management:

1. Understanding stack-back backup system
2. Adding stack-back to your compose setup
3. Configuring backup schedules and retention
4. Customizing backup targets (volumes and databases)
5. Monitoring backup status
6. Restoring from backups

## Usage

Invoke this skill when:
- Setting up backups for a new application
- Customizing backup configuration
- Troubleshooting backup issues
- Need to restore from backup

## stack-back Overview

stack-back is an automated incremental backup solution using [restic](https://restic.net/) for docker-compose setups.

**Features:**
- Automatic volume backups (docker volumes and bind mounts)
- Automatic database backups (PostgreSQL, MySQL, MariaDB)
- Scheduled backups via cron
- Incremental backups with restic
- Configurable retention policies
- Notifications via SMTP or Discord webhooks
- Simple restore with restic commands

**How It Works:**
- Runs as a Docker container in your compose stack
- Monitors Docker socket to detect volumes and databases
- Uses restic for efficient incremental backups
- Backs up to any restic-supported backend (B2, S3, local, etc.)

## Adding stack-back to Your Compose Setup

### Basic Setup

Add the stack-back service to your `docker-compose.yaml`:

```yaml
services:
  backup:
    image: ghcr.io/lawndoc/stack-back:latest
    env_file:
      - stack-back.env
    environment:
      - AUTO_BACKUP_ALL=true
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock:ro
      - backup_cache:/cache  # Persistent restic cache

  # Your application services
  web:
    image: nginx:alpine
    volumes:
      - web_data:/usr/share/nginx/html

  db:
    image: postgres:15
    environment:
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  backup_cache:
  web_data:
  db_data:
```

### Configuration File

Create a `stack-back.env` file in your project root:

```bash
# Required: Repository location
RESTIC_REPOSITORY=s3:s3.us-east-1.amazonaws.com/my-backup-bucket
# or for Backblaze B2:
# RESTIC_REPOSITORY=b2:bucket-name:path

# Required: Encryption password (don't lose this!)
RESTIC_PASSWORD=your-secure-password-here

# Retention policy (optional, defaults shown)
RESTIC_KEEP_DAILY=7
RESTIC_KEEP_WEEKLY=4
RESTIC_KEEP_MONTHLY=12
RESTIC_KEEP_YEARLY=3

# Schedule (optional, default is daily at 2 AM)
CRON_SCHEDULE="0 2 * * *"

# Backup everything automatically
AUTO_BACKUP_ALL=true
```

### For Backblaze B2

```bash
RESTIC_REPOSITORY=b2:my-backup-bucket:compose-app
RESTIC_PASSWORD=your-restic-encryption-password
B2_ACCOUNT_ID=your-b2-key-id
B2_ACCOUNT_KEY=your-b2-application-key
RESTIC_KEEP_DAILY=7
RESTIC_KEEP_WEEKLY=4
RESTIC_KEEP_MONTHLY=12
CRON_SCHEDULE="0 2 * * *"
AUTO_BACKUP_ALL=true
```

### For AWS S3

```bash
RESTIC_REPOSITORY=s3:s3.us-east-1.amazonaws.com/my-bucket/compose-app
RESTIC_PASSWORD=your-restic-encryption-password
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
RESTIC_KEEP_DAILY=7
CRON_SCHEDULE="0 2 * * *"
AUTO_BACKUP_ALL=true
```

## Customizing Backup Configuration

### Backup Schedule

Change the cron schedule in `stack-back.env`:

```bash
# Every 6 hours
CRON_SCHEDULE="0 */6 * * *"

# Twice daily (2 AM and 2 PM)
CRON_SCHEDULE="0 2,14 * * *"

# Weekly on Sunday at 3 AM
CRON_SCHEDULE="0 3 * * 0"

# Every 4 hours between 8 AM and 8 PM on weekdays
CRON_SCHEDULE="0 8-20/4 * * 1-5"
```

### Retention Policies

Configure how long backups are kept:

```bash
# Keep daily backups for 30 days
RESTIC_KEEP_DAILY=30

# Keep weekly backups for 8 weeks
RESTIC_KEEP_WEEKLY=8

# Keep monthly backups for 24 months
RESTIC_KEEP_MONTHLY=24

# Keep yearly backups for 5 years
RESTIC_KEEP_YEARLY=5
```

### Selective Backups with Compose Labels

By default with `AUTO_BACKUP_ALL=true`, everything is backed up. Use labels to exclude or include specific volumes:

```yaml
services:
  web:
    image: nginx:alpine
    labels:
      # Exclude specific volumes by name
      - stack-back.volumes.exclude: cache
    volumes:
      - web_data:/usr/share/nginx/html      # Backed up
      - cache:/var/cache/nginx              # Excluded

  db:
    image: postgres:15
    labels:
      # Disable database dump backup
      - stack-back.postgres: false
      # Also disable volume backup for this service
      - stack-back.volumes: false
    volumes:
      - db_data:/var/lib/postgresql/data
```

### Include Only Specific Volumes

```yaml
services:
  app:
    image: myapp
    labels:
      - stack-back.volumes: true
      - stack-back.volumes.include: "uploads,config"
    volumes:
      - uploads:/app/uploads      # Backed up
      - config:/app/config        # Backed up
      - cache:/app/cache          # Not backed up
      - temp:/app/temp            # Not backed up
```

### Database Backup Configuration

Enable database backups with labels:

```yaml
services:
  postgres:
    image: postgres:15
    labels:
      - stack-back.postgres: true
    environment:
      - POSTGRES_USER=appuser
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_DB=appdb
    volumes:
      - pgdata:/var/lib/postgresql/data

  mysql:
    image: mysql:8
    labels:
      - stack-back.mysql: true
    environment:
      - MYSQL_USER=appuser
      - MYSQL_PASSWORD=${DB_PASSWORD}
      - MYSQL_ROOT_PASSWORD=${DB_ROOT_PASSWORD}
    volumes:
      - mysqldata:/var/lib/mysql

  mariadb:
    image: mariadb:10
    labels:
      - stack-back.mariadb: true
    environment:
      - MARIADB_USER=appuser
      - MARIADB_PASSWORD=${DB_PASSWORD}
    volumes:
      - mariadbdata:/var/lib/mariadb
```

**Note:** When database backups are enabled, the database data volume is automatically excluded from volume backups to avoid redundancy.

## Monitoring Backups

### Check Backup Status

```bash
# View current configuration and what will be backed up
docker-compose exec backup rcb status

# Example output:
# INFO: Status for compose project 'myproject'
# INFO: Repository: 's3:s3.us-east-1.amazonaws.com/bucket'
# INFO: Backup currently running?: False
# INFO: --------------- Detected Config ---------------
# INFO: service: postgres
# INFO:  - postgres (is_ready=True)
# INFO: service: web
# INFO:  - volume: web_data
```

### View Backup Logs

```bash
# Follow backup logs
docker-compose logs -f backup

# View recent logs
docker-compose logs --tail=100 backup
```

### List Snapshots

```bash
# List all backup snapshots
docker-compose exec backup rcb snapshots

# Or use restic directly
docker-compose exec backup restic snapshots
```

### Check Repository

```bash
# Check repository integrity
docker-compose exec backup restic check

# With cache for faster checking (reduces read operations)
# Set in stack-back.env: CHECK_WITH_CACHE=true
```

## Notifications

### Email Notifications

Add to `stack-back.env`:

```bash
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_HOST_USER=your-email@gmail.com
EMAIL_HOST_PASSWORD=your-app-password
EMAIL_SEND_TO=alerts@example.com
```

### Discord Notifications

Add to `stack-back.env`:

```bash
DISCORD_WEBHOOK=https://discord.com/api/webhooks/your-webhook-url
```

### Test Notifications

```bash
docker-compose exec backup rcb alert
```

## Restoring from Backups

### List Available Snapshots

```bash
# List all snapshots
docker-compose exec backup restic snapshots

# Output shows snapshot IDs and paths:
# ID        Time                 Host        Tags        Paths
# --------------------------------------------------------------
# 4bba301e  2024-02-10 02:00:00  myhost                  /volumes/web/usr/share/nginx/html
# a3c7d42f  2024-02-10 02:00:00  myhost                  /databases/postgres/appdb.sql
```

### Restore a Volume

1. **Stop the application**
   ```bash
   docker-compose down
   ```

2. **Restore using restic**
   ```bash
   # Restore latest snapshot of a volume
   docker-compose run --rm backup restic restore latest \
     --target /restore \
     --path /volumes/web/usr/share/nginx/html

   # Or restore specific snapshot by ID
   docker-compose run --rm backup restic restore 4bba301e \
     --target /restore
   ```

3. **Copy restored data to volume**
   ```bash
   # The restored data is in /restore, copy to volume location
   docker run --rm \
     -v web_data:/data \
     -v /restore:/restore \
     alpine cp -a /restore/. /data/
   ```

4. **Restart application**
   ```bash
   docker-compose up -d
   ```

### Restore a Database

1. **Stop the application**
   ```bash
   docker-compose down
   ```

2. **Restore database dump**
   ```bash
   # List database snapshots
   docker-compose run --rm backup restic snapshots \
     --path /databases

   # Restore latest database dump to file
   docker-compose run --rm backup restic dump latest \
     /databases/postgres/appdb.sql > appdb_restore.sql
   ```

3. **Import to database**
   ```bash
   # Start database only
   docker-compose up -d postgres

   # Import dump
   docker-compose exec -T postgres psql -U appuser appdb < appdb_restore.sql

   # For MySQL
   docker-compose exec -T mysql mysql -u appuser -p appdb < appdb_restore.sql
   ```

4. **Restart application**
   ```bash
   docker-compose up -d
   ```

### Mount Snapshot for Browsing

```bash
# Mount a snapshot to browse files
docker-compose exec backup restic mount /mnt

# In another terminal, browse the mounted snapshot
docker-compose exec backup ls /mnt/snapshots/latest/volumes/
```

## Troubleshooting

### Backups Not Running

**Check container is running:**
```bash
docker-compose ps backup
```

**Check logs:**
```bash
docker-compose logs backup
```

**Verify cron configuration:**
```bash
docker-compose exec backup crontab -l
```

### Backup Failures

**Common issues:**
- Insufficient disk space for cache
- Invalid repository credentials
- Network connectivity issues
- Database connection failures

**Check repository access:**
```bash
# Test repository connectivity
docker-compose exec backup restic snapshots
```

**Check database connectivity:**
```bash
# Verify database is accessible
docker-compose exec backup rcb status
```

### Repository Errors

**Initialize repository if new:**
```bash
docker-compose exec backup restic init
```

**Unlock repository if locked:**
```bash
# If backup was interrupted, repository may be locked
docker-compose exec backup restic unlock
```

**Repair repository:**
```bash
docker-compose exec backup restic rebuild-index
docker-compose exec backup restic check
```

### Large Backup Sizes

**Check cache usage:**
```bash
# Cache speeds up operations but uses disk space
docker volume inspect backup_cache
```

**Prune old snapshots:**
```bash
# Manually trigger maintenance
docker-compose exec backup rcb cleanup
```

**Exclude unnecessary data:**
Use labels to exclude cache directories, logs, and temporary files.

## Advanced Configuration

### Exclude Bind Mounts

```bash
# Only backup docker volumes, not bind mounts
EXCLUDE_BIND_MOUNTS=true
```

### Include Project Name in Paths

```bash
# Useful when backing up multiple projects to same repository
INCLUDE_PROJECT_NAME=true
```

### Backup Multiple Compose Projects

```bash
# Backup all compose projects on the host
INCLUDE_ALL_COMPOSE_PROJECTS=true
```

### Custom Cron Command

```bash
# Run custom commands instead of default backup
CRON_COMMAND="source /env.sh && rcb backup && rcb snapshots > /proc/1/fd/1"
```

### Maintenance Schedule

```bash
# Run maintenance (forget + prune + check) on separate schedule
# Default: maintenance runs after every backup
MAINTENANCE_SCHEDULE="0 3 * * 0"  # Weekly on Sunday at 3 AM
```

### Stop Service During Backup

For services with files at risk of corruption (like SQLite databases):

```yaml
services:
  app:
    image: myapp
    labels:
      - stack-back.volumes: true
      - stack-back.volumes.stop-during-backup: true
    volumes:
      - app_data:/data
```

## Best Practices

1. **Secure Your Encryption Password**
   - Store `RESTIC_PASSWORD` securely (use Infisical or similar)
   - NEVER lose this password - backups are unrecoverable without it

2. **Test Restores Regularly**
   - Verify backups are actually restorable
   - Practice restore procedure in dev environment
   - Document your restore process

3. **Monitor Backup Success**
   - Set up notifications for backup failures
   - Regularly check logs and snapshot lists
   - Verify repository integrity with `restic check`

4. **Optimize Performance**
   - Use persistent cache volume for faster operations
   - Set `CHECK_WITH_CACHE=true` to reduce read operations
   - Consider separate maintenance schedule to limit operations

5. **3-2-1 Backup Strategy**
   - Keep 3 copies of data
   - On 2 different media
   - 1 copy offsite (cloud storage)

6. **Right-Size Retention**
   - Balance storage costs with recovery needs
   - More frequent backups for critical data
   - Longer retention for compliance requirements

## Resources

- [stack-back GitHub Repository](https://github.com/lawndoc/stack-back)
- [stack-back Documentation](https://stack-back.readthedocs.io)
- [restic Documentation](https://restic.readthedocs.io)
- [Backblaze B2 Documentation](https://www.backblaze.com/b2/docs/)

## Next Steps

After configuring backups:
- Test restore procedure in dev environment
- Set up monitoring and alerts
- Document restore process for your team
- Schedule regular backup verification
- Review and adjust retention policies quarterly
