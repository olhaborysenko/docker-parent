ARG GIT_REPO_DIR=DemoRepo
ARG GIT_URL=https://github.com/olhaborysenko/docker-parent.git

FROM amazon/aws-cli:latest AS fetcher
ARG GIT_REPO_DIR
ARG GIT_URL

RUN yum install -y git && \
    mkdir -p /home/demo

WORKDIR /home/demo

RUN git clone ${GIT_URL} ${GIT_REPO_DIR}

# --------- Builder Layer ---------
FROM alpine:3.13.0@sha256:d0710affa17fad5f466a70159cc458227bd25d4afb39514ef662ead3e6c99515 AS builder

ENV JAVA_HOME=/opt/jdk
ENV PATH=$JAVA_HOME/bin:$PATH
ARG GIT_REPO_DIR

# Download and extract Amazon Corretto JDK
ADD https://corretto.aws/downloads/latest/amazon-corretto-15-x64-alpine-jdk.tar.gz /tmp/corretto.tar.gz

RUN mkdir -p $JAVA_HOME && \
    tar --extract --file /tmp/corretto.tar.gz --directory "$JAVA_HOME" --strip-components 1 && \
    rm /tmp/corretto.tar.gz

RUN ["jlink", "--compress=2", \
    "--module-path", "/opt/jdk/jmods/", \
    "--add-modules", "java.base,java.logging", \
    "--no-header-files", "--no-man-pages", \
    "--output", "/netty-runtime"]

RUN apk update && \
    apk add --no-cache maven

# Copy cloned repo from fetcher stage
COPY --from=fetcher /home/demo/${GIT_REPO_DIR} /home/demo/${GIT_REPO_DIR}

WORKDIR /home/demo/${GIT_REPO_DIR}/java

RUN mvn clean package

# --------- Final Runtime Image ---------
FROM alpine:3.13.0@sha256:d0710affa17fad5f466a70159cc458227bd25d4afb39514ef662ead3e6c99515

ENV PATH=$PATH:/opt/jdk
ARG GIT_REPO_DIR

# Create a non-root user
RUN addgroup -S demogroup && adduser -S demouser -G demogroup
USER demouser

# Copy JDK and built JAR
COPY --from=builder /netty-runtime /opt/jdk
COPY --from=builder /home/demo/${GIT_REPO_DIR}/java/target/netty-example-1.0-SNAPSHOT.jar /opt/app/

EXPOSE 8090
ENTRYPOINT ["java", "-showversion", "-jar", "/opt/app/netty-example-1.0-SNAPSHOT.jar"]
