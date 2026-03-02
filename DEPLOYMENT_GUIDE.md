# Asset Manager - Complete Deployment Guide

**Deployment Date:** March 2, 2026

This guide provides step-by-step instructions for deploying the Asset Manager project:
- **Backend** on Render
- **Frontend** on Vercel
- **Database** on MongoDB Atlas

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Backend Deployment (Render)](#backend-deployment-render)
3. [Frontend Deployment (Vercel)](#frontend-deployment-vercel)
4. [Database Setup (MongoDB Atlas)](#database-setup-mongodb-atlas)
5. [Environment Variables](#environment-variables)
6. [Post-Deployment Testing](#post-deployment-testing)
7. [Troubleshooting](#troubleshooting)

---

## Prerequisites

Before you start, make sure you have:

- ✅ GitHub account with project repository pushed
- ✅ Render account (https://render.com) - Sign up free
- ✅ Vercel account (https://vercel.com) - Sign up free
- ✅ MongoDB Atlas account (https://www.mongodb.com/cloud/atlas) - Free tier available
- ✅ Node.js and npm installed locally
- ✅ Git installed and configured

---

## Backend Deployment (Render)

### Step 1: Prepare Backend for Deployment

#### 1.1 Update Backend `package.json`

Ensure the backend's `package.json` has the correct start script:

```json
{
  "name": "asset-manager-backend",
  "version": "1.0.0",
  "description": "Asset Manager Backend",
  "main": "dist/index.js",
  "scripts": {
    "dev": "ts-node src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "seed": "ts-node scripts/seed.ts"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.0.0",
    "cors": "^2.8.5",
    "dotenv": "^16.0.3",
    "bcryptjs": "^2.4.3",
    "jsonwebtoken": "^9.0.0"
  },
  "devDependencies": {
    "typescript": "^4.9.5",
    "@types/express": "^4.17.17",
    "@types/node": "^18.15.11",
    "ts-node": "^10.9.1"
  }
}
```

#### 1.2 Verify TypeScript Configuration

Check `backend/tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "moduleResolution": "node"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

#### 1.3 Create `.gitignore` (if not exists)

```
node_modules/
dist/
.env
.env.local
*.log
build/
```

### Step 2: Create Render Account & Service

#### 2.1 Sign Up / Log In to Render

1. Go to https://render.com
2. Sign up with GitHub (recommended) or create account
3. Authorize Render to access your GitHub repositories

#### 2.2 Create New Web Service

1. Click **New +** → **Web Service**
2. Select your GitHub repository (`adminPanal` or `testrepo`)
3. Click **Connect**

#### 2.3 Configure Web Service

Fill in the following settings:

| Setting | Value |
|---------|-------|
| **Name** | asset-manager-backend |
| **Environment** | Node |
| **Build Command** | `npm install && npm run build` |
| **Start Command** | `npm start` |
| **Plan** | Free (or Paid if needed) |

#### 2.4 Add Environment Variables

Click **Environment** and add the following variables:

```env
MONGO_URI=mongodb+srv://username:password@cluster.mongodb.net/asset-manager?retryWrites=true&w=majority
JWT_SECRET=your_super_secret_jwt_key_change_this
FRONTEND_URL=https://asset-manager-frontend.vercel.app
NODE_ENV=production
PORT=5001
```

**Important:** Get `MONGO_URI` from MongoDB Atlas (see [Database Setup](#database-setup-mongodb-atlas) section)

#### 2.5 Deploy Backend

1. Click **Create Web Service**
2. Wait for deployment to complete (2-5 minutes)
3. You'll get a URL like: `https://asset-manager-backend.onrender.com`
4. **Save this URL** - you'll need it for frontend deployment

#### 2.6 Monitor Deployment

- Click on your service to view logs
- Check for any errors during build or startup
- Once it shows "Live", your backend is deployed

---

## Frontend Deployment (Vercel)

### Step 1: Prepare Frontend for Deployment

#### 1.1 Update Frontend Environment Variables

Create `frontend/.env.production`:

```env
VITE_API_URL=https://asset-manager-backend.onrender.com/api
VITE_APP_NAME=Asset Manager
```

Replace the Render backend URL with your actual deployed backend URL.

#### 1.2 Update API Configuration

Edit `frontend/src/lib/api.ts`:

```typescript
const API_BASE = import.meta.env.VITE_API_URL || 'http://localhost:5001/api';

export const api = {
  baseURL: API_BASE,
  headers: {
    'Content-Type': 'application/json',
  },
};

// Axios instance
import axios from 'axios';

const instance = axios.create({
  baseURL: API_BASE,
});

// Add token to requests
instance.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

export default instance;
```

#### 1.3 Update `vite.config.ts`

```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:5001',
        changeOrigin: true,
      },
    },
  },
  build: {
    outDir: 'dist',
    sourcemap: false,
  },
})
```

#### 1.4 Update Frontend `package.json`

```json
{
  "name": "asset-manager-frontend",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext .ts,.tsx"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "axios": "^1.3.0",
    "react-router-dom": "^6.8.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^3.1.0",
    "vite": "^4.2.0",
    "typescript": "^4.9.5"
  }
}
```

### Step 2: Deploy to Vercel

#### 2.1 Sign Up / Log In to Vercel

1. Go to https://vercel.com
2. Click **Sign Up** → Select **Continue with GitHub**
3. Authorize Vercel to access your GitHub account

#### 2.2 Import Project

1. Click **Add New...** → **Project**
2. Select your repository (`adminPanal` or `testrepo`)
3. Click **Import**

#### 2.3 Configure Project

Fill in the following settings:

**Project Name:**
- `asset-manager-frontend`

**Framework Preset:**
- Select **Vite**

**Root Directory:**
- Set to `frontend`

**Build and Output Settings:**
- **Build Command:** `npm run build`
- **Output Directory:** `dist`
- **Install Command:** `npm install`

#### 2.4 Add Environment Variables

Click **Environment Variables** and add:

```env
VITE_API_URL=https://asset-manager-backend.onrender.com/api
VITE_APP_NAME=Asset Manager
```

#### 2.5 Deploy Frontend

1. Click **Deploy**
2. Wait for deployment to complete (1-3 minutes)
3. You'll get a URL like: `https://asset-manager-frontend.vercel.app`

#### 2.6 Add Custom Domain (Optional)

1. Go to project settings
2. Click **Domains**
3. Add your custom domain (requires DNS configuration)

---

## Database Setup (MongoDB Atlas)

### Step 1: Create MongoDB Atlas Account

1. Go to https://www.mongodb.com/cloud/atlas
2. Click **Sign Up** (or Log In)
3. Create account with email/password or OAuth

### Step 2: Create Free Cluster

1. Click **Create** → **Build a Database**
2. Select **M0 Free Tier** (good for development)
3. Select **AWS** as provider
4. Choose region closest to you
5. Enter **Cluster Name:** `asset-manager-cluster`
6. Click **Create Cluster** (takes 2-5 minutes)

### Step 3: Create Database User

1. In left sidebar, click **Database Access**
2. Click **Add New Database User**
3. Create username & password (save these!)
4. Click **Add User**

**Example Credentials:**
```
Username: assetmanager
Password: SecurePassword123!
```

### Step 4: Configure Network Access

1. In left sidebar, click **Network Access**
2. Click **Add IP Address**
3. Select **Allow Access from Anywhere** (for development)
   - OR add specific IPs (more secure)
4. Click **Confirm**

### Step 5: Get Connection String

1. Go to **Databases**
2. Click **Connect** button on your cluster
3. Select **Drivers** → **Node.js**
4. Copy the connection string

**Example:**
```
mongodb+srv://assetmanager:SecurePassword123!@asset-manager-cluster.mongodb.net/asset-manager?retryWrites=true&w=majority
```

5. Replace `<password>` with your database user password
6. Replace `<database>` with `asset-manager`

### Step 6: Verify Connection Locally

Test the connection string in your local `.env`:

```bash
cd backend
npm run dev
```

Check if the app connects to MongoDB without errors.

---

## Environment Variables

### Backend Environment Variables (Render)

Add these to Render's **Environment** section:

```env
# MongoDB
MONGO_URI=mongodb+srv://assetmanager:SecurePassword123!@asset-manager-cluster.mongodb.net/asset-manager?retryWrites=true&w=majority

# JWT
JWT_SECRET=your_secure_jwt_secret_key_min_32_chars

# Frontend URL
FRONTEND_URL=https://asset-manager-frontend.vercel.app

# Environment
NODE_ENV=production
PORT=5001

# Optional: AWS/Email Config
MAIL_SERVICE=gmail
MAIL_USER=your-email@gmail.com
MAIL_PASS=your-app-password
```

### Frontend Environment Variables (Vercel)

Add these to Vercel's **Environment Variables** section:

```env
VITE_API_URL=https://asset-manager-backend.onrender.com/api
VITE_APP_NAME=Asset Manager
VITE_ENV=production
```

### Local Development (`.env`)

Create `backend/.env` for local testing:

```env
MONGO_URI=mongodb+srv://assetmanager:SecurePassword123!@asset-manager-cluster.mongodb.net/asset-manager?retryWrites=true&w=majority
JWT_SECRET=dev_secret_key_change_in_production
FRONTEND_URL=http://localhost:5173
NODE_ENV=development
PORT=5001
```

---

## Post-Deployment Testing

### Step 1: Test Backend API

1. Get your Render backend URL: `https://asset-manager-backend.onrender.com`

2. Test using cURL:

```bash
# Test health check (if endpoint exists)
curl https://asset-manager-backend.onrender.com/api/auth/me

# Register user
curl -X POST https://asset-manager-backend.onrender.com/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "password123",
    "name": "Test User",
    "role": "student"
  }'

# Login
curl -X POST https://asset-manager-backend.onrender.com/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "password123"
  }'
```

3. Or use Postman to test endpoints with the base URL

### Step 2: Test Frontend

1. Open your Vercel frontend URL: `https://asset-manager-frontend.vercel.app`

2. Test the following scenarios:
   - User registration
   - User login
   - Navigate to different pages
   - View dashboard
   - Create/update/delete records

3. Open browser DevTools (F12) → **Console** to check for errors

4. Check **Network** tab to verify API calls reach your backend

### Step 3: Test Email/Notifications (if configured)

If you've configured email sending:

```bash
curl -X POST https://asset-manager-backend.onrender.com/api/notifications/send \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <your_jwt_token>" \
  -d '{
    "email": "recipient@example.com",
    "subject": "Test Email",
    "message": "This is a test"
  }'
```

---

## Troubleshooting

### Backend Issues

#### Error: "Cannot find module"

**Solution:**
1. Go to Render dashboard
2. Click your service
3. Click **Manual Deploy** → **Deploy latest commit**
4. Check build logs

```bash
# Or locally, verify dependencies
cd backend
npm install
npm run build
```

#### Error: "MongoDB connection failed"

**Solutions:**
1. Verify `MONGO_URI` is correct
2. Check database user credentials in MongoDB Atlas
3. Verify IP whitelist allows Render's IP (use "Allow from Anywhere" or check Render's IP)
4. Test connection string locally first

```bash
# Test locally
npm install mongodb
node -e "
const { MongoClient } = require('mongodb');
const uri = process.env.MONGO_URI;
MongoClient.connect(uri, (err, client) => {
  if (err) console.error(err);
  else console.log('Connected!');
  client?.close();
});
"
```

#### Error: "CORS error" on frontend

**Solution:**
Update backend's CORS settings in `src/index.ts`:

```typescript
import cors from 'cors';

app.use(cors({
  origin: [
    'http://localhost:5173',
    'http://localhost:3000',
    'https://asset-manager-frontend.vercel.app'
  ],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

Then redeploy backend on Render.

#### Error: "Service still starting"

**Solution:**
1. Wait 2-3 minutes for initial deployment
2. Free tier on Render spins down after 15 minutes of inactivity
3. First request after spin-down will be slow (30-60 seconds)
4. Upgrade to Paid plan for always-on service

### Frontend Issues

#### Error: "API call failing"

**Solutions:**

1. Verify `VITE_API_URL` is correct:
   ```bash
   # Check what's being used
   # Open DevTools → Console → type:
   console.log(import.meta.env.VITE_API_URL)
   ```

2. Rebuild and redeploy:
   ```bash
   cd frontend
   npm run build
   # Push to GitHub, Vercel will auto-deploy
   ```

3. Check Network tab in DevTools to see full API requests

#### Error: "Page not found" or "404"

**Solution:**
Add `vercel.json` to your frontend folder:

```json
{
  "routes": [
    {
      "src": "/(.*)",
      "dest": "/index.html"
    }
  ]
}
```

Then redeploy on Vercel.

#### Error: "Blank page loading"

**Solutions:**
1. Check browser console for JavaScript errors
2. Verify environment variables in Vercel are set correctly
3. Clear browser cache and reload
4. Check Vercel deployment logs for build errors

### Database Issues

#### Error: "MongoNetworkError"

**Solutions:**
1. Verify cluster is running in MongoDB Atlas
2. Check IP whitelist allows your IP
3. Delete and recreate cluster if persistent
4. Use "Allow from Anywhere" temporarily for debugging

#### Error: "Authentication failed"

**Solution:**
Verify database user password doesn't contain special characters that need escaping:

```
Bad: Pass@word!
Good: Pass123word (alphanumeric + underscore)
```

If using special characters, URL-encode them:
- `@` → `%40`
- `!` → `%21`
- `#` → `%23`

---

## Production Checklist

- [ ] Update all URLs from localhost to production URLs
- [ ] Change JWT_SECRET to something cryptographically secure
- [ ] Enable HTTPS everywhere
- [ ] Set NODE_ENV=production on backend
- [ ] Configure MongoDB with strong password
- [ ] Restrict MongoDB access by IP
- [ ] Set up error logging (Sentry, LogRocket, etc.)
- [ ] Configure email service for notifications
- [ ] Set up automated backups for MongoDB
- [ ] Enable rate limiting on backend
- [ ] Test all user flows end-to-end
- [ ] Set up monitoring/alerting
- [ ] Document API keys and secrets securely
- [ ] Plan disaster recovery procedures

---

## Scaling & Performance

### If backend is slow:

1. **Upgrade Render plan:**
   - Free tier has cold starts (30-60s)
   - Upgrade to Starter ($7/month) for better performance

2. **Optimize database queries:**
   - Add indexes to frequently queried fields
   - Implement pagination
   - Use projection to select only needed fields

3. **Enable caching:**
   - Use Redis for session/data caching
   - Implement HTTP caching headers

### If frontend is slow:

1. **Optimize bundle size:**
   ```bash
   cd frontend
   npm run build
   # Check bundle size
   ```

2. **Enable Vercel Analytics:**
   - Go to project settings
   - Enable Web Analytics

3. **Use image optimization:**
   - Convert PNG to WebP
   - Lazy load images

---

## Monitoring & Maintenance

### Monitor Backend (Render)

1. Go to Render dashboard
2. Click your service
3. Monitor:
   - CPU usage
   - Memory usage
   - Response times
   - Error logs

### Monitor Frontend (Vercel)

1. Go to Vercel project
2. Check **Analytics** → **Performance**
3. Monitor:
   - Page load times
   - Core Web Vitals
   - Error tracking

### Monitor Database (MongoDB Atlas)

1. Go to MongoDB Atlas dashboard
2. Click your cluster
3. Monitor:
   - Connections
   - Disk usage
   - Operations per second
   - Query performance

---

## Useful Commands

### Deploy Backend manually (Render):

```bash
git push origin main
# Render auto-deploys, or manually trigger in dashboard
```

### Deploy Frontend manually (Vercel):

```bash
git push origin main
# Vercel auto-deploys, or manually trigger in dashboard
```

### View Render logs:

Go to service → Logs (real-time streaming)

### View Vercel logs:

Go to project → Deployments → select version → View Logs

### Rollback Deployment:

**Render:** Go to service → Deploys → select previous → Re-deploy
**Vercel:** Go to Deployments → select version → Promote to Production

---

## Support & Resources

- **Render Docs:** https://render.com/docs
- **Vercel Docs:** https://vercel.com/docs
- **MongoDB Docs:** https://docs.mongodb.com
- **Asset Manager API Docs:** See `API_DOCUMENTATION.md` in this project

---

## Final Steps

Once everything is deployed:

1. ✅ Backend live at: `https://asset-manager-backend.onrender.com`
2. ✅ Frontend live at: `https://asset-manager-frontend.vercel.app`
3. ✅ Database: MongoDB Atlas
4. ✅ Share URLs with team
5. ✅ Create user accounts and test all features
6. ✅ Set up monitoring and alerts
7. ✅ Document any custom configurations

**Congratulations on deploying your Asset Manager project!** 🚀

---

**Last Updated:** March 2, 2026  
**Deployment Guide Version:** 1.0
