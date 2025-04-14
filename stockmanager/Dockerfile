FROM maven:3.9.6-eclipse-temurin-24 AS build
WORKDIR /app

COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src

RUN mvn clean package -DskipTests

FROM eclipse-temurin:24-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar

EXPOSE 8030
CMD ["java", "-jar", "/app.jar"] 
