FROM abhishekf5/maven-abhishek-docker-agent:v1

# Switch to root to install packages
USER root

# Update package lists, install sudo, and install Trivy
RUN apt-get update && \
    apt-get install -y sudo curl && \
    rm -rf /var/lib/apt/lists/* && \
    # Install Trivy using the provided command
    curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin && \
    # Verify Trivy installation
    echo "Trivy version:" && trivy --version && \
    echo "Trivy installation successful."
RUN chmod +x /usr/local/bin/trivy

# Set the PATH environment variable
ENV PATH="/usr/local/bin:${PATH}"

# Change back to jenkins user (optional, depending on your setup)
# USER jenkins