# Deploy Compose Application

Deploy and manage the Docker Compose application using the compose-template's deployment system. This skill covers application deployment, updates, and management.

## What This Skill Does

This skill guides Docker Compose application deployment:

1. Customizing the docker-compose.yaml file
2. Configuring Traefik labels for routing
3. Adding application secrets to Infisical
4. Deploying the application via Ansible
5. Managing updates and rollbacks

## Usage

Invoke this skill when:
- Deploying a new application for the first time
- Updating an existing application
- Troubleshooting deployment issues
- Configuring routing and secrets

## Application Deployment Workflow

### 1. Customize docker-compose.yaml

The template provides a base `docker-compose.yaml` that you modify for your app:

```yaml
version: '3.8'

services:
  # Your application containers go here
  web:
    image: nginx:alpine
    labels:
      # Traefik routing configuration
      - "traefik.enable=true"
      - "traefik.http.routers.web.rule=Host(`${APP_DOMAIN}`)"
      - "traefik.http.routers.web.entrypoints=websecure"
      - "traefik.http.routers.web.tls=true"
      - "traefik.http.services.web.loadbalancer.server.port=80"
    networks:
      - web

  # Add more services as needed
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}  # Injected by Infisical
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - web

networks:
  web:
    external: true

volumes:
  db_data:
```

### 2. Configure Traefik Labels

The frontend service needs Traefik labels for routing:

**Required labels:**
```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.{name}.rule=Host(`${APP_DOMAIN}`)"
  - "traefik.http.routers.{name}.entrypoints=websecure"
  - "traefik.http.routers.{name}.tls=true"
  - "traefik.http.services.{name}.loadbalancer.server.port=<port>"
```

**Optional labels:**
```yaml
# HTTPS redirect
- "traefik.http.routers.{name}-http.rule=Host(`${APP_DOMAIN}`)"
- "traefik.http.routers.{name}-http.entrypoints=web"
- "traefik.http.routers.{name}-http.middlewares=https-redirect"

# Middleware for redirects
- "traefik.http.middlewares.https-redirect.redirectscheme.scheme=https"
```

### 3. Add Application Secrets to Infisical

Add any app-specific secrets to the Infisical project:

1. **Login to Infisical**
   - Go to your Infisical dashboard
   - Find the project for your app (named after repository)

2. **Add Secrets**
   - Navigate to the appropriate environment (dev/prod)
   - Add secrets your app needs:
     - Database passwords
     - API keys
     - Service credentials
     - Configuration values

3. **Reference in Compose File**
   ```yaml
   environment:
     - API_KEY=${API_KEY}           # From Infisical
     - DATABASE_URL=${DATABASE_URL} # From Infisical
   ```

### 4. Deploy via Ansible

**Automated (GitHub Actions):**
- Push changes to repository
- GitHub Actions runs Ansible playbook
- Application is deployed with secrets injected

**Manual Deployment:**

1. **Set Up Local Environment**
   ```bash
   ./local-init.sh app
   source .env.local
   ```

2. **Deploy the Application**
   ```bash
   make compose
   ```
   This:
   - Uses Infisical CLI to inject secrets
   - Runs docker-compose up on remote host
   - Starts/restarts services as needed

## Secret Injection

A key feature of compose-template is just-in-time secret injection:

```bash
infisical run --env prod --command "make compose"
```

**Benefits:**
- Secrets never touch disk on server*
- Centralized secret management
- Easy rotation and updates
- Environment-specific configurations

*Some secrets may end up in container volumes

## Managing the Application

### View Running Containers
```bash
# SSH to server or use Ansible
docker ps

# View specific service
docker ps --filter "name=web"
```

### View Logs
```bash
# All services
docker-compose logs

# Specific service
docker-compose logs web

# Follow logs
docker-compose logs -f web
```

### Restart Services
```bash
# Restart all
docker-compose restart

# Restart specific service
docker-compose restart web
```

### Stop Application
```bash
docker-compose down
```

### Stop and Remove Volumes
```bash
# WARNING: This deletes data
docker-compose down -v
```

### Update Application
1. Modify docker-compose.yaml or update image tags
2. Commit and push (or deploy manually)
3. Ansible will gracefully restart services

## Application Updates

### Updating Container Images
```yaml
services:
  web:
    image: nginx:1.24  # Update version
```

### Adding New Services
1. Add service definition to docker-compose.yaml
2. Configure networking (use 'web' network)
3. Add required secrets to Infisical
4. Deploy

### Modifying Service Configuration
1. Update docker-compose.yaml
2. Test locally if possible
3. Deploy to server
4. Verify services restart correctly

## Networking

The template uses a shared 'web' network:

```yaml
networks:
  web:
    external: true
```

This network:
- Is created by the template
- Shared across all services
- Used by Traefik for routing
- Allows inter-service communication

## Volumes and Data Persistence

### Named Volumes
```yaml
volumes:
  app_data:
```

Automatically backed up by stack-back.

### Bind Mounts
```yaml
volumes:
  - ./config:/app/config
```

Not automatically backed up - use for configuration only.

### Volume Management
```bash
# List volumes
docker volume ls

# Inspect volume
docker volume inspect <volume_name>

# Remove unused volumes
docker volume prune
```

## Traefik Integration

The template includes Traefik as a reverse proxy:

### How It Works
1. Traefik monitors Docker labels
2. Automatically configures routes
3. Handles HTTPS with Let's Encrypt
4. Provides load balancing

### Debugging Traefik
```bash
# View Traefik logs
docker logs traefik

# Check Traefik configuration
docker exec traefik cat /etc/traefik/traefik.yml
```

## Troubleshooting

### Service Won't Start
**Check logs:**
```bash
docker-compose logs <service_name>
```
**Common issues:**
- Missing environment variables
- Port conflicts
- Image pull failures
- Volume permission issues

### Can't Access Application
**Check Traefik routing:**
```bash
docker logs traefik | grep <service_name>
```
**Verify:**
- Traefik labels are correct
- Service is running
- DNS is properly configured
- Cloudflare Tunnel is active

### Secrets Not Available
**Verify Infisical:**
- Secrets exist in correct environment
- Infisical CLI is authenticated
- Environment variables are referenced correctly

**Check deployment:**
```bash
# On server, verify environment
docker exec <container> env | grep <SECRET_NAME>
```

### Database Connection Issues
**Check:**
- Database service is running
- Network connectivity between services
- Database credentials are correct
- Database is initialized

### Performance Issues
**Monitor resources:**
```bash
# Container resource usage
docker stats

# System resources
htop
```

### Container Crashes
**Check logs:**
```bash
docker-compose logs --tail=100 <service>
```

**Common causes:**
- Out of memory
- Application errors
- Missing dependencies
- Configuration errors

## Rollback Strategy

If deployment goes wrong:

1. **Quick Rollback**
   ```bash
   # Revert to previous commit
   git revert HEAD
   git push
   
   # Or checkout previous version
   git checkout <previous-commit>
   make compose
   ```

2. **Manual Rollback**
   ```bash
   # Stop current version
   docker-compose down
   
   # Pull previous image versions
   docker-compose pull
   
   # Start previous version
   docker-compose up -d
   ```

## Best Practices

1. **Test Locally First**
   - Use Docker Compose locally
   - Verify all services start
   - Check logs for errors

2. **Incremental Updates**
   - Update one service at a time
   - Verify after each update
   - Monitor logs during update

3. **Health Checks**
   ```yaml
   services:
     web:
       healthcheck:
         test: ["CMD", "curl", "-f", "http://localhost/health"]
         interval: 30s
         timeout: 3s
         retries: 3
   ```

4. **Resource Limits**
   ```yaml
   services:
     web:
       deploy:
         resources:
           limits:
             cpus: '0.5'
             memory: 512M
   ```

5. **Logging Configuration**
   ```yaml
   services:
     web:
       logging:
         driver: "json-file"
         options:
           max-size: "10m"
           max-file: "3"
   ```

## Next Steps

After deployment:
- Use `/compose-backup` to configure backups
- Monitor application logs and metrics
- Set up alerting for issues
- Document any custom configurations
