# Configure Backups with stack-back

Configure and manage automated backups using stack-back. This skill helps set up backup schedules, configure backup targets, and restore from backups.

## What This Skill Does

This skill guides backup configuration and management:

1. Understanding stack-back backup system
2. Configuring backup schedules
3. Customizing backup targets (volumes and databases)
4. Restoring from backups
5. Monitoring backup status

## Usage

Invoke this skill when:
- Setting up backups for a new application
- Customizing backup configuration
- Troubleshooting backup issues
- Need to restore from backup

## stack-back Overview

stack-back is the backup solution integrated into compose-template:

**Features:**
- Automatic volume backups
- Automatic database backups (PostgreSQL, MySQL, MariaDB, MongoDB)
- Scheduled backups via cron
- Backup to Backblaze B2
- Configurable retention policies
- Simple restore process

**Default Behavior:**
- Backs up ALL Docker volumes
- Backs up ALL databases automatically
- Uses default schedule (daily at 2 AM)
- Uploads to B2 bucket created by Terraform

## stack-back Configuration

### Configuration File Location
```
/opt/stack-back/config.yaml
```

### Basic Configuration Structure
```yaml
# Backup schedule (cron format)
schedule: "0 2 * * *"  # Daily at 2 AM

# Backup destinations
destinations:
  - type: b2
    bucket: ${B2_BUCKET}
    key_id: ${B2_KEY_ID}
    app_key: ${B2_APP_KEY}

# Volume backups
volumes:
  - name: "*"  # All volumes
    compress: true
    
# Database backups
databases:
  auto_discover: true  # Automatically find and backup databases
```

## Customizing Backup Configuration

### Modify Backup Schedule

To change when backups run:

```yaml
# Every 6 hours
schedule: "0 */6 * * *"

# Twice daily (2 AM and 2 PM)
schedule: "0 2,14 * * *"

# Weekly on Sunday at 3 AM
schedule: "0 3 * * 0"

# Every 4 hours between 8 AM and 8 PM on weekdays
schedule: "0 8-20/4 * * 1-5"
```

### Selective Volume Backups

Instead of backing up all volumes:

```yaml
volumes:
  # Backup specific volumes only
  - name: "app_data"
    compress: true
    retention:
      days: 30
      
  - name: "user_uploads"
    compress: true
    retention:
      days: 90
      
  # Exclude specific volumes
  - name: "temp_cache"
    exclude: true
```

### Database Backup Configuration

Fine-tune database backups:

```yaml
databases:
  # Auto-discover with custom settings
  auto_discover: true
  retention:
    days: 30
    
  # Or specify databases explicitly
  explicit:
    - name: "postgres_app"
      type: "postgresql"
      host: "db"
      port: 5432
      database: "appdb"
      username: "${DB_USER}"
      password: "${DB_PASSWORD}"
      compress: true
      retention:
        days: 90
```

### Retention Policies

Configure how long backups are kept:

```yaml
# Global retention
retention:
  days: 30      # Keep for 30 days
  weeks: 4      # Keep weekly backups for 4 weeks
  months: 12    # Keep monthly backups for 12 months

# Per-target retention
volumes:
  - name: "critical_data"
    retention:
      days: 90
      months: 24
      
  - name: "temp_data"
    retention:
      days: 7
```

### Multiple Backup Destinations

Backup to multiple locations:

```yaml
destinations:
  # Primary: Backblaze B2
  - type: b2
    bucket: ${B2_BUCKET}
    key_id: ${B2_KEY_ID}
    app_key: ${B2_APP_KEY}
    
  # Secondary: Local storage
  - type: local
    path: /backup/local
    
  # Tertiary: S3
  - type: s3
    bucket: ${S3_BUCKET}
    region: us-east-1
    access_key: ${AWS_ACCESS_KEY}
    secret_key: ${AWS_SECRET_KEY}
```

## Applying Configuration Changes

### Via Ansible (Recommended)
1. Edit `ansible/roles/stack-back/templates/config.yaml.j2`
2. Commit changes
3. Run Ansible:
   ```bash
   make configure
   ```

### Manual Method
1. SSH to server
2. Edit `/opt/stack-back/config.yaml`
3. Restart stack-back:
   ```bash
   systemctl restart stack-back
   ```

## Monitoring Backups

### Check Backup Status
```bash
# View stack-back logs
journalctl -u stack-back -f

# Check last backup time
systemctl status stack-back

# List backup files in B2
# (Use B2 CLI or web interface)
```

### Verify Backup Success
```bash
# Check stack-back logs for errors
journalctl -u stack-back --since "24 hours ago" | grep -i error

# Verify backup files exist in destination
ls -lh /path/to/backups/
```

### Backup Notifications

Add notifications to config:

```yaml
notifications:
  # Email notifications
  - type: email
    smtp:
      host: smtp.gmail.com
      port: 587
      username: ${SMTP_USER}
      password: ${SMTP_PASSWORD}
    from: "[email protected]"
    to: "[email protected]"
    on_failure: true
    on_success: false  # Only notify on failure
    
  # Slack notifications
  - type: slack
    webhook_url: ${SLACK_WEBHOOK}
    on_failure: true
    on_success: false
```

## Restoring from Backups

### Restore Volumes

1. **Stop the application**
   ```bash
   docker-compose down
   ```

2. **Download backup from B2**
   ```bash
   # Use B2 CLI or stack-back restore command
   stack-back restore volume app_data --date 2024-01-15
   ```

3. **Extract backup**
   ```bash
   # Backups are typically tar.gz files
   tar -xzf app_data_2024-01-15.tar.gz -C /var/lib/docker/volumes/app_data/_data/
   ```

4. **Restart application**
   ```bash
   docker-compose up -d
   ```

### Restore Databases

1. **Stop the application**
   ```bash
   docker-compose down
   ```

2. **Download database backup**
   ```bash
   stack-back restore database appdb --date 2024-01-15
   ```

3. **Restore to database**
   ```bash
   # For PostgreSQL
   docker-compose up -d db
   docker exec -i db psql -U user appdb < appdb_2024-01-15.sql
   
   # For MySQL
   docker exec -i db mysql -u user -p appdb < appdb_2024-01-15.sql
   ```

4. **Restart application**
   ```bash
   docker-compose up -d
   ```

### Point-in-Time Recovery

For critical data, keep more frequent backups:

```yaml
volumes:
  - name: "critical_data"
    schedule: "0 */2 * * *"  # Every 2 hours
    retention:
      hours: 48   # Keep hourly backups for 48 hours
      days: 30    # Keep daily backups for 30 days
      weeks: 12   # Keep weekly backups for 12 weeks
```

## Troubleshooting

### Backups Not Running
**Check:**
```bash
# Is stack-back service running?
systemctl status stack-back

# Check for errors in logs
journalctl -u stack-back --since "24 hours ago"

# Verify cron schedule
journalctl -u cron --since "24 hours ago" | grep stack-back
```

### Backup Failures
**Common issues:**
- Insufficient disk space
- B2 credentials expired or invalid
- Network connectivity issues
- Database connection failures

**Solutions:**
```bash
# Check disk space
df -h

# Verify B2 credentials
# Update in Infisical if needed

# Test B2 connectivity
curl https://api.backblazeb2.com/b2api/v2/b2_authorize_account

# Test database connection
docker exec db pg_isready  # PostgreSQL
```

### Backup Files Too Large
**Optimize:**
```yaml
volumes:
  - name: "large_volume"
    compress: true  # Enable compression
    compression_level: 9  # Maximum compression
    
  # Exclude unnecessary files
  - name: "app_data"
    exclude_patterns:
      - "*.log"
      - "*.tmp"
      - "cache/*"
```

### Restore Fails
**Check:**
- Backup file integrity
- Sufficient disk space for restore
- Correct permissions
- Database is accessible

### Missing Backups
**Verify:**
```bash
# Check B2 bucket contents
# Use B2 web interface or CLI

# Verify stack-back configuration
cat /opt/stack-back/config.yaml

# Check if backups are being created but not uploaded
ls -lh /opt/stack-back/temp/
```

## Best Practices

1. **Test Restores Regularly**
   - Verify backups can actually be restored
   - Test restore procedure in dev environment
   - Document restore process

2. **Monitor Backup Size**
   - Watch for unexpected growth
   - Adjust retention as needed
   - Clean up old backups periodically

3. **Secure Backup Data**
   - Encrypt sensitive backups
   - Restrict access to backup storage
   - Rotate backup credentials regularly

4. **Multiple Backup Destinations**
   - Don't rely on single backup location
   - Consider geographic diversity
   - Test all backup destinations

5. **Document Configuration**
   - Keep notes on custom settings
   - Document restore procedures
   - Track backup schedule changes

6. **Automated Verification**
   ```yaml
   verification:
     enabled: true
     schedule: "0 4 * * 0"  # Weekly verification
     sample_size: 1  # Verify 1 random backup
   ```

## Advanced Configuration

### Incremental Backups
```yaml
volumes:
  - name: "large_dataset"
    incremental: true
    full_backup_schedule: "0 2 * * 0"  # Weekly full backup
    incremental_schedule: "0 2 * * 1-6"  # Daily incremental
```

### Pre/Post Backup Hooks
```yaml
volumes:
  - name: "app_data"
    pre_backup:
      - docker-compose exec app php artisan down
    post_backup:
      - docker-compose exec app php artisan up
```

### Database-Specific Options
```yaml
databases:
  explicit:
    - name: "postgres_app"
      type: postgresql
      options:
        dump_format: "custom"  # Custom format for pg_dump
        parallel_jobs: 4  # Parallel dump
        
    - name: "mysql_app"
      type: mysql
      options:
        single_transaction: true
        quick: true
```

## Resources

- [stack-back Documentation](https://stack-back.readthedocs.io)
- [Backblaze B2 Documentation](https://www.backblaze.com/b2/docs/)
- [Backup Best Practices](https://www.backblaze.com/blog/the-3-2-1-backup-strategy/)

## Next Steps

After configuring backups:
- Test restore procedure in dev environment
- Set up backup monitoring and alerts
- Document restore process for your team
- Review and adjust retention policies quarterly
