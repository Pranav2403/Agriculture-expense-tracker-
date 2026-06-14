# Java Microservice Setup Guide

This guide explains how to set up and integrate the Java microservice with your AgriTrack application.

## Quick Start

### Local Development

1. **Navigate to Java service directory:**
   ```bash
   cd java-service
   ```

2. **Build the project:**
   ```bash
   mvn clean package
   ```

3. **Run the service:**
   ```bash
   java -jar target/analytics-service-1.0.0.jar
   ```

   The service will start on `http://localhost:8080/api`

### Environment Variables

Create a `.env` file in the `java-service` directory:

```
DATABASE_URL=postgresql://user:password@host:5432/database
DB_USER=your_postgres_user
DB_PASSWORD=your_postgres_password
```

For Supabase, use:
```
DATABASE_URL=postgresql://postgres.xxxxx:password@db.xxxxx.supabase.co:5432/postgres
DB_USER=postgres
DB_PASSWORD=your_supabase_password
```

## API Endpoints

### 1. Monthly Analytics
**Endpoint:** `GET /api/analytics/monthly`

**Headers:**
```
X-User-ID: user-id-here
```

**Response:**
```json
{
  "months": [
    {
      "month": "2024-10",
      "income": 5000.00,
      "expenses": 2000.00,
      "profit": 3000.00
    }
  ],
  "totalIncome": 30000.00,
  "totalExpenses": 12000.00,
  "netProfit": 18000.00
}
```

### 2. Crop Recommendations
**Endpoint:** `POST /api/crops/recommend`

**Headers:**
```
X-User-ID: user-id-here
Content-Type: application/json
```

**Request Body:**
```json
{
  "location": "Maharashtra",
  "soilType": "loamy",
  "season": "rabi",
  "farmSize": 5.0
}
```

**Response:**
```json
{
  "recommendations": [
    {
      "name": "Tomato",
      "suitability": 0.95,
      "waterRequirement": "Moderate",
      "growingSeason": "Oct-Mar",
      "expectedYield": "300-350 quintals/acre"
    }
  ],
  "soilAnalysis": "Loamy soil is the most fertile...",
  "seasonalTips": "Winter crops thrive in cool temperatures..."
}
```

## Integration with Next.js

The Next.js application automatically proxies requests to the Java microservice:

- **`/api/analytics/monthly`** → Java service `/analytics/monthly`
- **`/api/crops/recommend`** → Java service `/crops/recommend`

If the Java service is unavailable, the application falls back to direct Supabase queries.

### Setting Java Service URL

In your Vercel environment variables, set:
```
JAVA_SERVICE_URL=https://your-java-service-url:8080/api
```

For local development, it defaults to `http://localhost:8080/api`

## Production Deployment

### Docker

1. **Create a Dockerfile in `java-service` directory:**
   ```dockerfile
   FROM maven:3.9-eclipse-temurin-17 AS build
   WORKDIR /build
   COPY . .
   RUN mvn clean package -DskipTests

   FROM eclipse-temurin:17-jdk-alpine
   COPY --from=build /build/target/analytics-service-1.0.0.jar /app/service.jar
   EXPOSE 8080
   ENTRYPOINT ["java", "-jar", "/app/service.jar"]
   ```

2. **Build and run:**
   ```bash
   docker build -t agritrack-java-service .
   docker run -p 8080:8080 -e DATABASE_URL=postgresql://... agritrack-java-service
   ```

### Cloud Deployment

#### Heroku
```bash
heroku create your-app-name
heroku config:set DATABASE_URL=postgresql://...
git push heroku main
```

#### AWS EC2
1. Launch an EC2 instance with Java 17
2. Clone the repository
3. Build and run with: `mvn spring-boot:run`
4. Use Nginx as reverse proxy

#### Google Cloud Run
```bash
gcloud run deploy agritrack-service \
  --source . \
  --platform managed \
  --region us-central1 \
  --set-env-vars DATABASE_URL=postgresql://...
```

## Testing

### Local Testing

```bash
# Start Java service
cd java-service
mvn spring-boot:run

# In another terminal, test endpoints
curl -H "X-User-ID: test-user-id" http://localhost:8080/api/analytics/monthly

curl -H "X-User-ID: test-user-id" -H "Content-Type: application/json" \
  -d '{"location":"Maharashtra","soilType":"loamy","season":"rabi","farmSize":5}' \
  -X POST http://localhost:8080/api/crops/recommend
```

## Architecture

- **Spring Boot 3.3.0** - Web framework
- **Spring Data JPA** - Database access
- **PostgreSQL** - Data persistence
- **Lombok** - Boilerplate reduction
- **Maven** - Build management

## Troubleshooting

### Connection Issues
- Verify `DATABASE_URL` environment variable is set correctly
- Ensure PostgreSQL database is accessible
- Check firewall rules for port 5432 (PostgreSQL)

### Missing Data
- Run database migration scripts from the main Next.js project
- Verify user IDs match between services

### Performance Issues
- Add database indexes on `user_id` columns
- Consider caching for frequently accessed data
- Monitor slow queries in PostgreSQL logs

## Next Steps

1. Add more complex business logic to the Java service
2. Implement caching with Redis
3. Add database connection pooling
4. Create comprehensive unit and integration tests
5. Set up CI/CD pipeline for automated deployments
