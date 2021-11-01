FROM maven:3.8.3-jdk-8-slim

WORKDIR /app 

COPY . .

RUN mvn package -DskipTests && \
    mv target/worker-jar-with-dependencies.jar /run/worker.jar && \
    rm -rf *

CMD java -jar /run/worker.jar
