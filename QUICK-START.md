# 🚀 Panduan Singkat Deploy ke Zeabur

## ⚡ Setup dalam 5 Menit

### Langkah 1: Persiapan File (2 menit)

Copy semua file ini ke folder project:
- `package.json` - Dependencies
- `Dockerfile` - Container config  
- `src/server.js` - Main app
- `.env.example` - Environment template
- `README.md` - Documentation

### Langkah 2: Setup Lokal (1 menit)

```bash
# Install dependencies
npm install

# Test lokal
npm start
```

Buka browser: http://localhost:3000/health

Harusnya return: `{"status":"ok",...}`

### Langkah 3: Push ke GitHub (1 menit)

```bash
git init
git add .
git commit -m "Media proxy project"
git remote add origin https://github.com/USERNAME/media-proxy-zeabur.git
git branch -M main
git push -u origin main
```

### Langkah 4: Deploy di Zeabur (1 menit)

1. Login ke https://zeabur.com
2. Click "New Project"
3. Select "Deploy from Git"
4. Connect GitHub dan pilih `media-proxy-zeabur` repo
5. Zeabur akan otomatis build (detect Dockerfile)

### Langkah 5: Set Environment Variables (1 menit)

Di dashboard Zeabur:
- Buka project → Settings → Environment Variables
- Add variables:
  ```
  PORT=3000
  PROXY_URL=http://efhjfxos:fqzez23px4o5@45.39.73.12:5427
  TARGET_HOST=mwmpos01.akamaized.net
  REFERER_URL=https://www.mewatch.sg/
  NODE_ENV=production
  ```

✅ Done! App akan auto-deploy setiap push ke main branch

---

## 🧪 Testing Deployment

Setelah deploy, Zeabur akan assign domain otomatis (e.g., `app-name-abc123.zeabur.app`)

Test dengan:
```bash
# Health check
curl https://your-app.zeabur.app/health

# Test proxy
curl https://your-app.zeabur.app/test/path?param=value
```

---

## 📊 Monitoring

Di Zeabur Dashboard:
- **Logs** → Real-time request logs
- **Metrics** → CPU, Memory, Bandwidth
- **Domains** → Custom domain setup (optional)
- **Settings** → Modify env vars

---

## 🔄 Update Code

Untuk update aplikasi:
```bash
# Edit code lokal
# Commit & push
git add .
git commit -m "Update description"
git push

# Zeabur akan auto-redeploy dalam 1-2 menit
```

---

## ❌ Troubleshooting

### Build Failed
- Check logs di Zeabur Dashboard
- Verify Dockerfile exists
- Ensure package.json valid JSON

### App Crashes
- Check Environment Variables set correctly
- Verify PROXY_URL format
- Check logs untuk error messages

### Timeout/Slow
- Check network connection
- Verify proxy server is accessible
- Monitor CPU/Memory di dashboard

### Domain Not Working
- Wait 2-3 minutes untuk DNS propagation
- Check status di Zeabur Dashboard
- Custom domain setup di Settings

---

## 💡 Tips

1. **Environment Variables** - Jangan hardcode credentials, selalu gunakan env vars
2. **Logs** - Check Zeabur logs untuk debug
3. **Health Check** - Monitor `/health` endpoint regularly
4. **Auto-Scaling** - Zeabur auto-scale resource based on demand
5. **GitHub Sync** - Automatic redeploy on push

---

## 📞 Helpful Commands

```bash
# Local testing
npm start                    # Start server
npm run dev                  # With auto-reload

# Git push trigger deploy
git push origin main         # Auto-deploy di Zeabur

# Check deployment logs
zeabur logs --project-name your-project-name
```

---

**🎉 Selamat! Project Anda ready untuk production!**
