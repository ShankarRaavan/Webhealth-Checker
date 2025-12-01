# cicd/docker/Dockerfile
FROM python:3.10-slim

# Avoid interactive prompts
ARG DEBIAN_FRONTEND=noninteractive
ENV DEBIAN_FRONTEND=${DEBIAN_FRONTEND}

# Install basic deps (including gnupg for key handling).
# Keep this in one RUN to reduce image layers.
RUN apt-get update \
  && apt-get install -y --no-install-recommends \
     ca-certificates apt-transport-https gnupg2 wget unzip curl \
     fonts-liberation libnss3 libxss1 libgtk-3-0 libx11-xcb1 libasound2 \
     xvfb procps lsb-release \
  && rm -rf /var/lib/apt/lists/*

# Add Google's signing key the modern way (dearmor) but only prepare the path.
# We'll add the repo only for amd64 later.
RUN mkdir -p /usr/share/keyrings

# Install Chrome (amd64) or Chromium (arm64) depending on architecture.
# - For amd64: add Google repo using signed-by and install google-chrome-stable.
# - For other arches (arm64): install chromium from Debian repos.
# Install Chromium and chromedriver on all architectures for consistency
RUN apt-get update \
  && apt-get install -y --no-install-recommends chromium chromium-driver \
  && if [ ! -e /usr/bin/chromedriver ]; then ln -sf $(command -v chromium-driver || command -v chromedriver) /usr/bin/chromedriver; fi \
  && rm -rf /var/lib/apt/lists/*

# Optional: if your app uses chromedriver via apt on the distro, you can try install:
# (This package name may not exist on all slim images — prefer a Python chromedriver package)
# RUN apt-get update && apt-get install -y chromium-driver || true

WORKDIR /app

# Copy source code and requirements
COPY src/ ./src/
COPY src/requirements.txt ./requirements.txt

# Install Python packages (use pip cache-free)
RUN pip install --no-cache-dir -r /app/requirements.txt

# Create non-root user and drop privileges
RUN useradd --create-home --shell /bin/bash appuser \
  && chown -R appuser:appuser /app

USER appuser
ENV HOME=/home/appuser
ENV PATH=$HOME/.local/bin:$PATH

# Expose any ports if needed
# EXPOSE 8080

# Default command
CMD ["python", "src/global/global_monitor.py"]
