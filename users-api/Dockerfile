# --- Stage 1: build ---
FROM maven:3.6-jdk-8 AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline -B
COPY src ./src
RUN mvn clean package -DskipTests

# --- Stage 2: runtime ---
FROM eclipse-temurin:8-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/users-api-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8083
ENTRYPOINT ["java", "-jar", "app.jar"]