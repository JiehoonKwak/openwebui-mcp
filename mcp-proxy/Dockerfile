FROM python:3.11-slim

WORKDIR /app

# Install system dependencies including Node.js 18.x
# Combine steps into fewer RUN layers to optimize image size
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

# Install Python dependencies using uv (after system dependencies are set)
# Using --no-cache-dir helps keep the layer smaller
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/


# The CMD instruction specifies the command to run when the container starts.
# It runs 'mcpo', listening on all interfaces (0.0.0.0) on port 8000.
# It expects the configuration file to be available at '/app/config/mcp-config.json'.
# This path matches the volume mount defined in docker-compose.yml.
CMD ["mcpo", "--host", "0.0.0.0", "--port", "8000", "--config", "/app/config/mcp-config.json"]
