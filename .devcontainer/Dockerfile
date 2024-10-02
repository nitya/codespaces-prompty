# Use specific version of Python as base
FROM mcr.microsoft.com/devcontainers/python:1-3.12-bullseye

# Combine multiple Python system updates in one RUN command to reduce layers. Clean up apt cache to reduce image size
RUN sudo apt-get update && sudo apt-get install -y \
    gcc \
    cmake \
    pkg-config \
    libdbus-1-dev \
    libglib2.0-dev \
 && python -m pip install --upgrade pip \
 && sudo apt-get clean \
 && sudo rm -rf /var/lib/apt/lists/*

# Install dependencies from requirements.txt and other other pre-defined packages in same layer to optimize image size and build time 
COPY requirements.txt .
RUN pip install -r requirements.txt \
    && pip install keyrings.alt dbus-python ipython ipykernel mkdocs-material mkdocs-rss-plugin

# Configure IPython kernel
RUN ipython kernel install --name "python3" --user