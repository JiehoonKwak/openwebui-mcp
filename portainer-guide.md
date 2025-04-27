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
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem@latest",
        "/app/backend/data"
      ]
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
5. Click **Deploy the stack**.

## 4. Verify the Deployment

1. Navigate to **Containers** to see if both containers are running.
2. Access OpenWebUI at `http://your-nas-ip:3000`.
3. The MCP proxy will be available at `http://your-nas-ip:8001`.

## 5. Troubleshooting

If you encounter issues:

1. Check container logs:

   - Navigate to **Containers**.
   - Click on the container name.
   - Click on the **Logs** tab.

2. Verify volume mounts:

   - Navigate to **Containers**.
   - Click on the container name.
   - Click on the **Inspect** tab.
   - Look for the "Mounts" section to ensure volumes are correctly mounted.

3. Check network connectivity:
   - Navigate to **Networks**.
   - Click on `openwebui_network`.
   - Verify that both containers are connected to this network.

## 6. Updating

To update the stack:

1. Navigate to **Stacks**.
2. Click on your stack name.
3. Click **Editor**.
4. Make any necessary changes.
5. Click **Update the stack**.

## 7. Backing Up

To back up your data:

1. Navigate to **Volumes** > **openwebui_data**.
2. Click **Browse**.
3. Use the download functionality to back up important files.
4. Repeat for the **openwebui_config** volume.
