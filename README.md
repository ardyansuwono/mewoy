# Media Proxy untuk Zeabur

Proxy server untuk streaming media dengan dukungan HTTP proxy, referer header customization, dan CORS support.

## 🎯 Fitur

- ✅ HTTP & HTTPS proxy support
- ✅ Custom referer headers
- ✅ CORS enabled (Access-Control-Allow-Origin: *)
- ✅ Support semua HTTP methods (GET, POST, PUT, PATCH, DELETE, OPTIONS)
- ✅ Stream response untuk file besar
- ✅ Error handling & logging
- ✅ Health check endpoint
- ✅ Graceful shutdown

## 📦 Dependencies

- **express**: Web framework
- **axios**: HTTP client dengan proxy support
- **http-proxy-agent**: HTTP proxy agent
- **https-proxy-agent**: HTTPS proxy agent

## 🚀 Quick Start

### 1. Setup Lokal

```bash
# Clone/buat project
mkdir media-proxy-zeabur && cd media-proxy-zeabur

# Buat .env file
cp .env.example .env

# Install dependencies
npm install

# Run server
npm start
```

Server akan berjalan di `http://localhost:3000`

### 2. Testing Lokal

```bash
# Health check
curl http://localhost:3000/health

# Proxy request (example)
curl http://localhost:3000/test/path?param=value
```

## 🌐 Deploy ke Zeabur

### Method 1: GitHub Auto-Deploy (Recommended)

1. **Push ke GitHub**
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin https://github.com/username/media-proxy-zeabur.git
   git push -u origin main
   ```

2. **Di Zeabur Dashboard**
   - Login ke https://zeabur.com
   - Click "New Project"
   - Select "Deploy from Git"
   - Pilih repository
   - Zeabur auto-detect Dockerfile

3. **Set Environment Variables**
   - Di Zeabur Dashboard → Project Settings → Environment Variables
   - Tambahkan:
     ```
     PORT=3000
     PROXY_URL=http://efhjfxos:fqzez23px4o5@45.39.73.12:5427
     TARGET_HOST=mwmpos01.akamaized.net
     REFERER_URL=https://www.mewatch.sg/
     NODE_ENV=production
     ```

### Method 2: CLI Deploy

```bash
# Install Zeabur CLI
npm install -g @zeabur/cli

# Login
zeabur auth

# Deploy
zeabur deploy

# Follow interactive prompts
```

## 📝 Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `PORT` | Server port | 3000 |
| `PROXY_URL` | HTTP proxy URL dengan auth | (from env) |
| `TARGET_HOST` | Target host untuk proxy | mwmpos01.akamaized.net |
| `REFERER_URL` | Custom referer header | https://www.mewatch.sg/ |
| `NODE_ENV` | Environment (development/production) | development |

## 🔍 API Endpoints

### GET/POST/etc /{path}

Proxy semua request ke target host.

**Example:**
```bash
# Original request
https://your-zeabur-app.app/video/stream.m3u8?token=abc123

# Akan di-proxy ke
https://mwmpos01.akamaized.net/video/stream.m3u8?token=abc123
```

**Response Headers Added:**
```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, OPTIONS, PUT, PATCH, DELETE
Access-Control-Allow-Headers: *
Referer: https://www.mewatch.sg/
```

### GET /health

Health check endpoint untuk monitoring.

**Response:**
```json
{
  "status": "ok",
  "timestamp": "2025-12-10T11:11:00.000Z"
}
```

## 🛠️ Development

### Local Development dengan Auto-Reload

```bash
npm run dev
```

### Debug Logging

Aplikasi otomatis log semua requests:
```
📨 GET /video/stream.m3u8
📨 POST /api/data
```

Passwords di-mask otomatis:
```
🔗 Proxy: http://efhjfxos:***@45.39.73.12:5427
```

## 📊 Monitoring di Zeabur

- **Logs**: Dashboard → Logs tab
- **Metrics**: CPU, Memory, Bandwidth usage
- **Health**: /health endpoint
- **Domains**: Custom domain assignment

## 🐛 Troubleshooting

### Port Already in Use
```bash
# Change port in .env
PORT=3001
```

### Proxy Connection Error
- Verify proxy URL format: `http://user:pass@host:port`
- Check proxy credentials are correct
- Ensure proxy server is accessible

### CORS Issues
- CORS headers automatically added
- If still having issues, check browser console

### Timeout on Large Files
- Axios timeout set to 30 seconds
- Modify di `src/server.js` baris `timeout: 30000`

## 📄 File Structure

```
media-proxy-zeabur/
├── src/
│   └── server.js              # Main application
├── .env.example               # Environment template
├── .gitignore                 # Git ignore rules
├── Dockerfile                 # Docker config
├── package.json               # NPM dependencies
└── README.md                  # This file
```

## 🔐 Security Notes

- ✅ Proxy credentials tidak stored di code
- ✅ Passwords di-mask di logs
- ✅ Environment variables digunakan
- ⚠️ CORS allowed all origins - gunakan di internal/trusted networks
- ⚠️ Monitor bandwidth usage untuk avoid abuse

## 📞 Support

Untuk issues atau questions:
1. Check logs di Zeabur Dashboard
2. Verify environment variables
3. Test locally dengan `npm start`
4. Check network connectivity ke proxy server

## 📜 License

MIT
