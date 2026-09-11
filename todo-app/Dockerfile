# Multi-stage build
# Stage 1: Build the application
FROM maven:3.9.6-eclipse-temurin-17 AS builder

WORKDIR /build

# Copy the pom.xml and download dependencies
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Copy the source code
COPY src ./src

# Build the application
RUN mvn clean package -DskipTests

# Stage 2: Create the runtime image
FROM eclipse-temurin:17-jre-alpine

# Remove unneeded package index/cache and any pre-installed shells/tools we don't need,
# keeping the image minimal. wget (used by HEALTHCHECK) ships via busybox already.
RUN apk --no-cache upgrade

# Create a dedicated non-root, non-login user/group to run the application.
RUN addgroup -S appgroup && adduser -S -G appgroup -H -D appuser

WORKDIR /app

# Copy the built JAR from the builder stage, owned by the non-root user.
COPY --from=builder --chown=appuser:appgroup /build/target/todo-app-1.0.0.jar app.jar

# Drop privileges before running the application.
USER appuser:appgroup

# Expose the port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --quiet --tries=1 --spider http://localhost:8080/api/todos/health || exit 1

# Run the application
ENTRYPOINT ["java", "-jar", "app.jar"]
