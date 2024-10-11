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

# Configure sudoers to allow jenkins to use sudo without password
RUN echo "jenkins ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

# Ensure PATH includes common binary locations
ENV PATH=$PATH:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# Change back to the jenkins user
USER jenkins

# Set the working directory
WORKDIR /home/jenkins

# Verify sudo installation (optional, can be removed in production)
RUN sudo --version