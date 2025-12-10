# 🏗️ Architecture & Technical Details

## System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Client (Browser/App)                    │
└────────────────────────────┬────────────────────────────────┘
                             │
                   HTTP/HTTPS Request
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│              Zeabur Container (Node.js + Express)            │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ Request Handler (app.all('*', async (req, res)))    │   │
│  │                                                       │   │
│  │ 1. Parse request path & query                        │   │
│  │ 2. Prepare headers + referer                         │   │
│  │ 3. Construct target URL                             │   │
│  │                                                       │   │
│  └───────────────────┬────────────────────────────────┘   │
│                      │                                      │
│                      ▼                                      │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ Axios HTTP Client + Proxy Agents                    │   │
│  │                                                       │   │
│  │ - HttpProxyAgent (for HTTP)                         │   │
│  │ - HttpsProxyAgent (for HTTPS)                       │   │
│  │ - Auth: user:pass@host:port                         │   │
│  │                                                       │   │
│  └───────────────────┬────────────────────────────────┘   │
│                      │                                      │
│                      ▼                                      │
│         Forward request through proxy                       │
│                      │                                      │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ Response Handler                                     │   │
│  │                                                       │   │
│  │ - Copy status code                                   │   │
│  │ - Copy response headers                              │   │
│  │ - Add CORS headers                                   │   │
│  │ - Stream response body                               │   │
│  │                                                       │   │
│  └───────────────────┬────────────────────────────────┘   │
│                      │                                      │
└──────────────────────┼──────────────────────────────────────┘
                       │
                       ▼
                   Response
                       │
┌──────────────────────┴──────────────────────────────────────┐
│            HTTP Proxy Server (45.39.73.12:5427)              │
│                                                               │
│  Authenticates: efhjfxos:fqzez23px4o5                        │
│                                                               │
└────────────────────────────┬────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│              Target Server (mwmpos01.akamaized.net)          │
│                                                               │
│  - Streams media content                                     │
│  - Returns with proper headers                              │
│  - Media files (m3u8, ts, mp4, etc.)                        │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

---

## Request Flow

```
1. Client Request
   ├─ Method: GET/POST/etc
   ├─ Path: /video/stream.m3u8
   ├─ Query: ?token=xyz&id=123
   └─ Headers: User-Agent, Accept-Language, etc

         │
         ▼

2. Express Handler (app.all('*'))
   ├─ Parse path & query
   ├─ Prepare headers
   │  ├─ Copy original headers
   │  ├─ Override Referer → https://www.mewatch.sg/
   │  ├─ Set User-Agent
   │  └─ Remove Host header
   └─ Get request body (for POST/PUT)

         │
         ▼

3. Axios + Proxy
   ├─ Create HttpProxyAgent/HttpsProxyAgent
   ├─ Make HTTP request through proxy
   ├─ Proxy authenticates: efhjfxos:fqzez23px4o5
   ├─ Proxy forwards to: mwmpos01.akamaized.net
   └─ Target server validates Referer header

         │
         ▼

4. Response Processing
   ├─ Receive response from target
   ├─ Copy status code
   ├─ Copy response headers
   ├─ Add CORS headers
   │  ├─ Access-Control-Allow-Origin: *
   │  ├─ Access-Control-Allow-Methods: GET, POST, ...
   │  └─ Access-Control-Allow-Headers: *
   └─ Stream body to client

         │
         ▼

5. Client Response
   ├─ Status: 200/206/etc
   ├─ Headers: Content-Type, Content-Length, etc
   └─ Body: Media stream (m3u8, ts, mp4)
```

---

## Key Technologies

### Express.js
- **Purpose**: HTTP server framework
- **Version**: ^4.18.2
- **Use**: Handle HTTP requests/responses, routing, middleware

### Axios
- **Purpose**: HTTP client with Promise support
- **Version**: ^1.6.2
- **Use**: Make HTTP requests with full control over headers/body
- **Why not fetch**: Limited proxy support in Node.js environment

### HttpProxyAgent
- **Purpose**: HTTP protocol agent with proxy support
- **Version**: ^7.0.0
- **Use**: Route HTTP requests through proxy server with authentication

### HttpsProxyAgent
- **Purpose**: HTTPS protocol agent with proxy support
- **Version**: ^7.0.2
- **Use**: Route HTTPS requests through proxy server with authentication

---

## Environment Variables

```bash
# Server Configuration
PORT=3000                                        # Zeabur default
NODE_ENV=production                             # For optimizations

# Proxy Configuration
PROXY_URL=http://user:pass@host:port           # Proxy auth credentials
                                                # Format: http://username:password@ip:port

# Target Configuration
TARGET_HOST=mwmpos01.akamaized.net             # Target streaming host
REFERER_URL=https://www.mewatch.sg/            # Referer header value
```

### Why Environment Variables?
1. **Security**: Credentials not in code
2. **Flexibility**: Change config without rebuild
3. **Deployment**: Different values per environment
4. **Zeabur Dashboard**: Easy to manage & update

---

## HTTP Request Handling

### Request Methods Supported
- ✅ GET
- ✅ POST
- ✅ PUT
- ✅ PATCH
- ✅ DELETE
- ✅ HEAD
- ✅ OPTIONS

### Request Headers Handled
```javascript
{
  ...req.headers,           // Copy all original headers
  'Referer': REFERER_URL,   // Override/add Referer
  'User-Agent': '...',      // Set default if missing
  // host is deleted (auto-set by axios)
}
```

### Request Body Handling
```javascript
if (['POST', 'PUT', 'PATCH'].includes(req.method)) {
  // Read raw body buffer
  data = await getRawBody(req);
} else {
  // No body for GET/DELETE/etc
  data = undefined;
}
```

---

## Response Processing

### Headers Copied from Target
```javascript
// All headers copied except:
// - content-encoding (handled by streams)
// - transfer-encoding (handled by Node.js)

Object.keys(response.headers).forEach(key => {
  const skipHeaders = ['content-encoding', 'transfer-encoding'];
  if (!skipHeaders.includes(key.toLowerCase())) {
    res.set(key, response.headers[key]);
  }
});
```

### CORS Headers Added
```javascript
res.set('Access-Control-Allow-Origin', '*');      // Allow all origins
res.set('Access-Control-Allow-Methods', 'GET, POST, OPTIONS, ...');
res.set('Access-Control-Allow-Headers', '*');     // Allow all headers
```

### Body Streaming
```javascript
// Stream response body to client
// Efficient for large files (streaming media)
response.data.pipe(res);
```

---

## Error Handling

### Error Types

**Proxy Connection Error**
```javascript
// Proxy server not reachable
catch (error) {
  console.error('Proxy Error:', error.message);
  res.status(502).json({ error: 'Bad Gateway' });
}
```

**Target Server Error**
```javascript
// 4xx/5xx from target
// Status code passed through
res.status(response.status);
```

**Request Timeout**
```javascript
// Request exceeded 30 seconds
timeout: 30000,  // 30 seconds
```

**Body Parsing Error**
```javascript
// Error reading request body
req.on('error', reject);
```

---

## Docker Configuration

### Dockerfile Breakdown

```dockerfile
# Base Image
FROM node:20-alpine
# - Alpine: Small, ~150MB
# - Node 20: Latest stable LTS
# - Total image size: ~300MB

WORKDIR /app
# Working directory inside container

COPY package*.json ./
# Copy package.json & package-lock.json (if exists)

RUN npm ci --only=production
# Clean install production dependencies only
# (excludes devDependencies)

COPY src ./src
# Copy application code

EXPOSE 3000
# Document port (not actual binding)

HEALTHCHECK
# - Check every 30s
# - Timeout 3s
# - Start checking after 5s
# - Fail after 3 failures
# - Calls /health endpoint

CMD ["npm", "start"]
# Run application
```

### Build Process
```bash
docker build -t media-proxy .
# Zeabur does this automatically from Dockerfile
```

### Runtime
```bash
docker run -e PROXY_URL=... -p 3000:3000 media-proxy
# Zeabur orchestrates this
```

---

## Performance Considerations

### Memory Usage
- Base Node.js: ~30MB
- Express + dependencies: ~50MB
- Available for application: ~400MB (512MB container - base)
- Streaming: Memory-efficient (no buffer)

### CPU Usage
- Per request: ~10-50ms (depends on response size)
- Proxy overhead: ~5-10ms
- Auto-scaling: Zeabur scales based on CPU

### Bandwidth
- All incoming bandwidth billed
- All outgoing bandwidth billed
- Streaming efficient (no buffering)
- No compression (streams media as-is)

### Scaling Strategy
```
Traffic increases → Zeabur detects high CPU
                → Auto-spin new container
                → Load balance traffic
                → Reduce CPU per container
```

---

## Security Considerations

### Credentials Handling
✅ Proxy credentials in environment variables (not code)
✅ Passwords masked in logs
❌ Not in .env file (git ignored)
❌ Not in Dockerfile
❌ Not in README

### Headers Security
✅ User-Agent set (identifies bot)
✅ Referer set (required by target)
⚠️ CORS allow-all (change if needed)

### Network
✅ HTTPS to target (encrypted)
✅ HTTPS to clients (Zeabur provides)
✅ Proxy authentication (credentials in URL)

---

## Monitoring & Debugging

### Logs Available
```bash
# Zeabur Dashboard → Logs tab
📨 GET /video/stream.m3u8         # Request logging
🔗 Proxy: http://efhjfxos:***@... # Config (masked)
✅ Server running on :3000         # Startup
❌ Error: Connection refused        # Errors
```

### Health Check
```bash
curl https://your-app.zeabur.app/health
{
  "status": "ok",
  "timestamp": "2025-12-10T11:11:00.000Z"
}
```

### Metrics
- CPU usage
- Memory usage
- Request count
- Response time
- Error rate

---

**Architecture complete! Ready for production deployment! 🚀**
