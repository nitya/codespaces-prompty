
# Use a specific version of Python image to ensure consistency across builds
FROM mcr.microsoft.com/devcontainers/python:3.12-bullseye

# Combine multiple Python system updates in one RUN command to reduce layers. 
# Clean up apt cache to reduce image size
RUN sudo apt-get update && sudo apt-get install -y \
    gcc \
    cmake \
    pkg-config \
    libdbus-1-dev \
    libglib2.0-dev \
 && python -m pip install --upgrade pip \
 && sudo apt-get clean \
 && sudo rm -rf /var/lib/apt/lists/*

# Copy requirements.txt and install the Python packages
# Install keyring-related and IPython packages in the same layer to reduce the image size and build time
COPY requirements.txt .
RUN pip install -r requirements.txt \    
    && pip install keyrings.alt dbus-python ipython ipykernel mkdocs-material

# Configure the IPython kernel
RUN ipython kernel install --name "python3" --user
