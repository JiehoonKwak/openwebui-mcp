# OpenWebUI Docker Compose Implementation Plan

## Overview
This document outlines the implementation plan for hosting OpenWebUI on a Synology DS920+ NAS using Docker Compose and Portainer.

## Directory Structure on Synology NAS

```
/volume1/docker/openwebui/
├── data/           # OpenWebUI persistent data
├── config/         # Configuration files
│   └── mcp-config.json  # Your custom MCP configuration file
```

## Docker Compose Configuration

### Architecture Diagram

```mermaid
graph TD
    User[User] -->|Port 3000| OpenWebUI[OpenWebUI Container]
    User -->|Port 8001| MCPProxy[MCP Proxy Container]
    OpenWebUI -->|Internal Network| MCPProxy
    OpenWebUI -->|Volume Mount| OpenWebUIData[/volume1/docker/openwebui/data]
    MCPProxy -->|Volume Mount| MCPConfig[/volume1/docker/openwebui/config/mcp-config.json]
    subgraph "Docker Network"
        OpenWebUI
        MCPProxy
    end
    subgraph "Synology NAS"
        OpenWebUIData
        MCPConfig
    end
```

## Docker Compose File

Here's the docker-compose.yml file that should be created:

```yaml
version: '3.8'

services:
  openwebui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    ports:
      - "3000:8080"
    volumes:
      - /volume1/docker/openwebui/data:/app/backend/data
    extra_hosts:
      - "host.docker.internal:host-gateway"
    restart: always
    networks:
      - openwebui_network

  mcp-proxy:
    build:
      context: ./mcp-proxy
      dockerfile: Dockerfile
    container_name: mcp-proxy
    ports:
      - "8001:8001"
    volumes:
      - /volume1/docker/openwebui/config/mcp-config.json:/app/config/mcp-config.json
    restart: always
    profiles:
      - tools
    networks:
      - openwebui_network

networks:
  openwebui_network:
    driver: bridge
```

## Implementation Steps

1. **Prepare the Synology NAS**:
   - Create the directory structure:
     ```
     mkdir -p /volume1/docker/openwebui/data
     mkdir -p /volume1/docker/openwebui/config
     ```
   - Place your custom mcp-config.json file in `/volume1/docker/openwebui/config/`

2. **Prepare the Project Files**:
   - Create the docker-compose.yml file with the content above
   - Ensure the mcp-proxy/Dockerfile exists with the correct content

3. **Deploy with Portainer**:
   - In Portainer, go to Stacks
   - Add a new stack
   - Upload the docker-compose.yml file or paste its contents
   - Deploy the stack

4. **Verify the Deployment**:
   - Access OpenWebUI at http://your-nas-ip:3000
   - Verify the MCP proxy is running at http://your-nas-ip:8001

## Maintenance

- **Backups**: Regularly backup the `/volume1/docker/openwebui/data` directory
- **Updates**: Update the containers by pulling new images and redeploying the stack
- **Logs**: Monitor logs through Portainer or using `docker logs open-webui` and `docker logs mcp-proxy`

## Troubleshooting

- If services fail to start, check the logs for errors
- Ensure the mcp-config.json file is correctly formatted and accessible
- Verify that ports 3000 and 8001 are not being used by other services