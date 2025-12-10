# Zeabur Media Proxy Deployment Guide

## Struktur Project

```
media-proxy-zeabur/
├── src/
│   └── server.js          # Main application
├── .env.example           # Environment variables template
├── package.json           # Dependencies
├── Dockerfile             # Container configuration
├── .dockerignore          # Ignore files for Docker
└── README.md              # Documentation
```

## Setup Instructions

### 1. Lokal Development
```bash
# Clone atau buat folder project
mkdir media-proxy-zeabur
cd media-proxy-zeabur

# Install dependencies
npm install

# Setup environment
cp .env.example .env
# Edit .env dan masukkan proxy credentials jika diperlukan

# Run locally
npm start
```

### 2. Deploy ke Zeabur

#### Option A: Auto-Deploy dari GitHub
1. Push code ke GitHub repository
2. Login ke Zeabur (zeabur.com)
3. Click "New Project" → "Deploy from Git"
4. Pilih repository Anda
5. Zeabur akan otomatis detect Dockerfile dan build

#### Option B: Manual Deploy
1. Install Zeabur CLI: `npm i -g @zeabur/cli`
2. Login: `zeabur auth`
3. Deploy: `zeabur deploy`
4. Ikuti interactive prompts

### 3. Environment Variables di Zeabur
Di dashboard Zeabur, setup environment variables:
- `PORT`: 3000 (default)
- `PROXY_URL`: http://efhjfxos:fqzez23px4o5@45.39.73.12:5427
- `TARGET_HOST`: mwmpos01.akamaized.net
- `REFERER_URL`: https://www.mewatch.sg/

## Testing

### Local Test
```bash
# Test proxy functionality
curl http://localhost:3000/test/path?param=value

# Test dengan real streaming URL
curl http://localhost:3000/video/stream.m3u8
```

### Production Test
Setelah deploy ke Zeabur, gunakan domain yang diberikan Zeabur:
```bash
curl https://your-app.zeabur.app/video/stream.m3u8
```

## Monitoring
- Logs: Lihat di Zeabur Dashboard → Logs tab
- Performance: Monitor CPU, Memory, dan bandwidth
- Errors: Check request/response logs untuk debugging

## Notes
- Proxy credentials tersimpan di environment variables, tidak dalam code
- CORS headers sudah dikonfigurasi
- Support GET, POST, dan OPTIONS methods
- Proxy digunakan untuk koneksi ke akamaized.net
