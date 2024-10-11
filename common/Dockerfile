FROM abhishekf5/maven-abhishek-docker-agent:v1

USER root
RUN apt-get update && \
    apt-get install -y sudo && \
    rm -rf /var/lib/apt/lists/*

USER jenkins  # Change back to the jenkins user
