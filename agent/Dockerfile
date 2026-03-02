FROM jenkins/inbound-agent:latest-jdk17

USER root

# Install docker CLI (and basic tools)
RUN apt-get update && \
    apt-get install -y --no-install-recommends docker.io ca-certificates curl && \
    rm -rf /var/lib/apt/lists/*

# (Optional) nice-to-have: show versions quickly
RUN docker --version || true

USER jenkins