FROM abhishekf5/maven-abhishek-docker-agent:v1

# Switch to root to install packages
USER root

# Create jenkins user and group
RUN groupadd -g 1000 jenkins && \
    useradd -u 1000 -g jenkins -m jenkins

# Update package lists, install sudo, and install curl
RUN apt-get update && \
    apt-get install -y sudo curl && \
    rm -rf /var/lib/apt/lists/*

# Install Trivy using the provided command
RUN curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin

# Verify Trivy installation
RUN /usr/local/bin/trivy --version

# Ensure jenkins user can execute trivy
RUN chown jenkins:jenkins /usr/local/bin/trivy && \
    chmod +x /usr/local/bin/trivy

# Change back to jenkins user
USER jenkins

# Update PATH for jenkins user
ENV PATH="/usr/local/bin:${PATH}"