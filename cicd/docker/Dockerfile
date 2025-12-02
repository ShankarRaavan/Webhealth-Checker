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
# Install Chrome (amd64) or Chromium (arm64) depending on architecture.
RUN ARCH="$(dpkg --print-architecture)" \
  && if [ "$ARCH" = "amd64" ]; then \
       # import and dearmor Google's signing key
       wget -qO- https://dl.google.com/linux/linux_signing_key.pub \
         | gpg --dearmor > /usr/share/keyrings/google-linux-signing-keyring.gpg \
       && echo "deb [arch=amd64 signed-by=/usr/share/keyrings/google-linux-signing-keyring.gpg] http://dl.google.com/linux/chrome/deb/ stable main" \
         > /etc/apt/sources.list.d/google-chrome.list \
       && apt-get update \
       && apt-get install -y --no-install-recommends google-chrome-stable \
       && rm -rf /var/lib/apt/lists/*; \
     else \
       apt-get update \
       && apt-get install -y --no-install-recommends chromium chromium-driver || true \
       && rm -rf /var/lib/apt/lists/*; \
     fi

# Optional: if your app uses chromedriver via apt on the distro, you can try install:
# (This package name may not exist on all slim images — prefer a Python chromedriver package)
# RUN apt-get update && apt-get install -y chromium-driver || true

# Set environment variables for ARM/System drivers
ENV CHROME_BINARY_PATH=/usr/bin/chromium \
    CHROMEDRIVER_PATH=/usr/bin/chromedriver \
    HEADLESS=true

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
