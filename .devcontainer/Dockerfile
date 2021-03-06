FROM debian:buster-slim

# Latest version of Terraform may be found at https://www.terraform.io/downloads.html
ARG TERRAFORM_VERSION=0.14.7

# Latest version of Terrform Linter may be found at https://github.com/terraform-linters/tflint/releases
ARG TFLINT_VERSION=0.24.1

# Latest version of helm may be found at https://github.com/helm/helm/releases
ARG HELM_VERSION=3.5.2

# Latest version of dotnet core SDK
ARG DOTNET_VERSION=5.0

# Azure Functions CLI may be found at https://github.com/Azure/azure-functions-core-tools/releases
ARG AZFUNC_CLI_VERSION=3.0.3233


WORKDIR /tmp

# Create a temp directory for downloads
RUN mkdir -p /tmp/downloads

# --------------------------------
# Install Dependencies
# --------------------------------

RUN apt-get update \
    && apt-get install -y \
        apt-transport-https \
        ca-certificates \
        cargo \
        curl \
        git \
        gnupg2 \
        gnupg-agent \
        iproute2 \
        libssl-dev \
        libffi-dev \
        lsb-release \
        openssh-client \
        procps \
        python3-dev \
        python3-pip \
        software-properties-common \
        squashfs-tools \
        unzip \
        wget \
    && rm -rf /var/lib/apt/lists/*

# --------------------------------
# Add Keys and Repos
# --------------------------------

# Docker
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
RUN add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"

# Microsoft
RUN curl -fsSL https://packages.microsoft.com/keys/microsoft.asc | apt-key add -
RUN apt-add-repository https://packages.microsoft.com/debian/10/prod
RUN add-apt-repository https://packages.microsoft.com/repos/azure-cli/

# Update package list
RUN apt-get update

# --------------------------------
# Install Packages
# --------------------------------

# Install Rust
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | bash -s -- -y

# Install Docker
RUN apt-get install -y docker-ce docker-ce-cli containerd.io

# # Install IoTEdgeDev
RUN pip3 install --upgrade iotedgehubdev

# Install Terraform, tflint, and graphviz
RUN curl -sSL -o /tmp/downloads/terraform.zip https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip \
    && unzip /tmp/downloads/terraform.zip \
    && mv terraform /usr/local/bin \
    && curl -sSL -o /tmp/downloads/tflint.zip https://github.com/terraform-linters/tflint/releases/download/v${TFLINT_VERSION}/tflint_linux_amd64.zip \
    && unzip /tmp/downloads/tflint.zip \
    && mv tflint /usr/local/bin \
    && cd ~ \ 
    && apt-get install -y graphviz

# Install Kubectl
RUN echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list \ 
    && curl -sL https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - 2>/dev/null \
    && apt-get update \
    && apt-get install -y kubectl

# Install Helm
RUN curl -sSL -o /tmp/downloads/helm.tar.gz https://get.helm.sh/helm-v${HELM_VERSION}-linux-amd64.tar.gz \
    && mkdir -p /tmp/downloads/helm \
    && tar -C /tmp/downloads/helm -zxvf /tmp/downloads/helm.tar.gz \
    && mv /tmp/downloads/helm/linux-amd64/helm /usr/local/bin \
    && helm repo add stable https://charts.helm.sh/stable

# Install .NET Core
ENV \
    # Enable detection of running in a container
    DOTNET_RUNNING_IN_CONTAINER=true \
    # Enable correct mode for dotnet watch (only mode supported in a container)
    DOTNET_USE_POLLING_FILE_WATCHER=true \
    # Skip extraction of XML docs - generally not useful within an image/container - helps performance
    NUGET_XMLDOC_MODE=skip

RUN echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-debian-buster-prod buster main" > /etc/apt/sources.list.d/microsoft.list \ 
    && apt-get update \
    && apt-get install -y dotnet-sdk-${DOTNET_VERSION}

# Install Azure Functions Core Tools v3
RUN curl -s -L https://github.com/Azure/azure-functions-core-tools/releases/download/${AZFUNC_CLI_VERSION}/Azure.Functions.Cli.linux-x64.${AZFUNC_CLI_VERSION}.zip -o /tmp/downloads/azfunc.zip \
    && mkdir -p /tmp/downloads/azfunc \
    && unzip -qq -d /tmp/downloads/azfunc /tmp/downloads/azfunc.zip \
    && mv /tmp/downloads/azfunc /usr/local/bin/azfunc \
    && cd /usr/local/bin/azfunc \
    && chmod +x func \
    && ln -s /usr/local/bin/azfunc/func /usr/bin/func

# Install the Azure CLI
RUN echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $(lsb_release -cs) main" > /etc/apt/sources.list.d/azure-cli.list \
    && curl -sL https://packages.microsoft.com/keys/microsoft.asc | apt-key add - 2>/dev/null \
    && apt-get update \
    && apt-get install -y azure-cli
