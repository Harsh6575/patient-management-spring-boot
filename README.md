# 🏥 Patient Management System (Microservices)

This project is a microservices-based **Patient Management System** built with Spring Boot, Docker, and PostgreSQL.  
Each service runs independently and communicates via the **API Gateway**.

---

## 📌 Services Overview

| Service           | Description                              | Port |
|-------------------|------------------------------------------|------|
| analytics-service | Collects and exposes analytics data       | 8081 |
| api-gateway       | Entry point for all client requests       | 8080 |
| auth-service      | Handles authentication & authorization    | 8082 |
| billing-service   | Manages patient billing and payments      | 8083 |
| patient-service   | Manages patient records & appointments    | 8084 |

---

## 📂 Databases

| Database             | Used by        | Port  |
|----------------------|----------------|-------|
| auth-service-db      | auth-service   | 5432  |
| patient-service-db   | patient-service| 5433  |

---

## ⚙️ Environment Variables

Below are the environment variables needed for each service.  
Update the values in your `.env` file (or directly in `docker-compose.override.yml`) before running.

---

## First Create a network

```bash
docker network create internal
```

### Auth Service DB

```bash
docker run -d \
  --name auth-service-db \
  --network internal \
  --env-file ./auth-service-db.env \
  -p 5434:5432 \
  postgres:latest
```

`auth-service-db.env`

```env
POSTGRES_USER=admin_user
POSTGRES_PASSWORD=password
POSTGRES_DB=db
```

### 🔑 Auth Service (`auth-service`)

```env
JWT_SECRET=your-secret-key
SPRING_DATASOURCE_PASSWORD=password
SPRING_DATASOURCE_URL=jdbc:postgresql://auth-service-db:5432/db
SPRING_DATASOURCE_USERNAME=admin_user
SPRING_JPA_HIBERNATE_DDL_AUTO=update
SPRING_SQL_INIT_MODE=always
```

### 🏥 Patient Service DB

```bash
docker run -d \
  --name patient-service-db \
  --network internal \
  --env-file ./patient-service-db.env \
  -p 5433:5432 \
  postgres:latest
```

`patient-service-db.env`

```env
POSTGRES_USER=admin_user
POSTGRES_PASSWORD=password
POSTGRES_DB=db
```

### 🏥 Patient Service (`patient-service`)

```env
BILLING_SERVICE_ADDRESS=billing-service
BILLING_SERVICE_GRPC_PORT=9001
SPRING_DATASOURCE_PASSWORD=password
SPRING_DATASOURCE_URL=jdbc:postgresql://patient-service-db:5432/db
SPRING_DATASOURCE_USERNAME=admin_user
SPRING_JPA_HIBERNATE_DDL_AUTO=update
SPRING_KAFKA_BOOTSTRAP_SERVERS=kafka:9092
SPRING_SQL_INIT_MODE=always
```

### 📊 Analytics Service (`analytics-service`)

```env
SPRING_KAFKA_BOOTSTRAP_SERVERS=kafka:9092
```

### 🌐 API Gateway (`api-gateway`)

```env
AUTH_SERVICE_URL=http://auth-service:4005
```

### Kafka

```env
KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://kafka:9092,EXTERNAL://localhost:9094
KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=0@kafka:9093
KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,EXTERNAL:PLAINTEXT,PLAINTEXT:PLAINTEXT KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094
KAFKA_CFG_NODE_ID=0
KAFKA_CFG_PROCESS_ROLES=controller,broker
```

---

## 📖 Notes

- Default credentials are for local development only.
- Replace dummy passwords and JWT secrets in production.
- Add volumes in Docker if you want persistent database storage.

### Due to my community edition I can not use Ultimate Edition features so here is the code for create a topic inside kafka container

#### First get inside the Kafka container

```bash
docker exec -it <your-kafka-container-id-or-name> bash
```

#### Then run the following command to create a topic named `patient-updates` with 3 partitions and a replication factor of 1

```bash
kafka-topics.sh --create \
  --topic patient-updates \
  --bootstrap-server localhost:9092 \
  --partitions 3 \
  --replication-factor 1
```