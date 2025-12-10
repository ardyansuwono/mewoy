# 📝 Konversi dari Cloudflare Worker ke Node.js

## Perbandingan Kode

### Original Cloudflare Worker

```javascript
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  const url = new URL(request.url)
  const path = url.pathname + url.search

  const targetUrl = 'https://mwmpos01.akamaized.net' + path
  const refererUrl = 'https://www.mewatch.sg/'

  const modifiedRequest = new Request(targetUrl, {
    method: request.method,
    headers: {
      ...request.headers,
      'Referer': refererUrl,
    },
    body: request.method !== 'GET' ? await request.text() : undefined
  })

  const response = await fetch(modifiedRequest)

  const modifiedResponse = new Response(response.body, response)
  modifiedResponse.headers.set('Access-Control-Allow-Origin', '*')
  modifiedResponse.headers.set('Access-Control-Allow-Methods', 'GET, POST, OPTIONS')
  modifiedResponse.headers.set('Access-Control-Allow-Headers', '*')

  return modifiedResponse
}
```

**Keterbatasan CF Worker:**
- ❌ Tidak support HTTP proxy credentials
- ❌ Terbatas resource & execution time
- ❌ Environment variables limited

---

### Converted Node.js + Express + Axios

```javascript
import express from 'express';
import axios from 'axios';
import { HttpProxyAgent } from 'http-proxy-agent';
import { HttpsProxyAgent } from 'https-proxy-agent';

const app = express();
const PROXY_URL = process.env.PROXY_URL;
const TARGET_HOST = process.env.TARGET_HOST;

const httpAgent = new HttpProxyAgent(PROXY_URL);
const httpsAgent = new HttpsProxyAgent(PROXY_URL);

app.all('*', async (req, res) => {
  try {
    const targetUrl = `https://${TARGET_HOST}${req.path}${Object.keys(req.query).length ? '?' + new URLSearchParams(req.query) : ''}`;
    
    const response = await axios({
      method: req.method.toLowerCase(),
      url: targetUrl,
      headers: {
        ...req.headers,
        'Referer': 'https://www.mewatch.sg/',
      },
      httpAgent,
      httpsAgent,
      responseType: 'stream',
    });

    res.status(response.status);
    Object.entries(response.headers).forEach(([key, value]) => {
      res.set(key, value);
    });
    res.set('Access-Control-Allow-Origin', '*');
    
    response.data.pipe(res);
  } catch (error) {
    res.status(502).json({ error: error.message });
  }
});

app.listen(3000);
```

**Keuntungan Node.js:**
- ✅ HTTP Proxy Agent support
- ✅ Unlimited execution time
- ✅ Full Node.js ecosystem
- ✅ Easy scaling
- ✅ Better error handling

---

## Perbedaan Utama

| Aspek | Cloudflare Worker | Node.js (Zeabur) |
|-------|-------------------|------------------|
| **Proxy Support** | ❌ Native fetch tanpa auth | ✅ HttpProxyAgent |
| **Execution Time** | ⏱️ Max 30s | ⏱️ Unlimited |
| **Memory** | 128MB | 512MB+ |
| **Cost** | Free (limited) | Free tier (Zeabur) |
| **Scaling** | Auto (CF) | Auto (Zeabur) |
| **Framework** | Worker API | Node.js/Express |
| **Debugging** | Wrangler CLI | Local npm + logs |
| **Deployment** | `wrangler deploy` | `git push` |

---

## Migrasi Checklist

- ✅ Convert Fetch API → Axios
- ✅ Handle request body parsing
- ✅ Setup HTTP/HTTPS proxy agents
- ✅ Copy headers dari original request
- ✅ Add CORS headers
- ✅ Implement error handling
- ✅ Setup environment variables
- ✅ Create Dockerfile
- ✅ Setup package.json
- ✅ Deploy ke Zeabur

---

## Key Improvements

### 1. Proxy Support
**Cloudflare Worker:**
```javascript
// Cannot use proxy credentials
const response = await fetch(targetUrl, {
  headers: { 'Referer': refererUrl }
  // Proxy tidak support
})
```

**Node.js:**
```javascript
const httpAgent = new HttpProxyAgent('http://user:pass@host:port');
const response = await axios({
  httpAgent,
  httpsAgent,
  // Proxy credentials supported!
})
```

### 2. Error Handling
**Cloudflare Worker:**
```javascript
try {
  const response = await fetch(targetUrl)
  // Limited error info
} catch (error) {
  // Error handling limited
}
```

**Node.js:**
```javascript
try {
  const response = await axios(config);
  console.log(`Status: ${response.status}`);
} catch (error) {
  if (error.response) {
    // Server responded with error
    console.error(`HTTP ${error.response.status}`);
  } else if (error.request) {
    // Request made but no response
    console.error('No response received');
  } else {
    // Error in request setup
    console.error(error.message);
  }
}
```

### 3. Logging
**Cloudflare Worker:**
```javascript
// Limited logging to CF dashboard
console.log('Request received') // Only in CF logs
```

**Node.js:**
```javascript
console.log(`📨 ${req.method} ${req.path}`);
console.log(`🔗 Proxy: ${PROXY_URL}`);
console.error(`❌ Error: ${error.message}`);
// All logs visible in Zeabur dashboard
```

### 4. Environment Variables
**Cloudflare Worker (wrangler.toml):**
```toml
[env.production]
vars = { TARGET_HOST = "..." }
```

**Node.js (.env):**
```bash
PORT=3000
PROXY_URL=http://user:pass@host:port
TARGET_HOST=mwmpos01.akamaized.net
REFERER_URL=https://www.mewatch.sg/
```

---

## Performance Considerations

### Memory Usage
- Cloudflare Worker: 128MB shared
- Node.js Zeabur: 512MB+ isolated
- Streaming: Efficient pipe handling

### Bandwidth
- Both: Pay for egress
- CF Worker: Often cheaper for small-scale
- Zeabur: Better for sustained traffic

### Latency
- CF Worker: Global edge, very fast
- Zeabur: Specific region, slightly higher
- Mitigation: Use CDN (Cloudflare) in front of Zeabur

---

## Migration Guide Summary

```bash
# Step 1: Install dependencies
npm install express axios http-proxy-agent https-proxy-agent

# Step 2: Create Express app (src/server.js)
# - Copy request handling logic
# - Add proxy agents
# - Implement error handling

# Step 3: Setup Docker
# - Create Dockerfile
# - Create .dockerignore

# Step 4: Setup environment
# - Create .env.example
# - Update package.json

# Step 5: Deploy
# - Push to GitHub
# - Deploy to Zeabur
# - Set env vars in dashboard
```

---

## Troubleshooting Migration

### Issue: Proxy not working
```
Solution: Verify PROXY_URL format
Format: http://user:pass@host:port (for HTTP)
Format: http://user:pass@host:port (for HTTPS too)
```

### Issue: Headers not forwarding
```
Solution: Ensure headers copied correctly
axios({
  headers: {
    ...request.headers,
    'Referer': customReferer,
  }
})
```

### Issue: CORS still failing
```
Solution: Ensure CORS headers set
res.set('Access-Control-Allow-Origin', '*');
res.set('Access-Control-Allow-Methods', 'GET, POST, OPTIONS, ...');
res.set('Access-Control-Allow-Headers', '*');
```

---

**Migration complete! 🎉 Nikmati benefits dari Node.js + Zeabur!**
