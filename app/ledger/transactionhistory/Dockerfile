# Use an official Maven image to build the application
FROM maven:3.8.6-openjdk-11 AS build

# Set the working directory inside the container
WORKDIR /build

# Copy the pom.xml and download dependencies
COPY pom.xml .
RUN mvn dependency:go-offline

# Copy the source code and build the application
COPY src ./src
RUN mvn package -DskipTests

# Use an official OpenJDK runtime as a base image
FROM openjdk:11-jre-slim

# Set the working directory inside the container
WORKDIR /app

# Copy the jar file from the build stage
COPY --from=build /build/target/transactionhistory.jar ./transactionhistory.jar

# Expose the necessary port
EXPOSE 8080

# Run the application
CMD ["java", "-jar", "balancereader.jar"]
