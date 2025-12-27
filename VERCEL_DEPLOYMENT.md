# Vercel Deployment Guide

Complete step-by-step guide for deploying the AI Assistant application to Vercel.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Vercel Account Setup](#vercel-account-setup)
3. [Database Setup](#database-setup)
4. [Project Preparation](#project-preparation)
5. [Deployment Methods](#deployment-methods)
6. [Environment Variables](#environment-variables)
7. [Database Migrations](#database-migrations)
8. [Post-Deployment Setup](#post-deployment-setup)
9. [Custom Domain](#custom-domain)
10. [Monitoring & Troubleshooting](#monitoring--troubleshooting)
11. [CI/CD with Git](#cicd-with-git)

## Prerequisites

Before deploying to Vercel, ensure you have:

- ✅ **GitHub, GitLab, or Bitbucket account** (for Git integration)
- ✅ **Vercel account** (free tier available)
- ✅ **PostgreSQL database** (Neon, Supabase, or Railway recommended)
- ✅ **OpenAI API key**
- ✅ **Git repository** with your code pushed

## Vercel Account Setup

### 1. Create Vercel Account

1. Go to [vercel.com](https://vercel.com)
2. Click **"Sign Up"**
3. Sign up with GitHub, GitLab, or Bitbucket (recommended for easy integration)

### 2. Install Vercel CLI (Optional)

For local deployment and testing:

```bash
npm install -g vercel
```

Verify installation:

```bash
vercel --version
```

## Database Setup

Vercel requires an external PostgreSQL database. Recommended providers:

### Option 1: Neon (Recommended - Serverless PostgreSQL)

1. **Sign up**: Go to [neon.tech](https://neon.tech) and create an account
2. **Create project**: Click "New Project"
3. **Get connection string**:
   - Go to your project dashboard
   - Click "Connection Details"
   - Copy the connection string (format: `postgresql://user:password@ep-xxx.region.aws.neon.tech/dbname?sslmode=require`)

**Advantages:**

- Serverless (scales automatically)
- Free tier available
- Built for serverless/edge functions
- Auto-scaling

### Option 2: Supabase

1. **Sign up**: Go to [supabase.com](https://supabase.com)
2. **Create project**: Click "New Project"
3. **Get connection string**:
   - Go to Project Settings → Database
   - Copy the connection string under "Connection string" → "URI"

**Advantages:**

- Free tier available
- Additional features (auth, storage, etc.)
- Good documentation

### Option 3: Railway

1. **Sign up**: Go to [railway.app](https://railway.app)
2. **Create PostgreSQL**: Click "New" → "Database" → "PostgreSQL"
3. **Get connection string**:
   - Click on your database
   - Go to "Variables" tab
   - Copy `DATABASE_URL`

**Advantages:**

- Simple setup
- Good free tier
- Easy to scale

### Option 4: Vercel Postgres (Beta)

1. In Vercel dashboard, go to **Storage** → **Create Database**
2. Select **Postgres**
3. Choose region and plan
4. Connection string is automatically available as environment variable

## Project Preparation

### 1. Ensure Project is Ready

Verify your project structure:

```bash
# Check if all files are present
ls -la

# Verify package.json has build script
cat package.json | grep "build"
```

### 2. Update Build Configuration (if needed)

Vercel automatically detects Next.js projects, but you can create `vercel.json` for custom configuration:

```json
{
  "buildCommand": "npm run build",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "framework": "nextjs",
  "regions": ["iad1"]
}
```

### 3. Verify Prisma Setup

Ensure `prisma/schema.prisma` is configured correctly:

```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

## Deployment Methods

### Method 1: Deploy via Vercel Dashboard (Recommended for First Time)

1. **Import Project**:
   - Go to [vercel.com/dashboard](https://vercel.com/dashboard)
   - Click **"Add New..."** → **"Project"**
   - Import your Git repository (GitHub/GitLab/Bitbucket)

2. **Configure Project**:
   - **Framework Preset**: Next.js (auto-detected)
   - **Root Directory**: `./` (or `frontend` if your repo has multiple folders)
   - **Build Command**: `npm run build` (default)
   - **Output Directory**: `.next` (default)
   - **Install Command**: `npm install` (default)

3. **Add Environment Variables** (see [Environment Variables](#environment-variables) section)

4. **Deploy**:
   - Click **"Deploy"**
   - Wait for build to complete (usually 2-5 minutes)

### Method 2: Deploy via Vercel CLI

1. **Login to Vercel**:

   ```bash
   vercel login
   ```

2. **Link Project** (first time):

   ```bash
   vercel link
   ```

   - Select or create a project
   - Choose settings (use defaults)

3. **Deploy to Preview**:

   ```bash
   vercel
   ```

4. **Deploy to Production**:
   ```bash
   vercel --prod
   ```

### Method 3: Deploy via Git Push (CI/CD)

1. **Connect Repository** (one-time setup):
   - Go to Vercel Dashboard → Your Project → Settings → Git
   - Connect your repository if not already connected

2. **Automatic Deployments**:
   - **Production**: Deploys on push to `main` or `master` branch
   - **Preview**: Deploys on push to other branches or pull requests

3. **Deploy**:
   ```bash
   git push origin main
   ```
   Vercel will automatically build and deploy!

## Environment Variables

### Required Environment Variables

Add these in **Vercel Dashboard** → **Your Project** → **Settings** → **Environment Variables**:

| Variable         | Description                                  | Example                                               |
| ---------------- | -------------------------------------------- | ----------------------------------------------------- |
| `DATABASE_URL`   | PostgreSQL connection string                 | `postgresql://user:pass@host:5432/db?sslmode=require` |
| `SESSION_SECRET` | Secret for session encryption (min 32 chars) | Generate with: `openssl rand -base64 64`              |
| `OPENAI_API_KEY` | Your OpenAI API key                          | `sk-...`                                              |
| `NODE_ENV`       | Environment mode                             | `production`                                          |

### Adding Environment Variables

1. Go to **Project Settings** → **Environment Variables**
2. Add each variable:
   - **Name**: Variable name (e.g., `DATABASE_URL`)
   - **Value**: Variable value
   - **Environment**: Select where it applies:
     - ✅ **Production**
     - ✅ **Preview** (for pull requests)
     - ✅ **Development** (for `vercel dev`)

3. Click **"Save"**

### Generate Session Secret

```bash
# Linux/macOS
openssl rand -base64 64

# Windows (PowerShell)
[Convert]::ToBase64String((1..64 | ForEach-Object { Get-Random -Minimum 0 -Maximum 256 }))

# Node.js
node -e "console.log(require('crypto').randomBytes(64).toString('base64'))"
```

### Environment Variables for Different Environments

You can set different values for Production, Preview, and Development:

- **Production**: Use production database and API keys
- **Preview**: Use staging database (optional)
- **Development**: Use local development values (for `vercel dev`)

## Database Migrations

### Option 1: Run Migrations via Vercel Build (Recommended)

Add a build script that runs migrations:

1. **Update `package.json`**:

   ```json
   {
     "scripts": {
       "build": "prisma generate && prisma migrate deploy && next build",
       "postinstall": "prisma generate"
     }
   }
   ```

2. **Vercel will automatically run migrations** during build

### Option 2: Run Migrations Manually (First Time)

1. **Install Vercel CLI**:

   ```bash
   npm install -g vercel
   ```

2. **Run migrations**:

   ```bash
   # Set environment variables locally
   export DATABASE_URL="your-database-url"

   # Run migrations
   npx prisma migrate deploy

   # Seed database (optional)
   npm run db:seed
   ```

### Option 3: Use Vercel Build Command

In Vercel Dashboard → Project Settings → General:

**Build Command**:

```bash
npm install && npx prisma generate && npx prisma migrate deploy && npm run build
```

**Install Command**:

```bash
npm install
```

### Verify Migrations

After deployment, verify tables were created:

```bash
# Connect to your database
psql $DATABASE_URL

# List tables
\dt

# Should see: User, Session, Conversation, Message, _prisma_migrations
```

## Post-Deployment Setup

### 1. Verify Deployment

1. Go to your Vercel project dashboard
2. Check **"Deployments"** tab
3. Click on the latest deployment
4. Verify it shows **"Ready"** status

### 2. Test Application

1. Open your deployment URL: `https://your-project.vercel.app`
2. Test signup/login
3. Test chat functionality
4. Verify database connections

### 3. Seed Database (Optional)

If you need initial data (admin user, etc.):

**Option A: Via Vercel CLI**:

```bash
vercel env pull .env.local
npx prisma db seed
```

**Option B: Add to build process**:

```json
{
  "scripts": {
    "build": "prisma generate && prisma migrate deploy && npm run db:seed && next build"
  }
}
```

**Option C: Manual seeding**:

```bash
# Connect to your production database
export DATABASE_URL="your-production-database-url"
npm run db:seed
```

### 4. Update Admin Password

**IMPORTANT**: Change the default admin password immediately!

1. Login with default credentials:
   - Email: `admin@example.com`
   - Password: `admin123`

2. Go to admin panel and change password

## Custom Domain

### Add Custom Domain

1. Go to **Project Settings** → **Domains**
2. Click **"Add Domain"**
3. Enter your domain (e.g., `example.com`)
4. Follow DNS configuration instructions

### DNS Configuration

Vercel will provide DNS records to add:

**For Root Domain** (`example.com`):

```
Type: A
Name: @
Value: 76.76.21.21
```

**For Subdomain** (`www.example.com`):

```
Type: CNAME
Name: www
Value: cname.vercel-dns.com
```

### SSL Certificate

- Vercel automatically provisions SSL certificates
- Certificates are managed automatically
- HTTPS is enabled by default

## Monitoring & Troubleshooting

### View Logs

1. **Vercel Dashboard**:
   - Go to your project → **Deployments**
   - Click on a deployment
   - Click **"Functions"** tab to see serverless function logs

2. **Vercel CLI**:
   ```bash
   vercel logs [deployment-url]
   ```

### Common Issues

#### Issue: Build Fails

**Error**: `Module not found` or `Cannot find module`

**Solution**:

- Check `package.json` dependencies
- Ensure all imports are correct
- Clear `.next` cache: `rm -rf .next`

**Error**: `Prisma Client not generated`

**Solution**:

- Add to build command: `prisma generate`
- Or add to `package.json`: `"postinstall": "prisma generate"`

#### Issue: Database Connection Fails

**Error**: `Can't reach database server`

**Solution**:

1. Verify `DATABASE_URL` is set correctly in Vercel
2. Check database provider allows connections from Vercel IPs
3. For Neon/Supabase: Ensure connection string includes `?sslmode=require`
4. Check database is not paused (Neon free tier pauses after inactivity)

#### Issue: Environment Variables Not Loading

**Error**: `process.env.VARIABLE is undefined`

**Solution**:

1. Verify variable is set in Vercel Dashboard
2. Ensure variable is added to correct environment (Production/Preview)
3. Redeploy after adding variables
4. For Next.js: Public variables must start with `NEXT_PUBLIC_`

#### Issue: Migrations Not Running

**Error**: Tables don't exist

**Solution**:

1. Add `prisma migrate deploy` to build command
2. Or run manually: `npx prisma migrate deploy`
3. Verify `DATABASE_URL` is correct

#### Issue: Function Timeout

**Error**: `Function exceeded maximum duration`

**Solution**:

1. Vercel Hobby plan: 10s timeout
2. Vercel Pro plan: 60s timeout
3. Optimize database queries
4. Use edge functions for faster responses
5. Consider upgrading plan

### Performance Optimization

1. **Enable Edge Functions** (if applicable):
   - Move API routes to edge runtime
   - Faster response times globally

2. **Database Connection Pooling**:
   - Use connection pooler (Neon, Supabase provide this)
   - Reduces connection overhead

3. **Caching**:
   - Use Vercel's edge caching
   - Cache static assets
   - Use ISR (Incremental Static Regeneration) for pages

## CI/CD with Git

### Automatic Deployments

Vercel automatically deploys when you push to Git:

- **Production**: `main` or `master` branch
- **Preview**: Other branches and pull requests

### Deployment Workflow

1. **Make changes** in your code
2. **Commit and push**:
   ```bash
   git add .
   git commit -m "Update feature"
   git push origin main
   ```
3. **Vercel automatically**:
   - Detects the push
   - Runs build
   - Deploys to production
   - Sends notification (if configured)

### Preview Deployments

- Every branch gets a unique preview URL
- Perfect for testing before merging
- Share preview URLs with team

### Deployment Protection

1. **Go to Project Settings** → **Git**
2. **Enable "Production Deployment Protection"**:
   - Require approval before deploying
   - Prevent accidental deployments

### Rollback

1. Go to **Deployments** tab
2. Find previous working deployment
3. Click **"..."** → **"Promote to Production"**

## Best Practices

### 1. Environment Management

- ✅ Use different databases for production and preview
- ✅ Never commit `.env` files
- ✅ Use Vercel's environment variable management
- ✅ Rotate secrets periodically

### 2. Database

- ✅ Use connection pooling
- ✅ Enable SSL connections
- ✅ Set up database backups
- ✅ Monitor database usage

### 3. Security

- ✅ Use strong `SESSION_SECRET` (64+ characters)
- ✅ Enable HTTPS (automatic on Vercel)
- ✅ Keep dependencies updated
- ✅ Review Vercel security settings

### 4. Performance

- ✅ Optimize images
- ✅ Use edge functions where possible
- ✅ Enable caching
- ✅ Monitor function execution time

### 5. Monitoring

- ✅ Set up Vercel Analytics (optional)
- ✅ Monitor error rates
- ✅ Track API usage
- ✅ Set up alerts

## Quick Start Checklist

- [ ] Create Vercel account
- [ ] Set up PostgreSQL database (Neon/Supabase/Railway)
- [ ] Push code to Git repository
- [ ] Import project to Vercel
- [ ] Add environment variables:
  - [ ] `DATABASE_URL`
  - [ ] `SESSION_SECRET`
  - [ ] `OPENAI_API_KEY`
  - [ ] `NODE_ENV=production`
- [ ] Configure build command (include Prisma migrations)
- [ ] Deploy project
- [ ] Verify deployment works
- [ ] Run database migrations
- [ ] Seed database (if needed)
- [ ] Test application
- [ ] Change default admin password
- [ ] Add custom domain (optional)
- [ ] Set up monitoring

## Support & Resources

- **Vercel Documentation**: [vercel.com/docs](https://vercel.com/docs)
- **Vercel Status**: [vercel-status.com](https://vercel-status.com)
- **Vercel Community**: [github.com/vercel/vercel/discussions](https://github.com/vercel/vercel/discussions)
- **Next.js Documentation**: [nextjs.org/docs](https://nextjs.org/docs)
- **Prisma Documentation**: [prisma.io/docs](https://prisma.io/docs)

## Troubleshooting Commands

```bash
# Check Vercel CLI version
vercel --version

# View project info
vercel inspect

# Pull environment variables locally
vercel env pull .env.local

# View deployment logs
vercel logs [deployment-url]

# Test build locally
vercel build

# Run development server with Vercel
vercel dev
```

---

**Last Updated**: December 2025  
**Vercel Compatibility**: Next.js 16+  
**Recommended Database**: Neon, Supabase, or Railway
