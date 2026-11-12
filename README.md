<!-- v1.5.0 -->
# MCP Server for Curl

[![npm version](https://badge.fury.io/js/@247arjun%2Fmcp-curl.svg)](https://badge.fury.io/js/@247arjun%2Fmcp-curl)
[![npm downloads](https://img.shields.io/npm/dm/@247arjun/mcp-curl.svg)](https://www.npmjs.com/package/@247arjun/mcp-curl)

A Model Context Protocol (MCP) server that provides curl functionality, allowing AI assistants to make HTTP requests directly from their environment.

<a href="https://glama.ai/mcp/servers/@247arjun/mcp-curl">
  <img width="380" height="200" src="https://glama.ai/mcp/servers/@247arjun/mcp-curl/badge" alt="Curl Server MCP server" />
</a>

## Features

- **HTTP Methods**: Support for GET, POST, PUT, DELETE requests
- **File Downloads**: Download files with curl
- **Advanced Options**: Custom headers, timeouts, redirects, and more
- **JSON Support**: Built-in JSON data handling for POST/PUT requests
- **User Agent**: Custom User-Agent string support
- **Security**: Safe subprocess execution with input validation
- **Full curl pass-through** (`curl_raw`): arbitrary flags with explicit `allow_unsafe` gate
- **SSRF protection**: blocks cloud metadata (169.254.169.254), RFC1918, loopback by default
- **Rate limiting**: configurable per-IP requests per minute
- **Audit log**: every `curl_raw` invocation logged with secrets redacted
- **File retrieval**: `read_file` tool for chunked base64 reads from the downloads directory
- **HTTP transport**: Streamable HTTP endpoint for remote clients

## Installation

### Method 1: NPM Installation (Recommended)

```bash
# Install globally
npm install -g @247arjun/mcp-curl

# Or install locally in your project
npm install @247arjun/mcp-curl
```

### Method 2: From Source

```bash
# Clone the repository
git clone https://github.com/247arjun/mcp-curl.git
cd mcp-curl

# Install dependencies
npm install

# Build the project
npm run build

# Optional: Link globally
npm link
```

### Method 3: Direct from GitHub

```bash
# Install directly from GitHub
npm install -g git+https://github.com/247arjun/mcp-curl.git
```

## Configuration

### Claude Desktop Setup

Add to your Claude Desktop configuration file:

**Location:**
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%/Claude/claude_desktop_config.json`

**Configuration:**
```json
{
  "mcpServers": {
    "mcp-curl": {
      "command": "mcp-curl",
      "args": []
    }
  }
}
```

**Alternative: Using npx (no global install needed)**
```json
{
  "mcpServers": {
    "mcp-curl": {
      "command": "npx",
      "args": ["@247arjun/mcp-curl"]
    }
  }
}
```

**Local Development Setup**
```json
{
  "mcpServers": {
    "mcp-curl": {
      "command": "node",
      "args": ["/absolute/path/to/mcp-curl/build/index.js"]
    }
  }
}
```

After adding the configuration, restart Claude Desktop to load the MCP server.

## Available Tools

### 1. `curl_get`
Make HTTP GET requests.

**Parameters:**
- `url` (string): The URL to make the GET request to
- `headers` (array, optional): HTTP headers in the format 'Header: Value'
- `timeout` (number, optional): Request timeout in seconds
- `follow_redirects` (boolean, optional): Whether to follow redirects
- `user_agent` (string, optional): Custom User-Agent string

**Example:**
```json
{
  "url": "https://api.example.com/data",
  "headers": ["Authorization: Bearer token"],
  "timeout": 30,
  "follow_redirects": true,
  "user_agent": "MyApp/1.0"
}
```

### 2. `curl_post`
Make HTTP POST requests with data.

**Parameters:**
- `url` (string): The URL to make the POST request to
- `json_data` (object, optional): JSON object to send as POST data
- `data` (string, optional): Data to send in the POST request body
- `headers` (array, optional): HTTP headers
- `content_type` (string, optional): Content-Type header
- `timeout` (number, optional): Request timeout in seconds
- `follow_redirects` (boolean, optional): Whether to follow redirects

**Example:**
```json
{
  "url": "https://api.example.com/data",
  "json_data": {"key": "value"},
  "headers": ["Content-Type: application/json"]
}
```

### 3. `curl_put`
Make HTTP PUT requests.

**Parameters:**
- `url` (string): The URL to make the PUT request to
- `json_data` (object, optional): JSON object to send as PUT data
- `data` (string, optional): Data to send in the PUT request body
- `headers` (array, optional): HTTP headers
- `content_type` (string, optional): Content-Type header
- `timeout` (number, optional): Request timeout in seconds
- `follow_redirects` (boolean, optional): Whether to follow redirects

**Example:**
```json
{
  "url": "https://api.example.com/data/123",
  "data": "raw data",
  "content_type": "text/plain"
}
```

### 4. `curl_delete`
Make HTTP DELETE requests.

**Parameters:**
- `url` (string): The URL to make the DELETE request to
- `headers` (array, optional): HTTP headers
- `timeout` (number, optional): Request timeout in seconds
- `follow_redirects` (boolean, optional): Whether to follow redirects

**Example:**
```json
{
  "url": "https://api.example.com/data/123",
  "headers": ["Authorization: Bearer token"]
}
```

### 5. `curl_download`
Download files.

**Parameters:**
- `url` (string): The URL of the file to download
- `output_filename` (string, optional): Output filename
- `resume` (boolean, optional): Resume partial download if file exists
- `timeout` (number, optional): Request timeout in seconds
- `follow_redirects` (boolean, optional): Whether to follow redirects

**Example:**
```json
{
  "url": "https://example.com/file.zip",
  "output_filename": "downloaded_file.zip",
  "resume": true
}
```

### 6. `curl_advanced`
Execute curl with custom arguments (advanced users).

**Parameters:**
- `args` (array): Array of curl arguments (excluding 'curl' itself)

**Example:**
```json
{
  "args": ["-X", "PATCH", "-H", "Content-Type: application/json", "-d", "{\"status\":\"updated\"}", "https://api.example.com/items/1"]
}
```

### 7. `curl_raw`

Execute curl with arbitrary arguments - full pass-through, **no blacklist**. Requires `allow_unsafe: true`.

**Parameters:**
- `args` (array, required): curl arguments (excluding `curl` itself)
- `allow_unsafe` (boolean, required): explicit confirmation to run without safety checks
- `allow_private` (boolean, optional, default `false`): allow requests to private/loopback/link-local IPs. By default blocks SSRF (incl. `169.254.169.254` cloud metadata), RFC1918, loopback, CGNAT, IPv6 ULA/link-local.
- `max_output_bytes` (number, optional, default `1048576`): truncate stdout to this many bytes (1 MiB)

**Defaults** (prepended unless you specify your own): `--max-time 300` (5 min), `--max-filesize 209715200` (200 MiB).

**Example - `-o` (blocked in `curl_advanced`):**
```json
{ "args": ["-s", "-o", "output.json", "https://api.example.com/data"], "allow_unsafe": true }
```

**Example - multipart upload with `-F`:**
```json
{ "args": ["-F", "file=@./doc.pdf", "https://api.example.com/upload"], "allow_unsafe": true }
```

**Example - blocked by SSRF guard:**
```json
{ "args": ["http://169.254.169.254/latest/meta-data/"], "allow_unsafe": true }
```
Returns `Error: Refusing to connect to private/loopback/link-local address 169.254.169.254...`

**Memory warning (1 GB RAM servers):** `curl_raw` without `-o` buffers the **entire** response body in Node memory before truncating to `max_output_bytes` (1 MiB default). A single 200 MB download without `-o` = 200 MB peak RAM. Several concurrent large downloads can OOM a 1 GB VPS. **Always use `-o <file>` for downloads larger than 50 MB.** With `-o`, curl writes to disk and Node memory stays flat.

### Notion AI Integration

To connect this server to Notion AI:
1. Add a new MCP Connection in Notion.
2. **MCP server URL:** `https://<your-domain>/mcp`
3. **Authentication:** Select `Bearer token`.
4. **Prefix:** Change the dropdown to `Bearer` (do not use `Token`).
5. **Token:** Paste your `MCP_API_KEY`.
6. **(Optional but recommended)** In *Advanced settings*, add a custom header `X-API-Key` with your `MCP_API_KEY` as the value.

## HTTP Transport (Remote Clients)

By default the server uses **stdio**. For remote access, enable **Streamable HTTP** via env vars.

### Start HTTP mode

```bash
# Local (loopback only, no auth)
PORT=3000 node build/index.js

# With API key
PORT=3000 MCP_API_KEY=your-secret-key node build/index.js
```

Server listens on `http://127.0.0.1:3000/mcp`. `127.0.0.1` means **not exposed to network** - use SSH tunnel for remote access.

### Environment variables

| Variable | Default | Purpose |
|---|---|---|
| `MCP_TRANSPORT` | `stdio` | Set to `http` to enable HTTP mode |
| `PORT` | `3000` | HTTP listen port (also enables HTTP mode) |
| `HOST` | `127.0.0.1` | Bind address. `0.0.0.0` exposes to network - **set `MCP_API_KEY` first** |
| `MCP_API_KEY` | *(unset)* | If set, requires `X-API-Key` or `Authorization: Bearer <key>` |
| `MCP_ALLOWED_IPS` | *(unset)* | Comma-separated IP allowlist. Empty = accept all. Check server stderr for client IP |
| `MCP_RATE_LIMIT_PER_MIN` | `60` | Max requests per IP per minute |
| `MCP_CURL_RAW_LOG` | `./curl_raw.log` | Path to `curl_raw` audit log |

### Client configuration

**Claude Code** (native HTTP):
```bash
claude mcp add --transport http mcp-curl http://127.0.0.1:3000/mcp
```

**Claude Desktop** (via `mcp-remote`):
```json
{
  "mcpServers": {
    "mcp-curl": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "http://127.0.0.1:3000/mcp", "--transport", "http"]
    }
  }
}
```

With API key, add `"--header", "X-API-Key: your-secret-key"` to args.

### Testing HTTP endpoint

```bash
curl -X POST http://127.0.0.1:3000/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}'
```

`Accept: application/json, text/event-stream` header is **required** by the MCP Streamable HTTP spec.

### Health check

```bash
curl http://127.0.0.1:3000/health
# {"status":"ok","transport":"http"}
```

### VPS deployment (no Docker, no HTTPS)

**SSH tunnel (recommended - safest):**
```bash
# On VPS: bind to loopback only
HOST=127.0.0.1 PORT=8080 MCP_RATE_LIMIT_PER_MIN=20 node build/index.js

# On client: forward local port
ssh -L 8080:127.0.0.1:8080 user@your-vps
# Then use http://127.0.0.1:8080/mcp locally
```

**Public IP with allowlist + API key:**
```bash
# On VPS
HOST=0.0.0.0 PORT=8080 MCP_API_KEY=$(openssl rand -hex 32) node build/index.js
sudo ufw allow from YOUR_CLIENT_IP to any port 8080 proto tcp
sudo ufw deny 8080/tcp
sudo iptables -A OUTPUT -d 169.254.169.254 -j REJECT
```

### Discovering client IP

Without `MCP_ALLOWED_IPS` set, the server logs every `/mcp` request to stderr with the source IP:
```
[mcp] 2026-11-10T17:03:54.486Z ip=203.0.113.45 method=POST path=/mcp
```

On systemd: `sudo journalctl -u mcp-curl -f | grep '\[mcp\]'`

Take the IP from the log, add to `MCP_ALLOWED_IPS`, restart the service.

### 8. `read_file`

Read a file from the `/opt/mcp-curl/downloads` directory (on the server) in chunks. Returns base64-encoded data.

**Parameters:**
- `path` (string, required): Relative path to the file within `downloads/`
- `offset` (number, optional, default `0`): Byte offset to start reading from
- `length` (number, optional, default `65536`): Maximum bytes to read (capped at 10 MB per request)

**Example:**
```json
{ "path": "test.png", "offset": 0, "length": 1000000 }
```
**Response:**
```json
{
  "fileSize": 224566,
  "offset": 0,
  "length": 224566,
  "nextOffset": null,
  "hasMore": false,
  "base64": "iVBORw0KGgoAAAANSUhEUg..."
}
```
*Tip: If `hasMore` is `true`, call `read_file` again with `offset` set to `nextOffset`.*

## Usage Examples

### Make a GET request
```json
{
  "tool": "curl_get",
  "url": "https://api.example.com/users",
  "headers": ["Authorization: Bearer your-token"]
}
```

### POST JSON data
```json
{
  "tool": "curl_post",
  "url": "https://api.example.com/users",
  "json_data": {
    "name": "John Doe",
    "email": "john@example.com"
  }
}
```

### Download a file
```json
{
  "tool": "curl_download",
  "url": "https://example.com/file.zip",
  "output_filename": "download.zip"
}
```

## Development

### Prerequisites

- Node.js 18.0.0 or higher
- npm package manager
- curl command-line tool

### Building

```bash
npm run build
```

### Testing

Test the server manually:

```bash
echo '{"jsonrpc": "2.0", "id": 1, "method": "tools/list", "params": {}}' | node build/index.js
```

Run tests:
```bash
npm test
```

### Linting

```bash
npm run lint
npm run lint:fix
```

### Project Structure

```
mcp-curl/
├── src/
│   └── index.ts          # Main server implementation
├── build/
│   └── index.js          # Compiled JavaScript
├── test/
│   └── basic.test.cjs     # Basic functionality tests
├── examples/
│   └── test-server.js    # Example usage
├── .github/
│   └── workflows/
│       └── ci.yml        # CI/CD pipeline
├── package.json          # Project configuration
├── tsconfig.json         # TypeScript configuration
├── .eslintrc.json        # ESLint configuration
├── LICENSE               # MIT License
├── DEPLOYMENT.md         # Deployment guide
├── CONTRIBUTING.md       # Contribution guidelines
├── CHANGELOG.md          # Version history
└── README.md             # This file
```

## Verification

Test that the server is working:

```bash
# Test the built server
node build/index.js

# Should show: "Curl MCP Server running on stdio"
# Press Ctrl+C to exit
```

## Troubleshooting

### Common Issues

1. **"Command not found" error**
   - Ensure mcp-curl is installed globally: `npm install -g @247arjun/mcp-curl`
   - Or use npx: `"command": "npx", "args": ["@247arjun/mcp-curl"]`

2. **"Permission denied" error**
   - Check file permissions: `chmod +x build/index.js`
   - Rebuild the project: `npm run build`

3. **MCP server not appearing in Claude**
   - Verify JSON syntax in configuration file
   - Restart Claude Desktop completely
   - Check that the command path is correct

4. **"curl command not found"**
   - Install curl on your system (usually pre-installed on macOS/Linux)
   - Windows users: Install via package manager or download from curl website

### Debugging

Enable verbose logging by setting environment variable:
```bash
# For development
DEBUG=1 node build/index.js

# Test with sample input
echo '{"jsonrpc": "2.0", "method": "initialize", "params": {}}' | node build/index.js
```

## Security Notes

- Input validation and sanitization for all curl arguments
- Restricted file operations in advanced mode
- Safe subprocess execution using spawn instead of shell
- No arbitrary shell command execution
- Input validation with Zod schemas

### curl_raw security

`curl_raw` bypasses the blacklist of `curl_advanced` and can use **any** curl flag, including `-o` (write files), `-T` (upload local files), `-F @file` (multipart uploads), `--cert`/`--key` (client certs), `--proxy` (route through proxies). Requires explicit `allow_unsafe: true`.

**Built-in protections:**

- **SSRF guard** (on by default): DNS resolution checked against `169.254.0.0/16` (cloud metadata / IMDS), RFC1918 (`10/8`, `172.16/12`, `192.168/16`), loopback (`127/8`, `::1`), CGNAT (`100.64/10`), IPv6 ULA (`fc00::/7`) and link-local (`fe80::/10`). Override with `allow_private: true`.
- **Defaults**: `--max-time 300` and `--max-filesize 209715200` (200 MiB) are prepended unless you specify your own.
- **Audit log**: every invocation written to `curl_raw.log` (JSON lines) with `Authorization:`, `Cookie:`, `X-API-Key:`, `-u` values redacted. Override path with `MCP_CURL_RAW_LOG`.
- **Rate limit** (HTTP mode): `MCP_RATE_LIMIT_PER_MIN` requests per IP per minute.
- **IP allowlist** (HTTP mode): `MCP_ALLOWED_IPS` filters requests before they reach tools.

**What is NOT protected:**

- **Reading local files** (`-d @~/.ssh/id_rsa`, `-T ~/.aws/credentials`): the SSRF guard checks the *destination* URL, not local file reads. Mitigate with `ProtectHome=read-only` + `ProtectSystem=strict` (systemd) or `--read-only` (Docker).
- **DNS rebinding (TOCTOU)**: the SSRF guard resolves once, `curl` resolves again. A malicious DNS server can return a public IP for the check and `169.254.169.254` for the actual request. Mitigate with `iptables -A OUTPUT -d 169.254.169.254 -j REJECT`.

**Recommended on public servers:**

1. Run as unprivileged user (`User=mcp-curl` in systemd, `USER node` in Docker).
2. `ProtectSystem=strict`, `ProtectHome=read-only`, `PrivateTmp=true` (systemd) or `--read-only --tmpfs /tmp` (Docker).
3. `iptables -A OUTPUT -d 169.254.169.254 -j REJECT`.
4. Set `MCP_API_KEY` and `MCP_ALLOWED_IPS`.
5. Bind to `127.0.0.1` and use SSH tunnel - safest option.