# Step 1: Start with a base OS
FROM ubuntu:20.04

# Step 2: Install dependencies (update, install curl, git, etc.)
RUN apt-get update && \
    apt-get install -y curl git apt-transport-https ca-certificates software-properties-common gnupg2 lsb-release

# Step 3: Set Java version and download OpenJDK
ENV JAVA_VERSION=jdk-11.0.11+9
RUN ARCH="$(dpkg --print-architecture)" && \
    case "${ARCH}" in \
    amd64|x86_64) BINARY_URL='https://github.com/AdoptOpenJDK/openjdk11-binaries/releases/download/jdk-11.0.11+9/OpenJDK11U-jdk_x64_linux_hotspot_11.0.11_9.tar.gz'; ESUM='e99b98f851541202ab64401594901e583b764e368814320eba442095251e78cb'; ;; \
    *) echo "Unsupported arch: ${ARCH}"; exit 1 ;; \
    esac && \
    curl -LfsSo /tmp/openjdk.tar.gz ${BINARY_URL} && \
    echo "${ESUM} */tmp/openjdk.tar.gz" | sha256sum -c - && \
    mkdir -p /opt/java/openjdk && \
    tar -xf /tmp/openjdk.tar.gz -C /opt/java/openjdk --strip-components=1 && \
    rm -rf /tmp/openjdk.tar.gz

# Set Java environment variables
ENV JAVA_HOME=/opt/java/openjdk
ENV PATH="${JAVA_HOME}/bin:${PATH}"

# Step 4: Set Maven version and download
ARG MAVEN_VERSION=3.8.1
ARG SHA=0ec48eb515d93f8515d4abe465570dfded6fa13a3ceb9aab8031428442d9912ec20f066b2afbf56964ffe1ceb56f80321b50db73cf77a0e2445ad0211fb8e38d
ARG BASE_URL=https://apache.osuosl.org/maven/maven-3/${MAVEN_VERSION}/binaries

RUN mkdir -p /usr/share/maven /usr/share/maven/ref && \
    curl -fsSL -o /tmp/apache-maven.tar.gz ${BASE_URL}/apache-maven-${MAVEN_VERSION}-bin.tar.gz && \
    echo "${SHA} /tmp/apache-maven.tar.gz" | sha512sum -c - && \
    tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 && \
    rm -f /tmp/apache-maven.tar.gz && \
    ln -s /usr/share/maven/bin/mvn /usr/bin/mvn

# Set Maven environment variables
ENV MAVEN_HOME=/usr/share/maven
ENV MAVEN_CONFIG="/root/.m2"

# Step 5: Install Docker
RUN apt-get install -y docker.io

# Step 6: Define entry point for Maven
COPY mvn-entrypoint.sh /usr/local/bin/mvn-entrypoint.sh
RUN chmod +x /usr/local/bin/mvn-entrypoint.sh

ENTRYPOINT ["/usr/local/bin/mvn-entrypoint.sh"]

CMD ["mvn"]