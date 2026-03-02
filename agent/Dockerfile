FROM docker:27-cli

# Tools + Java runtime for Jenkins agent.jar
RUN apk add --no-cache \
    bash curl git openssh-client \
    openjdk17-jre \
    ca-certificates

# Create jenkins user (UID 1000 matches common Jenkins expectations)
RUN addgroup -g 1000 jenkins && adduser -D -u 1000 -G jenkins -h /home/jenkins jenkins

USER jenkins
WORKDIR /home/jenkins

# Download Jenkins remoting agent.jar
RUN curl -fsSL -o agent.jar https://repo.jenkins-ci.org/public/org/jenkins-ci/main/remoting/3299.v0d0d06908537/remoting-3299.v0d0d06908537.jar

# Connect using env vars provided in docker-compose.yml
ENTRYPOINT ["sh","-lc","java -jar agent.jar -url \"$JENKINS_URL\" -secret \"$JENKINS_SECRET\" -name \"$JENKINS_AGENT_NAME\" -workDir \"$JENKINS_AGENT_WORKDIR\""]