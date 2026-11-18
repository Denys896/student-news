# ---- Stage 1: build the JAR (JDK + Maven) ----
FROM maven:3.10-eclipse-temurin-27 AS build
WORKDIR /app
# Dependencies first: this layer is rebuilt only when pom.xml changes.
COPY pom.xml .
RUN mvn -B -q dependency:go-offline
# Source second: a Java change invalidates only the layers from here down.
COPY src ./src
RUN mvn -B -q clean package -DskipTests

# ---- Stage 2: runtime (JRE only, Alpine) ----
FROM eclipse-temurin:27-jre-alpine
WORKDIR /app
# Unprivileged user with a fixed numeric id.
RUN addgroup -S -g 10001 app && adduser -S -u 10001 -G app app
# Upload directory, owned by that user so the app can write to it.
RUN mkdir -p /data/avatars && chown -R app:app /data/avatars
COPY --from=build /app/target/avatar-service.jar app.jar
USER 10001:10001
EXPOSE 8090
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
