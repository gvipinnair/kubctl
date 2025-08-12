# Use official OpenJDK image as base
FROM openjdk:17-jdk-slim

# Copy the jar file (after build) to container
ARG JAR_FILE=target/demo-0.0.1-SNAPSHOT.jar
COPY ${JAR_FILE} app.jar

# Run the jar file
ENTRYPOINT ["java", "-jar", "/app.jar"]