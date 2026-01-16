# Docker Deployment Guide for Farm Management System

## Prerequisites
- Install [Docker](https://www.docker.com/products/docker-desktop)
- Install [Docker Compose](https://docs.docker.com/compose/install/)

## Files Created

### 1. **Dockerfile**
- Builds the Flask application image
- Uses Python 3.9 slim image
- Installs all dependencies from requirements.txt
- Exposes port 5000

### 2. **docker-compose.yml**
- Orchestrates two services: MySQL database and Flask application
- MySQL container with persistent volume
- Flask app automatically connects to MySQL
- Health checks to ensure proper startup order

### 3. **requirements.txt**
- Python package dependencies

## Deployment Steps

### Step 1: Update main.py Database Connection
Modify the database URI in `main.py` to use environment variables:

```python
# OLD (line 25):
app.config['SQLALCHEMY_DATABASE_URI']='mysql://root:@localhost/farmers'

# NEW:
import os
db_host = os.getenv('MYSQL_HOST', 'localhost')
db_user = os.getenv('MYSQL_USER', 'root')
db_pass = os.getenv('MYSQL_PASSWORD', '')
db_name = os.getenv('MYSQL_DB', 'farmers')
app.config['SQLALCHEMY_DATABASE_URI']=f'mysql://{db_user}:{db_pass}@{db_host}:3306/{db_name}'
```

### Step 2: Run with Docker Compose

```bash
cd "farmer system"
docker-compose up --build
```

The application will:
1. Start MySQL container and initialize the database
2. Wait for MySQL to be healthy
3. Start the Flask app on http://localhost:5000

### Step 3: Verify Deployment

```bash
# Check running containers
docker ps

# View logs
docker-compose logs -f

# Access the application
# Open http://localhost:5000 in your browser
```

## Useful Docker Commands

```bash
# Stop containers
docker-compose down

# Stop and remove volumes
docker-compose down -v

# Rebuild images
docker-compose build --no-cache

# View logs
docker-compose logs flask_app
docker-compose logs mysql

# Execute command in container
docker-compose exec flask_app bash
docker-compose exec mysql bash

# Scale containers
docker-compose up -d --scale flask_app=3
```

## Production Deployment

For production, update `docker-compose.yml`:
1. Use strong passwords for MySQL_ROOT_PASSWORD
2. Add environment variables file (.env)
3. Use volume for persistent uploads
4. Configure reverse proxy (Nginx)
5. Enable SSL/TLS
6. Use secrets management
7. Set proper logging

## Troubleshooting

**Connection refused error:**
```bash
# Ensure MySQL is running and healthy
docker-compose ps
docker-compose logs mysql
```

**Port already in use:**
```bash
# Change ports in docker-compose.yml:
ports:
  - "3307:3306"  # MySQL
  - "5001:5000"  # Flask
```

**Database not initialized:**
```bash
# Manually run farmers.sql
docker-compose exec mysql mysql -uroot -proot farmers < farmers.sql
```
