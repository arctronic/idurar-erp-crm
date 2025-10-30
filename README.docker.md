# Docker Deployment Guide for IDURAR ERP CRM

This guide explains how to deploy the IDURAR ERP CRM application using Docker and Docker Compose.

## Quick Start

### 1. Prerequisites

- Docker Engine 20.10 or higher
- Docker Compose 2.0 or higher

### 2. Setup Environment

```bash
# Clone the repository
git clone https://github.com/idurar/idurar-erp-crm.git
cd idurar-erp-crm

# Create environment file
cp .env.example .env

# Edit .env and set your JWT_SECRET (REQUIRED)
# Windows: notepad .env
# Linux/Mac: nano .env
```

### 3. Deploy

```bash
# Build and start all services
docker-compose up -d

# Wait for services to start (about 30-60 seconds)
# Then initialize the database
docker-compose exec backend npm run setup
```

### 4. Access

- **Application URL**: http://localhost:8085
- **Default Login**:
  - Email: `admin@admin.com`
  - Password: `admin123`

## What Gets Deployed

The Docker setup includes three services:

1. **MongoDB** (internal only)
   - Database for storing all application data
   - Persistent volume for data storage
   - Not exposed to host machine

2. **Backend** (port 8888 internal)
   - Node.js/Express API server
   - Connects to MongoDB
   - Not exposed to host machine

3. **Frontend** (port 8085)
   - React application served by Nginx
   - Proxies API requests to backend
   - **Accessible at http://localhost:8085**

## Useful Commands

### Service Management

```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# Restart services
docker-compose restart

# View logs
docker-compose logs -f

# View logs for specific service
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f mongodb
```

### Database Operations

```bash
# Run database setup (creates admin user)
docker-compose exec backend npm run setup

# Run database upgrade
docker-compose exec backend npm run upgrade

# Reset database (WARNING: deletes all data)
docker-compose exec backend npm run reset

# Access MongoDB shell
docker-compose exec mongodb mongosh
```

### Rebuild After Code Changes

```bash
# Rebuild and restart
docker-compose up -d --build

# Rebuild specific service
docker-compose up -d --build backend
docker-compose up -d --build frontend
```

### Cleanup

```bash
# Stop and remove containers (keeps volumes)
docker-compose down

# Stop and remove everything including data
docker-compose down -v

# Remove all unused images
docker image prune -a
```

## Environment Variables

Edit the `.env` file to configure the application. Key variables:

```bash
# Database (internal Docker network)
DATABASE=mongodb://mongodb:27017/idurar_db

# Security (REQUIRED - change this!)
JWT_SECRET=your_private_jwt_secret_key_change_this_in_production

# Application
NODE_ENV=production
PUBLIC_SERVER_FILE=http://localhost:8085/

# Optional: Email service
RESEND_API=your_resend_api_key

# Optional: AI features
OPENAI_API_KEY=your_openai_api_key
```

## EasyPanel Deployment

To deploy on EasyPanel:

1. Create a new project in EasyPanel dashboard
2. Connect your Git repository
3. Configure environment variables in EasyPanel:
   - Set `JWT_SECRET` to a secure random string
   - Set `DATABASE` to `mongodb://mongodb:27017/idurar_db`
   - Set `PUBLIC_SERVER_FILE` to your domain URL
4. Deploy using the `docker-compose.yml` file
5. Run setup: `docker-compose exec backend npm run setup`

## Troubleshooting

### Services won't start

```bash
# Check service status
docker-compose ps

# Check logs
docker-compose logs -f

# Restart services
docker-compose restart
```

### Database connection errors

```bash
# Make sure MongoDB is running
docker-compose ps mongodb

# Check MongoDB logs
docker-compose logs mongodb

# Restart MongoDB
docker-compose restart mongodb
```

### Port 8085 already in use

Edit `docker-compose.yml` and change the frontend port mapping:
```yaml
frontend:
  ports:
    - "9090:8085"  # Use port 9090 instead
```

### Cannot access application

1. Check all services are running: `docker-compose ps`
2. Check frontend logs: `docker-compose logs frontend`
3. Verify port 8085 is accessible: `curl http://localhost:8085`
4. Try restarting: `docker-compose restart`

### Reset everything

```bash
# Stop and remove everything
docker-compose down -v

# Remove images
docker-compose down --rmi all

# Start fresh
docker-compose up -d
docker-compose exec backend npm run setup
```

## Production Deployment

For production deployments:

1. **Change JWT_SECRET** to a strong random string
2. **Use external MongoDB** for better performance and backups
3. **Configure SSL/TLS** using a reverse proxy (nginx, traefik)
4. **Set up backups** for MongoDB volumes
5. **Monitor logs** using a logging solution
6. **Update regularly** to get security patches

Example production `.env`:
```bash
DATABASE=mongodb://your-mongodb-host:27017/idurar_db
JWT_SECRET=very-long-random-secure-string-here
NODE_ENV=production
PUBLIC_SERVER_FILE=https://yourdomain.com/
```

## Data Persistence

Docker volumes are used for data persistence:
- `mongodb_data`: Database files
- `mongodb_config`: MongoDB configuration
- `backend_uploads`: User-uploaded files

These volumes persist even when containers are stopped or removed.

To backup your data:
```bash
# Backup MongoDB
docker-compose exec mongodb mongodump --out=/data/backup

# Copy backup from container
docker cp idurar_mongodb:/data/backup ./mongodb-backup
```

## Support

For issues and questions:
- GitHub Issues: https://github.com/idurar/idurar-erp-crm/issues
- Documentation: https://www.idurarapp.com
