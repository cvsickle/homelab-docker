FROM mcr.microsoft.com/devcontainers/base:resolute

ARG TARGETARCH=amd64
ARG SOPS_VERSION=3.13.3

RUN apt-get update && apt-get install -y \
    curl \
    wget \
    tar \
    unzip \
    ca-certificates \
    gnupg \
    bash-completion \
    && rm -rf /var/lib/apt/lists/*

RUN curl -fsSL \
    "https://github.com/getsops/sops/releases/download/v${SOPS_VERSION}/sops-v${SOPS_VERSION}.linux.${TARGETARCH}" \
    -o /usr/local/bin/sops \
    && chmod +x /usr/local/bin/sops