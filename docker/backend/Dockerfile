# Build context: repository root (needs ./backend and ./docker/backend)

# ---- Build stage: Maven + JDK 17 ----
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /src
# Resolve dependencies first so this layer is cached until pom.xml changes
COPY backend/pom.xml .
RUN mvn -B -q dependency:go-offline
COPY backend/src ./src
# Swap in the env-driven configuration: secrets never enter the image
COPY docker/backend/application.properties ./src/main/resources/application.properties
RUN mvn -B -q package -DskipTests && cp target/*.jar /app.jar

# ---- Runtime stage: JRE only, non-root ----
FROM eclipse-temurin:17-jre-jammy
RUN useradd --system --uid 10001 app && mkdir -p /app/uploads && chown app /app/uploads
WORKDIR /app
COPY --from=build /app.jar app.jar
USER 10001
EXPOSE 8084
ENV JAVA_OPTS="-XX:MaxRAMPercentage=75"
ENTRYPOINT ["sh", "-c", "exec java $JAVA_OPTS -jar app.jar"]
