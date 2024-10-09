FROM abhishekf5/maven-abhishek-docker-agent:v1

# Switch to root user to install packages
USER root

# Update package lists and install sudo
RUN apt-get update && \
    apt-get install -y sudo && \
    rm -rf /var/lib/apt/lists/*

# Create the jenkins user and add it to the sudo group
RUN useradd -m -s /bin/bash jenkins && \
    echo "jenkins:jenkins" | chpasswd && \
    usermod -aG sudo jenkins

# Change back to the jenkins user
USER jenkins

# Set the working directory (optional)
WORKDIR /home/jenkins