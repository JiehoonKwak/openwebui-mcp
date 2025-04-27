# Portainer Setup Guide for OpenWebUI with MCP

This guide provides step-by-step instructions for setting up OpenWebUI with MCP on your Synology NAS using Portainer, without requiring SSH access.

## 1. Create Volumes in Portainer

1. Log in to your Portainer instance.
2. Navigate to **Volumes** in the left sidebar.
3. Click **Add volume**.
4. Create the following volumes:
   - `openwebui_data` - For OpenWebUI persistent data
   - `openwebui_config` - For configuration files

## 2. Prepare the MCP Configuration

1. In Portainer, navigate to **Volumes** > **openwebui_config**.
2. Click **Browse**.
3. Click **Create file**.
4. Name the file `mcp-config.json`.
5. Paste the following content (or your custom configuration):

```json
{
  "mcpServers": {
    "fetch": {
      "command": "uvx",
      "args": ["mcp-server-fetch", "--ignore-robots-txt"]
    },
    "time": {
      "command": "uvx",
      "args": ["mcp-server-time", "--local-timezone=Asia/Seoul"]
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem@latest", "/app/backend/data"]
    },
    "think": {
      "command": "uvx",
      "args": ["mcp-server-think"]
    }
  }
}
```

6. Click **Create**.

## 3. Deploy the Stack

1. Navigate to **Stacks** in the left sidebar.
2. Click **Add stack**.
3. Enter a name for your stack (e.g., `openwebui`).
4. In the **Web editor** tab, paste the contents of the `docker-compose.portainer.yml` file.
5. **Important**: Make sure the `command` line for the mcp-proxy service is included:
   ```yaml
   command: python -m mcpo --host 0.0.0.0 --port 8001 --config /app/config/mcp-config.json
   ```
6. Click **Deploy the stack**.

## 4. Forcing a Rebuild of the Image

If you're still encountering issues with the mcp-proxy container, you may need to force a rebuild of the image:

1. Navigate to **Stacks** in the left sidebar.
2. Click on your stack name.
3. Click **Editor**.
4. Add the following line to the `build` section of the mcp-proxy service:
   ```yaml
   build:
     context: ./mcp-proxy
     dockerfile: Dockerfile
     no_cache: true  # Add this line to force a rebuild
   ```
5. Click **Update the stack**.

## 5. Verify the Deployment

1. Navigate to **Containers** to see if both containers are running.
2. Access OpenWebUI at `http://your-nas-ip:3000`.
3. The MCP proxy will be available at `http://your-nas-ip:8001`.

## 6. Troubleshooting

### Common Issues

#### "mcpo: executable file not found in $PATH" Error

If you encounter an error like:
```
Error response from daemon: failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "mcpo": executable file not found in $PATH: unknown
```

This indicates that the MCP proxy container cannot find the mcpo executable. We've addressed this issue in two ways:

1. **Updated Dockerfile**: The Dockerfile now installs mcpo using pip directly and runs it using Python's module system.
2. **Explicit Command**: The docker-compose.yml file now includes an explicit command to run mcpo using Python's module system.

If you still encounter this error:

1. Make sure the `command` line is included in your docker-compose.yml file:
   ```yaml
   command: python -m mcpo --host 0.0.0.0 --port 8001 --config /app/config/mcp-config.json
   ```

2. Force a rebuild of the image by adding `no_cache: true` to the build section.

3. If using a Git repository, make sure you're using the latest commit.

#### Other Common Issues

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

### Checking Container Logs

1. Navigate to **Containers**.
2. Click on the container name.
3. Click on the **Logs** tab.

### Verifying Volume Mounts

1. Navigate to **Containers**.
2. Click on the container name.
3. Click on the **Inspect** tab.
4. Look for the "Mounts" section to ensure volumes are correctly mounted.

### Checking Network Connectivity

1. Navigate to **Networks**.
2. Click on `openwebui_network`.
3. Verify that both containers are connected to this network.

## 7. Updating

To update the stack:

1. Navigate to **Stacks**.
2. Click on your stack name.
3. Click **Editor**.
4. Make any necessary changes.
5. Click **Update the stack**.

## 8. Backing Up

To back up your data:

1. Navigate to **Volumes** > **openwebui_data**.
2. Click **Browse**.
3. Use the download functionality to back up important files.
4. Repeat for the **openwebui_config** volume.
