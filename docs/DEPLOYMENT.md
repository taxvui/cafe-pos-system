# Cafe Harmony - Production Deployment Guide

## Deployment Strategies

### Option 1: VPS Linux Deployment (Recommended)

#### Prerequisites
- Ubuntu 20.04+ or Debian 11+
- 2GB+ RAM, 20GB+ storage
- Root or sudo access

#### Setup Steps

```bash
# 1. Update system
sudo apt update && sudo apt upgrade -y

# 2. Install Docker & Docker Compose
sudo apt install -y curl wget git
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# 3. Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# 4. Clone project
git clone https://github.com/yourusername/cafe-pos-system.git
cd cafe-pos-system

# 5. Configure environment
cp .env.example .env.production
# Edit .env.production with your settings
nano .env.production

# 6. Create data directories
mkdir -p data/postgres data/redis logs

# 7. Set permissions
chmod 755 data/postgres data/redis logs

# 8. Start services
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# 9. Check status
docker-compose ps
docker-compose logs -f
```

#### Production Environment File (.env.production)

```env
# Update these for production
DATABASE_URL=postgresql://cafe_user:secure_password@postgres:5432/cafe_pos_prod
REDIS_URL=redis://redis:6379
JWT_SECRET=generate_secure_random_string_min_32_chars
NEXTAUTH_SECRET=another_secure_random_string
NODE_ENV=production
NEXTAUTH_URL=https://your-domain.com

# VietQR
VIETQR_API_KEY=your_napas_api_key
VIETQR_WEBHOOK_URL=https://your-domain.com/api/webhooks/vietqr

# Email
SMTP_HOST=smtp.gmail.com
SMTP_USER=your-email@gmail.com
SMTP_PASSWORD=your-app-password
```

#### Nginx Reverse Proxy Setup

```bash
# Install Nginx
sudo apt install -y nginx

# Copy config
sudo cp docker/nginx.conf /etc/nginx/sites-available/cafe-pos

# Enable site
sudo ln -s /etc/nginx/sites-available/cafe-pos /etc/nginx/sites-enabled/

# Test config
sudo nginx -t

# Start Nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

#### SSL/TLS with Let's Encrypt

```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx

# Get certificate
sudo certbot certonly --nginx -d your-domain.com -d www.your-domain.com

# Auto-renewal
sudo systemctl enable certbot.timer
sudo systemctl start certbot.timer
```

#### Monitoring & Logs

```bash
# View logs
docker-compose logs -f app
docker-compose logs -f postgres
docker-compose logs postgres --tail 100

# Check container health
docker-compose ps

# Monitor system resources
docker stats

# Check disk usage
df -h
du -sh *
```

#### Backup Strategy

```bash
# Daily backup script
#!/bin/bash
# backup.sh

BACKUP_DIR="/backup/cafe-pos"
DATE=$(date +%Y%m%d_%H%M%S)

# PostgreSQL backup
docker-compose exec -T postgres pg_dump -U postgres cafe_pos_prod | gzip > "$BACKUP_DIR/db_$DATE.sql.gz"

# Redis backup
docker-compose exec -T redis redis-cli --rdb $BACKUP_DIR/redis_$DATE.rdb

# Data files backup
tar -czf "$BACKUP_DIR/data_$DATE.tar.gz" data/

# Upload to S3 (Optional)
# aws s3 cp "$BACKUP_DIR/" s3://your-bucket/backups/

# Cleanup old backups (keep last 30 days)
find "$BACKUP_DIR" -mtime +30 -delete

echo "Backup completed: $DATE"
```

Schedule with crontab:
```
# Edit crontab
crontab -e

# Add daily backup at 2 AM
0 2 * * * /path/to/backup.sh
```

---

### Option 2: Docker Hub Deployment

```bash
# 1. Build image
docker build -t yourusername/cafe-pos:latest .

# 2. Push to Docker Hub
docker login
docker push yourusername/cafe-pos:latest

# 3. On server, pull and run
docker pull yourusername/cafe-pos:latest
docker run -d \
  -p 3000:3000 \
  --env-file .env.production \
  --name cafe-pos \
  yourusername/cafe-pos:latest
```

---

### Option 3: Kubernetes Deployment (Advanced)

#### Create Namespace

```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: cafe-pos
```

#### Deployment

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: cafe-pos
  name: cafe-pos-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: cafe-pos
  template:
    metadata:
      labels:
        app: cafe-pos
    spec:
      containers:
      - name: app
        image: yourusername/cafe-pos:latest
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: cafe-pos-secrets
              key: database-url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: cafe-pos-secrets
              key: redis-url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /api/health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
```

#### Service

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  namespace: cafe-pos
  name: cafe-pos-service
spec:
  selector:
    app: cafe-pos
  ports:
  - port: 80
    targetPort: 3000
  type: LoadBalancer
```

Deploy:
```bash
kubectl apply -f namespace.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

---

## Performance Optimization

### 1. Database Optimization

```sql
-- Create indexes for frequently queried fields
CREATE INDEX idx_orders_created_at ON orders(created_at);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_tables_status ON tables(status);
CREATE INDEX idx_payments_vietqr_id ON payments(vietqr_id);

-- Analyze tables
ANALYZE;
```

### 2. Redis Configuration

```bash
# Optimize Redis
redis-cli CONFIG SET maxmemory-policy allkeys-lru
redis-cli CONFIG SET databases 2
redis-cli CONFIG REWRITE
```

### 3. Application Level

```javascript
// Next.js compression
npm install compression
// In server.js
const compression = require('compression');
app.use(compression());
```

### 4. CDN Setup

Use CloudFlare for:
- Static asset caching
- DDoS protection
- SSL/TLS termination
- Image optimization

---

## Monitoring & Alerting

### Setup Monitoring

```bash
# Install Prometheus & Grafana
docker-compose -f docker-compose.monitoring.yml up -d
```

### Key Metrics to Monitor

- API response time (< 200ms)
- Database query time (< 100ms)
- CPU usage (< 70%)
- Memory usage (< 80%)
- Disk usage (< 85%)
- Error rate (< 0.1%)
- Payment webhook latency (< 5s)

### Alert Triggers

```yaml
# High error rate
- alert: HighErrorRate
  expr: rate(http_request_errors_total[5m]) > 0.01
  
# Database slow queries
- alert: SlowQueries
  expr: database_query_duration_seconds > 1
  
# Low disk space
- alert: LowDiskSpace
  expr: node_filesystem_avail_bytes < 5368709120
```

---

## Security Hardening

### 1. Firewall Rules

```bash
sudo ufw enable
sudo ufw allow 22/tcp      # SSH
sudo ufw allow 80/tcp      # HTTP
sudo ufw allow 443/tcp     # HTTPS
sudo ufw allow 3000/tcp    # Next.js (internal)
```

### 2. Database Security

```sql
-- Create restricted user
CREATE USER cafe_user WITH PASSWORD 'strong_password';
GRANT CONNECT ON DATABASE cafe_pos_prod TO cafe_user;
GRANT USAGE ON SCHEMA public TO cafe_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO cafe_user;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO cafe_user;
```

### 3. Environment Secrets

```bash
# Use Docker secrets for production
echo "your_jwt_secret" | docker secret create jwt_secret -
echo "your_db_password" | docker secret create db_password -
```

### 4. Regular Updates

```bash
# Auto-update packages
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades
```

---

## Troubleshooting

### Issue: Port Already in Use

```bash
# Find process using port 3000
sudo lsof -i :3000
# Kill process
sudo kill -9 <PID>
```

### Issue: Database Connection Error

```bash
# Check PostgreSQL status
docker-compose ps postgres

# Check logs
docker-compose logs postgres

# Restart database
docker-compose restart postgres
```

### Issue: Redis Connection Error

```bash
# Test Redis connection
redis-cli ping
# Should return: PONG

# Clear Redis cache
redis-cli FLUSHALL
```

### Issue: Webhook Not Receiving

```bash
# Check webhook endpoint
curl -X POST http://localhost:3000/api/webhooks/vietqr \
  -H "Content-Type: application/json" \
  -d '{"test": "data"}'

# Check logs
docker-compose logs app | grep webhook
```

---

## Rollback Strategy

```bash
# Keep previous versions tagged
docker tag yourusername/cafe-pos:latest yourusername/cafe-pos:v1.0.0
docker push yourusername/cafe-pos:v1.0.0

# To rollback
docker pull yourusername/cafe-pos:v1.0.0
docker stop cafe-pos
docker run -d -p 3000:3000 yourusername/cafe-pos:v1.0.0
```

---

## Maintenance Schedule

| Task | Frequency | Command |
|------|-----------|---------|
| Database Backup | Daily | `backup.sh` |
| Update Dependencies | Monthly | `docker-compose pull` |
| Certificate Renewal | Auto | Certbot auto-renewal |
| Log Cleanup | Weekly | `docker system prune` |
| Performance Analysis | Weekly | Check dashboards |
| Security Updates | As available | `apt upgrade` |

---

## Support

For deployment issues:
- Check logs: `docker-compose logs -f`
- Verify environment variables: `docker-compose exec app env`
- Test API: `curl http://localhost:3000/api/health`
- Database shell: `docker-compose exec postgres psql -U postgres`

**Repository**: https://github.com/taxvui/cafe-pos-system
**Issues**: https://github.com/taxvui/cafe-pos-system/issues
