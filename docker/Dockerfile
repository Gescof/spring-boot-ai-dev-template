FROM eclipse-temurin:21-jdk AS builder
WORKDIR /app

# Cache dependency resolution separately from source compilation
COPY .mvn/ .mvn/
COPY mvnw pom.xml ./
RUN ./mvnw dependency:go-offline -q

COPY src/ src/
RUN ./mvnw clean package -DskipTests -q

FROM eclipse-temurin:21-jre AS runtime
WORKDIR /app

COPY --from=builder /app/target/*.jar app.jar

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
