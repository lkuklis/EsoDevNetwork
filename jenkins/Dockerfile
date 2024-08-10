# Use an official Jenkins image as a parent image
FROM jenkins/jenkins:lts

# Switch to root user for installing packages
USER root

# Install .NET Core SDK and other prerequisites
RUN apt-get update && \
    apt-get install -y apt-transport-https wget && \
    wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb && \
    dpkg -i packages-microsoft-prod.deb && \
    apt-get update && \
    apt-get install -y dotnet-sdk-6.0

# Install Docker CLI
RUN apt-get update && \
    apt-get install -y docker.io

# Add NodeSource repository for Node.js 20 and install it
RUN apt-get update && \
    apt-get install -y ca-certificates curl gnupg && \
    mkdir -p /etc/apt/keyrings && \
    curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg && \
    echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_20.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list && \
    apt-get update && \
    apt-get install -y nodejs

# Confirm that Node.js and npm were installed
RUN node -v && \
    npm -v


RUN apt-get install -y zip jq

RUN npm install -g badlinks
# Install Heroku CLI
RUN curl https://cli-assets.heroku.com/install.sh | sh
# Switch back to Jenkins user
USER jenkins

# Pre-install Jenkins plugins (Optional, modify as needed)
RUN jenkins-plugin-cli --plugins "docker pipeline:1.25 workflow-aggregator:2.6 git:4.11.5"

