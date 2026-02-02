# EasyExpressPro Deployment Guide

## Overview

This guide covers deploying EasyExpressPro to production environments. The application consists of a Next.js frontend and NestJS backend with MongoDB and Redis.

## Prerequisites

- Node.js >= 18.0.0
- pnpm >= 8.0.0
- MongoDB (local or MongoDB Atlas)
- Redis (for caching in production)
- AWS S3 (for file storage)
- Clerk account (for authentication)
- Stripe account (for payments)
- Sentry account (for error monitoring)

## Environment Variables

### Backend (.env)

```bash
# Application
NODE_ENV=production
PORT=3001
FRONTEND_URL=https://your-domain.com

# Database
MONGODB_URI=mongodb://localhost:27017/easy-express-pro
# Or MongoDB Atlas:
# MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/easy-express-pro

# Redis (for caching)
REDIS_URL=redis://localhost:6379
# Or Redis Cloud:
# REDIS_URL=redis://username:password@host:port

# Clerk Authentication
CLERK_SECRET_KEY=sk_live_your_secret_key
CLERK_PUBLISHABLE_KEY=pk_live_your_publishable_key
CLERK_WEBHOOK_SECRET=whsec_your_webhook_secret

# Stripe Payments
STRIPE_SECRET_KEY=sk_live_your_stripe_secret_key
STRIPE_WEBHOOK_SECRET=whsec_your_stripe_webhook_secret

# AWS S3 (for file storage)
AWS_ACCESS_KEY_ID=your_aws_access_key
AWS_SECRET_ACCESS_KEY=your_aws_secret_key
AWS_REGION=us-east-1
AWS_S3_BUCKET_NAME=your-bucket-name

# SendGrid (for emails)
SENDGRID_API_KEY=your_sendgrid_api_key

# Sentry (for error monitoring)
SENTRY_DSN=https://your-sentry-dsn@sentry.io/project-id
```

### Frontend (.env.local)

```bash
# Clerk Authentication
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_live_your_publishable_key
CLERK_SECRET_KEY=sk_live_your_secret_key

# Backend URL
NEXT_PUBLIC_BACKEND_URL=https://api.your-domain.com

# Site URL
NEXT_PUBLIC_SITE_URL=https://your-domain.com

# AWS S3 (for direct uploads)
NEXT_PUBLIC_AWS_REGION=us-east-1
NEXT_PUBLIC_AWS_S3_BUCKET_NAME=your-bucket-name
```

## Docker Deployment

### Dockerfile (Backend)

```dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

# Install pnpm
RUN npm install -g pnpm

# Copy package files
COPY package.json pnpm-lock.yaml ./
COPY apps/backend/package.json ./apps/backend/
COPY shared/types/package.json ./shared/types/

# Install dependencies
RUN pnpm install --frozen-lockfile

# Copy source code
COPY . .

# Build the application
RUN pnpm build

# Production stage
FROM node:18-alpine AS production

WORKDIR /app

# Install pnpm
RUN npm install -g pnpm

# Copy package files
COPY package.json pnpm-lock.yaml ./
COPY apps/backend/package.json ./apps/backend/
COPY shared/types/package.json ./shared/types/

# Install production dependencies
RUN pnpm install --frozen-lockfile --prod

# Copy built application
COPY --from=builder /app/apps/backend/dist ./apps/backend/dist
COPY --from=builder /app/shared/types/dist ./shared/types/dist

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nestjs -u 1001
USER nestjs

EXPOSE 3001

CMD ["node", "apps/backend/dist/main"]
```

### Dockerfile (Frontend)

```dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

# Install pnpm
RUN npm install -g pnpm

# Copy package files
COPY package.json pnpm-lock.yaml ./
COPY apps/frontend/package.json ./apps/frontend/
COPY shared/types/package.json ./shared/types/

# Install dependencies
RUN pnpm install --frozen-lockfile

# Copy source code
COPY . .

# Build the application
RUN pnpm build

# Production stage
FROM node:18-alpine AS production

WORKDIR /app

# Install pnpm
RUN npm install -g pnpm

# Copy package files
COPY package.json pnpm-lock.yaml ./
COPY apps/frontend/package.json ./apps/frontend/

# Install production dependencies
RUN pnpm install --frozen-lockfile --prod

# Copy built application
COPY --from=builder /app/apps/frontend/.next ./apps/frontend/.next
COPY --from=builder /app/apps/frontend/public ./apps/frontend/public
COPY --from=builder /app/shared/types/dist ./shared/types/dist

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001
USER nextjs

EXPOSE 3000

CMD ["node", "apps/frontend/server.js"]
```

### docker-compose.yml

```yaml
version: '3.8'

services:
  frontend:
    build:
      context: .
      dockerfile: apps/frontend/Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - NEXT_PUBLIC_BACKEND_URL=http://backend:3001
    depends_on:
      - backend

  backend:
    build:
      context: .
      dockerfile: apps/backend/Dockerfile
    ports:
      - "3001:3001"
    environment:
      - NODE_ENV=production
      - MONGODB_URI=mongodb://mongo:27017/easy-express-pro
      - REDIS_URL=redis://redis:6379
    depends_on:
      - mongo
      - redis

  mongo:
    image: mongo:7
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  mongo_data:
  redis_data:
```

## Cloud Deployment

### AWS Deployment

#### 1. EC2 Setup

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install pnpm
sudo npm install -g pnpm

# Install PM2 for process management
sudo npm install -g pm2

# Install Nginx
sudo apt install nginx -y

# Install MongoDB
wget -qO - https://www.mongodb.org/static/pgp/server-7.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org

# Install Redis
sudo apt install redis-server -y
```

#### 2. Application Setup

```bash
# Clone repository
git clone https://github.com/your-username/easy-express-pro.git
cd easy-express-pro

# Install dependencies
pnpm install

# Build applications
pnpm build

# Set up environment variables
cp apps/backend/.env.example apps/backend/.env
cp apps/frontend/.env.example apps/frontend/.env.local
# Edit the files with your production values
```

#### 3. PM2 Configuration

Create `ecosystem.config.js`:

```javascript
module.exports = {
  apps: [
    {
      name: 'easy-express-pro-backend',
      script: 'apps/backend/dist/main.js',
      instances: 'max',
      exec_mode: 'cluster',
      env: {
        NODE_ENV: 'production',
        PORT: 3001,
      },
      error_file: './logs/backend-error.log',
      out_file: './logs/backend-out.log',
      log_file: './logs/backend-combined.log',
    },
    {
      name: 'easy-express-pro-frontend',
      script: 'apps/frontend/server.js',
      instances: 'max',
      exec_mode: 'cluster',
      env: {
        NODE_ENV: 'production',
        PORT: 3000,
      },
      error_file: './logs/frontend-error.log',
      out_file: './logs/frontend-out.log',
      log_file: './logs/frontend-combined.log',
    },
  ],
};
```

```bash
# Start applications
pm2 start ecosystem.config.js

# Save PM2 configuration
pm2 save
pm2 startup
```

#### 4. Nginx Configuration

Create `/etc/nginx/sites-available/easy-express-pro`:

```nginx
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;

    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com www.your-domain.com;

    # SSL configuration
    ssl_certificate /path/to/your/certificate.crt;
    ssl_certificate_key /path/to/your/private.key;

    # Frontend
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    # Backend API
    location /api/ {
        proxy_pass http://localhost:3001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    # WebSocket
    location /socket.io/ {
        proxy_pass http://localhost:3001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # File uploads
    client_max_body_size 10M;
}
```

```bash
# Enable site
sudo ln -s /etc/nginx/sites-available/easy-express-pro /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### Vercel Deployment (Frontend)

1. Connect your GitHub repository to Vercel
2. Set environment variables in Vercel dashboard
3. Deploy automatically on push to main branch

### Railway Deployment

1. Connect your GitHub repository to Railway
2. Set environment variables
3. Deploy automatically

## Database Setup

### MongoDB Atlas

1. Create a MongoDB Atlas account
2. Create a new cluster
3. Create a database user
4. Whitelist your IP addresses
5. Get the connection string

### Local MongoDB

```bash
# Start MongoDB service
sudo systemctl start mongod
sudo systemctl enable mongod

# Create database and user
mongo
use easy-express-pro
db.createUser({
  user: "app_user",
  pwd: "app_password",
  roles: [{ role: "readWrite", db: "easy-express-pro" }]
})
```

## Redis Setup

### Redis Cloud

1. Create a Redis Cloud account
2. Create a new database
3. Get the connection string

### Local Redis

```bash
# Start Redis service
sudo systemctl start redis-server
sudo systemctl enable redis-server

# Test connection
redis-cli ping
```

## SSL Certificate

### Let's Encrypt (Free)

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx -y

# Get certificate
sudo certbot --nginx -d your-domain.com -d www.your-domain.com

# Auto-renewal
sudo crontab -e
# Add: 0 12 * * * /usr/bin/certbot renew --quiet
```

## Monitoring

### Application Monitoring

```bash
# PM2 monitoring
pm2 monit

# View logs
pm2 logs

# Restart applications
pm2 restart all
```

### System Monitoring

```bash
# Install monitoring tools
sudo apt install htop iotop nethogs -y

# Monitor system resources
htop
iotop
nethogs
```

## Backup Strategy

### Database Backup

```bash
# Create backup script
cat > backup.sh << 'EOF'
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
mongodump --uri="mongodb://localhost:27017/easy-express-pro" --out="/backups/mongodb_$DATE"
tar -czf "/backups/mongodb_$DATE.tar.gz" "/backups/mongodb_$DATE"
rm -rf "/backups/mongodb_$DATE"
EOF

chmod +x backup.sh

# Schedule daily backups
crontab -e
# Add: 0 2 * * * /path/to/backup.sh
```

### File Storage Backup

- Enable S3 versioning
- Set up S3 lifecycle policies
- Configure cross-region replication

## Security Checklist

- [ ] Use strong passwords for all services
- [ ] Enable firewall (UFW)
- [ ] Keep system updated
- [ ] Use HTTPS everywhere
- [ ] Enable MongoDB authentication
- [ ] Configure Redis authentication
- [ ] Set up proper CORS policies
- [ ] Enable rate limiting
- [ ] Monitor error logs
- [ ] Set up alerts for critical issues

## Troubleshooting

### Common Issues

1. **Port already in use**
   ```bash
   sudo lsof -i :3000
   sudo kill -9 <PID>
   ```

2. **MongoDB connection failed**
   ```bash
   sudo systemctl status mongod
   sudo systemctl restart mongod
   ```

3. **Redis connection failed**
   ```bash
   sudo systemctl status redis-server
   sudo systemctl restart redis-server
   ```

4. **Nginx configuration error**
   ```bash
   sudo nginx -t
   sudo systemctl reload nginx
   ```

### Log Locations

- Application logs: `./logs/`
- Nginx logs: `/var/log/nginx/`
- System logs: `/var/log/syslog`
- MongoDB logs: `/var/log/mongodb/`
- Redis logs: `/var/log/redis/`

## Performance Optimization

### Database Optimization

```javascript
// Add indexes
db.shipments.createIndex({ "customerId": 1 })
db.shipments.createIndex({ "branchId": 1 })
db.shipments.createIndex({ "status": 1 })
db.shipments.createIndex({ "createdAt": -1 })
```

### Caching Strategy

- Enable Redis caching in production
- Set appropriate TTL values
- Use cache tags for invalidation
- Monitor cache hit rates

### CDN Setup

- Use CloudFront for static assets
- Enable gzip compression
- Set proper cache headers
- Optimize images

## Scaling

### Horizontal Scaling

- Use load balancer (ALB/ELB)
- Deploy multiple instances
- Use Redis for session storage
- Implement database sharding

### Vertical Scaling

- Increase server resources
- Optimize database queries
- Use connection pooling
- Implement caching layers