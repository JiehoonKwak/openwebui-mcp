FROM python:3.11-slim

WORKDIR /app

RUN apt-get update && \
    # Install curl and gnupg needed for adding NodeSource repository
    apt-get install -y --no-install-recommends \
    curl \
    gnupg \
    # Add NodeSource repository for Node.js 18.x and run the setup script
    && curl -fsSL https://deb.nodesource.com/setup_18.x | bash - \
    # Install Node.js itself
    && apt-get install -y --no-install-recommends \
    nodejs \
    # Clean up APT cache and lists to reduce final image size
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install mcpo using pip directly
RUN pip install --no-cache-dir mcpo

# Create config directory
RUN mkdir -p /app/config

# Run mcpo directly
CMD ["python", "-m", "mcpo", "--host", "0.0.0.0", "--port", "8001", "--config", "/app/config/mcp-config.json"]
