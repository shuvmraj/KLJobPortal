# Multi-stage build for Spring Boot application
FROM maven:3.9-eclipse-temurin-21 AS builder
WORKDIR /build
COPY pom.xml .
RUN mvn dependency:go-offline -q || true
COPY . .
RUN mvn clean package -q -DskipTests

FROM eclipse-temurin:21-jre
WORKDIR /app
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
COPY --from=builder /build/target/*.jar app.jar
USER appuser
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD java -cp app.jar org.springframework.boot.loader.JarLauncher || exit 1
CMD ["java", "-jar", "app.jar"]
