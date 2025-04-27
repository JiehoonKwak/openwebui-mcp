# OpenWebUI with MCP for Synology NAS

This repository contains the necessary configuration files to deploy OpenWebUI with Model Context Protocol (MCP) support on a Synology NAS using Docker and Portainer.

## Overview

- **OpenWebUI**: A web interface for Ollama and various LLM APIs
- **MCP Proxy**: A Model Context Protocol server that provides tool functionality

## Prerequisites

- Synology NAS with Docker and Portainer installed
- Git installed on your local machine (for cloning this repository)
- Basic knowledge of Docker and Portainer

## Deployment Instructions

### 1. Clone and Prepare the Repository

1. Clone this repository to your local machine:
   ```bash
   git clone https://github.com/yourusername/openwebui-mcp.git
   cd openwebui-mcp
   ```

2. Customize the configuration files if needed:
   - Modify `.env` to change ports or other settings
   - Create your own `mcp-config.json` or use the sample provided

3. Commit any changes to your own Git repository if desired.

### 2. Prepare Your Synology NAS Using Portainer

1. Log in to your Portainer instance on your Synology NAS.

2. Create the required directories using Portainer's File Browser:
   - Navigate to **Volumes** in the left sidebar
   - Click **Add volume**
   - Create a volume named `openwebui_data` for OpenWebUI data
   - Create a volume named `openwebui_config` for configuration files

3. Upload your `mcp-config.json` file:
   - Navigate to the `openwebui_config` volume
   - Use the **Upload** button to upload your `mcp-config.json` file
   - Alternatively, you can use the **Create file** option to create it directly in Portainer

### 3. Deploy Using Portainer

1. In Portainer, navigate to **Stacks** in the left sidebar.

2. Click **Add stack**.

3. Enter a name for your stack (e.g., `openwebui`).

4. For the build method, select one of the following options:
   
   **Option 1: Web editor**
   - In the **Web editor** tab, paste the contents of the `docker-compose.yml` file from this repository.
   - Adjust the volume paths if you're using named volumes:
     ```yaml
     volumes:
       - openwebui_data:/app/backend/data
       # Instead of:
       # - /volume1/docker/openwebui/data:/app/backend/data
     ```

   **Option 2: Git repository**
   - In the **Repository** tab, enter your Git repository URL
   - Specify the reference name (branch, tag, or commit)
   - Set the compose path to `docker-compose.yml`

5. Click **Deploy the stack**.

6. Wait for the deployment to complete.

### 4. Verify the Deployment

1. Access OpenWebUI at `http://your-nas-ip:3000`

2. The MCP proxy will be available at `http://your-nas-ip:8001`

## Configuration

### Directory Structure

When using direct path mapping (as in the default docker-compose.yml):

```
/volume1/docker/openwebui/
├── data/           # OpenWebUI persistent data
├── config/         # Configuration files
│   └── mcp-config.json  # Your custom MCP configuration file
```

### Environment Variables

The following environment variables can be configured in the `.env` file:

- `OPENWEBUI_IMAGE_TAG`: The tag of the OpenWebUI image to use (default: `main`)
- `OPENWEBUI_PORT`: The port to expose OpenWebUI on (default: `3000`)
- `MCP_PROXY_PORT`: The port to expose the MCP proxy on (default: `8001`)
- `COMPOSE_PROFILES`: The Docker Compose profiles to enable (default: `tools`)

### MCP Configuration

A sample MCP configuration file is provided at `mcp-proxy/mcp-config.json.sample`. You can use this as a starting point for your own configuration.

## Maintenance

### Updating

To update the containers:

1. In Portainer, navigate to your stack.
2. Click **Editor**.
3. Make any necessary changes to the configuration.
4. Click **Update the stack**.

### Backing Up

Regularly back up your OpenWebUI data:

1. In Portainer, navigate to **Volumes**.
2. Select the volume containing your OpenWebUI data.
3. Use the **Backup** option if available, or export the data using the file browser.

### Logs

View logs in Portainer:

1. Navigate to **Containers** in the left sidebar.
2. Click on the container name (e.g., `open-webui` or `mcp-proxy`).
3. Click on the **Logs** tab to view the container logs.

## Troubleshooting

### Common Issues

1. **Services fail to start**:
   - Check the logs for error messages
   - Ensure the required directories exist and have appropriate permissions
   - Verify that the ports are not already in use by other services

2. **MCP proxy not working**:
   - Verify that your `mcp-config.json` file is correctly formatted
   - Check that the file is accessible to the container
   - Review the MCP proxy logs for any error messages

3. **Network connectivity issues**:
   - Ensure that the Docker network is properly configured
   - Check that the host.docker.internal extra host is correctly set up

## Alternative Volume Configuration

If you prefer to use named volumes instead of direct path mapping, modify the docker-compose.yml file as follows:

```yaml
version: '3.8'

services:
  openwebui:
    # ... other settings ...
    volumes:
      - openwebui_data:/app/backend/data
    # ... other settings ...

  mcp-proxy:
    # ... other settings ...
    volumes:
      - openwebui_config:/app/config
    # ... other settings ...

volumes:
  openwebui_data:
    external: true
  openwebui_config:
    external: true
```

With this configuration, you'll need to create the named volumes in Portainer before deploying the stack.

## License

This project is distributed under the terms of the open-source license.

## References

- [OpenWebUI Documentation](https://docs.openwebui.com/)
- [MCPO GitHub Repository](https://github.com/open-webui/mcpo)
- [Portainer Documentation](https://docs.portainer.io/)
