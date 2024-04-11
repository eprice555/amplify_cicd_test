# Update 'VARIANT' to pick a default version of Node.js: 16, 14, 12.
# Append -bullseye to pin to local arm64/Apple Silicon.
# Append -buster to pin to Debian.
# ARG VARIANT=16-buster

# FROM mcr.microsoft.com/vscode/devcontainers/javascript-node:0-${VARIANT}

# Use an official Node.js image as a parent image
FROM node:14-slim

# Install system packages required by Vault and other tools.
RUN apt-get update && apt-get install -y --no-install-recommends \
    wget \
    unzip \
    ca-certificates \
    python3 \
    python3-pip \
    && rm -rf /var/lib/apt/lists/*

# Install AWS CLI (useful for Amplify and general AWS operations)
RUN pip3 install awscli

# Install Vault CLI
RUN wget https://releases.hashicorp.com/vault/1.7.3/vault_1.7.3_linux_amd64.zip \
    && unzip vault_1.7.3_linux_amd64.zip \
    && mv vault /usr/local/bin/ \
    && rm vault_1.7.3_linux_amd64.zip

# Set the working directory in the container
WORKDIR /workspace
