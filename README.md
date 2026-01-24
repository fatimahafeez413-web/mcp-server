# Weather MCP Server - Usage Guide

A Model Context Protocol (MCP) server that provides weather information using the National Weather Service API. Supports both local (stdio) and remote (HTTP/SSE) access.

## Features

- **Dual Transport Support**: Run locally via stdio or remotely via HTTP/SSE
- **Weather Alerts**: Get active weather alerts by state
- **Weather Forecasts**: Get detailed forecasts by coordinates (US locations only)
- **Session Management**: Automatic session handling for remote connections

## Installation

```bash
npm install
npm run build
```

## Usage

### Local Mode (Stdio Transport)

For local use with MCP clients like Claude Desktop:

```bash
# Default mode - no environment variable needed
node build/index.js

# Or explicitly set stdio mode
TRANSPORT=stdio node build/index.js
```

**Configuration for Claude Desktop** (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "weather": {
      "command": "node",
      "args": ["/path/to/weather/build/index.js"]
    }
  }
}
```

### Remote Mode (HTTP/SSE Transport)

For remote access via HTTP:

```bash
# Start HTTP server on default port 3000
TRANSPORT=http node build/index.js

# Or specify a custom port
TRANSPORT=http PORT=8080 node build/index.js
```

The server will start on `http://localhost:3000/mcp` (or your specified port).

## Remote Access Examples

### 1. Initialize Session

```bash
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "initialize",
    "params": {
      "protocolVersion": "2024-11-05",
      "capabilities": {},
      "clientInfo": {
        "name": "my-client",
        "version": "1.0.0"
      }
    }
  }'
```

**Response**: Returns session ID in `mcp-session-id` header. Save this for subsequent requests.

### 2. Get Weather Forecast

```bash
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "mcp-session-id: YOUR_SESSION_ID" \
  -d '{
    "jsonrpc": "2.0",
    "id": 2,
    "method": "tools/call",
    "params": {
      "name": "get_forecast",
      "arguments": {
        "latitude": 37.7749,
        "longitude": -122.4194
      }
    }
  }'
```

### 3. Get Weather Alerts

```bash
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "mcp-session-id: YOUR_SESSION_ID" \
  -d '{
    "jsonrpc": "2.0",
    "id": 3,
    "method": "tools/call",
    "params": {
      "name": "get_alerts",
      "arguments": {
        "state": "CA"
      }
    }
  }'
```

### 4. Open SSE Stream (Optional)

For receiving server-initiated messages:

```bash
curl -N http://localhost:3000/mcp \
  -H "Accept: text/event-stream" \
  -H "mcp-session-id: YOUR_SESSION_ID"
```

### 5. Terminate Session

```bash
curl -X DELETE http://localhost:3000/mcp \
  -H "mcp-session-id: YOUR_SESSION_ID"
```

## Available Tools

### `get_forecast`

Get weather forecast for a location.

**Parameters:**
- `latitude` (number): Latitude of the location (-90 to 90)
- `longitude` (number): Longitude of the location (-180 to 180)

**Note**: Only US locations are supported by the National Weather Service API.

### `get_alerts`

Get active weather alerts for a US state.

**Parameters:**
- `state` (string): Two-letter state code (e.g., "CA", "NY")

## Environment Variables

| Variable | Description | Default | Example |
|----------|-------------|---------|---------|
| `TRANSPORT` | Transport mode: `stdio` or `http` | `stdio` | `http` |
| `PORT` | HTTP server port (HTTP mode only) | `3000` | `8080` |

## Architecture

### Stdio Mode
- Single server instance
- Communicates via stdin/stdout
- Ideal for local MCP clients

### HTTP Mode
- Express-based HTTP server
- Session-based architecture
- Multiple concurrent sessions supported
- Automatic session cleanup on disconnect
- SSE streaming for real-time updates

## Deployment

### Local Development
```bash
TRANSPORT=http PORT=3000 node build/index.js
```

### Production (with PM2)
```bash
pm2 start build/index.js --name weather-mcp -- TRANSPORT=http PORT=3000
```

### Docker
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY build ./build
ENV TRANSPORT=http
ENV PORT=3000
EXPOSE 3000
CMD ["node", "build/index.js"]
```

## Troubleshooting

### "Failed to retrieve grid point data"
- Ensure coordinates are within the United States
- NWS API only supports US locations

### "Invalid or missing session ID"
- Initialize a session first using the `initialize` method
- Include the `mcp-session-id` header in all subsequent requests

### Server not starting in HTTP mode
- Check if port is already in use
- Verify `TRANSPORT=http` environment variable is set

## License

ISC
