FROM maven:3.6.0-jdk-11-slim AS build
COPY src /home/app/src
COPY pom.xml /home/app
RUN mvn -f /home/app/pom.xml clean package
RUN ls -laR /home/app

#
# Package stage
#
FROM openjdk:11-jre-slim
COPY --from=build /home/app/target/CVE-2021-45105-1.0.0-SNAPSHOT.jar /usr/local/lib/demo.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","/usr/local/lib/demo.jar"]