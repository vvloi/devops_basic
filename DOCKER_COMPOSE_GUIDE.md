# Docker Compose Guide

This docker-compose setup includes the following services with their latest images:

## Services

### 1. PostgreSQL
- **Image**: `postgres:latest`
- **Port**: 5432
- **Default Credentials**:
  - Username: `admin`
  - Password: `admin123`
  - Database: `mydb`

### 2. Redis
- **Image**: `redis:latest`
- **Port**: 6379
- **Description**: In-memory data store

### 3. MongoDB
- **Image**: `mongo:latest`
- **Port**: 27017
- **Default Credentials**:
  - Username: `admin`
  - Password: `admin123`

### 4. Grafana
- **Image**: `grafana/grafana:latest`
- **Port**: 3000
- **Default Credentials**:
  - Username: `admin`
  - Password: `admin123`
- **Access**: http://localhost:3000

### 5. Prometheus
- **Image**: `prom/prometheus:latest`
- **Port**: 9090
- **Access**: http://localhost:9090
- **Configuration**: Uses `prometheus.yml` for monitoring configuration

### 6. Keycloak
- **Image**: `quay.io/keycloak/keycloak:latest`
- **Port**: 8080
- **Default Credentials**:
  - Username: `admin`
  - Password: `admin123`
- **Access**: http://localhost:8080
- **Note**: Configured to use PostgreSQL as database

## Usage

### Start all services:
```bash
docker compose up -d
```

### Stop all services:
```bash
docker compose down
```

### View logs:
```bash
docker compose logs -f
```

### View logs for a specific service:
```bash
docker compose logs -f <service_name>
```

### Stop and remove all containers and volumes:
```bash
docker compose down -v
```

## Volumes

All services use named volumes to persist data:
- `postgres_data`: PostgreSQL database files
- `redis_data`: Redis data files
- `mongodb_data`: MongoDB database files
- `grafana_data`: Grafana configuration and dashboards
- `prometheus_data`: Prometheus metrics data

## Network

All services are connected through a bridge network called `devops_network` for inter-service communication.

## Important Notes

1. **Security**: Default passwords are provided for development purposes. Please change them for production use.
2. **Keycloak**: Requires PostgreSQL to be running first (handled by `depends_on`).
3. **Grafana**: Can be integrated with Prometheus for monitoring dashboards.
4. **Prometheus**: Basic configuration is provided in `prometheus.yml`. Customize it based on your monitoring needs.
