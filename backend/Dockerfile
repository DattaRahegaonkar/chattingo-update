# ----------------------- Stage 1 --------------------------

# base image for building the Spring Boot backend
FROM maven:3.8.3-openjdk-17-slim AS builder

# set working directory for the app
WORKDIR /src

# copy the pom.xml into the container working directory
COPY pom.xml .

# copy the rest of the app source code into the container working directory
COPY . /src

# install app dependencies and build the Spring Boot app
RUN mvn clean install -DskipTests=true

# ----------------------- Stage 2 --------------------------

# base image for running the Spring Boot backend
FROM eclipse-temurin:17-jre-alpine

# set working directory for the app
WORKDIR /src

# copy the built jar file from the builder stage to the current stage
COPY --from=builder /src/target/*.jar /src/app.jar

EXPOSE 8081

ENTRYPOINT ["java", "-jar", "/src/app.jar"]