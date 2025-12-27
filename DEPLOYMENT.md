# Deployment Guide - AI Assistant Application

This guide provides complete instructions for deploying and running the AI Assistant application.

## Table of Contents

1. [System Requirements](#system-requirements)
2. [Environment Variables](#environment-variables)
3. [Database Setup](#database-setup)
4. [Installation](#installation)
5. [Running the Application](#running-the-application)
6. [Production Deployment](#production-deployment)
7. [Troubleshooting](#troubleshooting)
8. [Security Considerations](#security-considerations)

## System Requirements

### Prerequisites

- **Node.js**: v18 or higher
- **PostgreSQL**: v14 or higher
- **npm**: v8 or higher (comes with Node.js)
- **Git**: Latest version

### Operating Systems

- Linux (Ubuntu 20.04+, Debian 11+, CentOS 8+)
- macOS (10.15+)
- Windows 10/11 (with WSL2 recommended)

## Environment Variables

Create a `.env` file in the project root with the following variables:

```env
# Database Connection
DATABASE_URL="postgresql://username:password@host:5432/database_name?schema=public"

# Session Security
SESSION_SECRET="your-secure-random-string-minimum-32-characters-long"

# OpenAI API
OPENAI_API_KEY="sk-your-openai-api-key-here"

# Optional: Cloud Assistant-UI (for advanced features)
# NEXT_PUBLIC_ASSISTANT_BASE_URL=

# Environment
NODE_ENV="production"
```

### Environment Variable Details

#### DATABASE_URL

PostgreSQL connection string format:
```
postgresql://[username]:[password]@[host]:[port]/[database]?schema=public
```

**Examples:**

Local PostgreSQL:
```
DATABASE_URL="postgresql://postgres:mypassword@localhost:5432/assistant_db?schema=public"
```

Cloud providers:

**Neon:**
```
DATABASE_URL="postgresql://user:password@ep-example-123456.us-east-1.aws.neon.tech/neondb?sslmode=require"
```

**Supabase:**
```
DATABASE_URL="postgresql://postgres:password@db.projectref.supabase.co:5432/postgres"
```

**Railway:**
```
DATABASE_URL="postgresql://postgres:password@containers-us-west-123.railway.app:5432/railway"
```

#### SESSION_SECRET

- **Minimum**: 32 characters
- **Recommended**: 64 characters
- **Generation**: Use a cryptographically secure random string

Generate securely:
```bash
# Linux/macOS
openssl rand -base64 64

# Node.js
node -e "console.log(require('crypto').randomBytes(64).toString('base64'))"
```

#### OPENAI_API_KEY

Get your API key from: https://platform.openai.com/api-keys

#### NODE_ENV

- `development` - Development mode with hot reload and detailed errors
- `production` - Production mode with optimizations and minimal error exposure

## Database Setup

### Option 1: Local PostgreSQL

#### Install PostgreSQL

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

**macOS (using Homebrew):**
```bash
brew install postgresql@16
brew services start postgresql@16
```

**Windows:**
Download and install from: https://www.postgresql.org/download/windows/

#### Create Database

```bash
# Connect to PostgreSQL
sudo -u postgres psql

# Create database and user
CREATE DATABASE assistant_db;
CREATE USER assistant_user WITH PASSWORD 'your_secure_password';
GRANT ALL PRIVILEGES ON DATABASE assistant_db TO assistant_user;

# Grant schema privileges (PostgreSQL 15+)
\c assistant_db
GRANT ALL ON SCHEMA public TO assistant_user;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO assistant_user;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO assistant_user;

# Exit
\q
```

### Option 2: Cloud Database (Recommended for Production)

#### Neon (Serverless PostgreSQL)

1. Sign up at https://neon.tech
2. Create a new project
3. Copy the connection string
4. Update `.env` with the connection string

#### Supabase

1. Sign up at https://supabase.com
2. Create a new project
3. Go to Project Settings → Database
4. Copy the connection string
5. Update `.env` with the connection string

#### Railway

1. Sign up at https://railway.app
2. Create a new PostgreSQL database
3. Copy the connection string from variables
4. Update `.env` with the connection string

## Installation

### 1. Clone Repository

```bash
git clone <repository-url>
cd assistant/frontend
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Configure Environment

```bash
# Copy example env file
cp .env.example .env

# Edit .env with your values
nano .env  # or use your preferred editor
```

### 4. Database Migration

```bash
# Generate Prisma Client
npx prisma generate

# Run migrations to create tables
npx prisma migrate deploy

# Seed database with default admin user
npm run db:seed
```

**Default Admin Credentials:**
- Email: `admin@example.com`
- Password: `admin123`

**⚠️ IMPORTANT:** Change the admin password immediately after first login in production!

## Running the Application

### Development Mode

```bash
npm run dev
```

Access at: http://localhost:3000

### Production Mode

```bash
# Build the application
npm run build

# Start production server
npm run start
```

Access at: http://localhost:3000 (or configured port)

### Using Process Manager (PM2)

```bash
# Install PM2 globally
npm install -g pm2

# Build the app
npm run build

# Start with PM2
pm2 start npm --name "ai-assistant" -- start

# Save PM2 configuration
pm2 save

# Setup PM2 to start on system boot
pm2 startup
```

**PM2 Commands:**
```bash
pm2 list                 # List all processes
pm2 logs ai-assistant    # View logs
pm2 restart ai-assistant # Restart app
pm2 stop ai-assistant    # Stop app
pm2 delete ai-assistant  # Remove from PM2
```

## Production Deployment

### Option 1: Vercel (Recommended for Next.js)

1. Install Vercel CLI:
```bash
npm install -g vercel
```

2. Deploy:
```bash
vercel --prod
```

3. Configure environment variables in Vercel dashboard

4. Connect your PostgreSQL database (Neon, Supabase, etc.)

### Option 2: Docker Deployment

Create `Dockerfile`:
```dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./
COPY prisma ./prisma/

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Generate Prisma Client
RUN npx prisma generate

# Build Next.js app
RUN npm run build

# Expose port
EXPOSE 3000

# Start application
CMD ["npm", "run", "start"]
```

Create `docker-compose.yml`:
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - SESSION_SECRET=${SESSION_SECRET}
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - NODE_ENV=production
    depends_on:
      - db
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    environment:
      - POSTGRES_DB=assistant_db
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  postgres_data:
```

Deploy:
```bash
docker-compose up -d
```

### Option 3: Traditional Server (Ubuntu/Debian)

```bash
# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install PostgreSQL (if not using cloud)
sudo apt install postgresql postgresql-contrib

# Clone and setup application
git clone <repository-url>
cd assistant/frontend
npm install
npm run build

# Setup PM2
sudo npm install -g pm2
pm2 start npm --name "ai-assistant" -- start
pm2 save
pm2 startup

# Setup Nginx as reverse proxy
sudo apt install nginx

# Create Nginx config
sudo nano /etc/nginx/sites-available/ai-assistant
```

Nginx configuration:
```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Enable and start:
```bash
sudo ln -s /etc/nginx/sites-available/ai-assistant /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

# Setup SSL with Certbot
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
```

## Troubleshooting

### Database Connection Issues

**Error: Connection refused**
```bash
# Check if PostgreSQL is running
sudo systemctl status postgresql

# Check connection string format
# Ensure host, port, username, password are correct
```

**Error: Authentication failed**
```bash
# Verify credentials
psql -U username -h host -d database

# Check pg_hba.conf for connection permissions
sudo nano /etc/postgresql/16/main/pg_hba.conf
```

### Migration Issues

**Error: Migration already applied**
```bash
# Reset migration state
npx prisma migrate reset

# WARNING: This deletes all data!
```

**Error: Schema out of sync**
```bash
# Generate new client
npx prisma generate

# Apply migrations
npx prisma migrate deploy
```

### Build Errors

**Error: Out of memory**
```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run build
```

**Error: Module not found**
```bash
# Clear cache and reinstall
rm -rf node_modules package-lock.json
npm install
```

### Runtime Errors

**Error: OPENAI_API_KEY not set**
- Verify `.env` file exists
- Check environment variable is loaded
- Restart the application

**Error: Session secret not configured**
- Set `SESSION_SECRET` in `.env`
- Must be at least 32 characters

## Security Considerations

### Production Checklist

- [ ] Change default admin password
- [ ] Use strong, unique SESSION_SECRET (64+ characters)
- [ ] Enable HTTPS/SSL certificates
- [ ] Set `NODE_ENV=production`
- [ ] Use cloud database with SSL connection
- [ ] Enable database backups
- [ ] Set up monitoring and logging
- [ ] Configure firewall rules
- [ ] Keep dependencies updated
- [ ] Implement rate limiting (recommended)

### Sensitive Data

**Never commit to version control:**
- `.env` file
- Database credentials
- API keys
- Session secrets

**Always use:**
- Environment variables for secrets
- `.gitignore` to exclude sensitive files
- Encrypted connection strings for databases

### API Key Security

- Never expose OpenAI API key client-side
- All AI API calls are server-side only
- Rotate API keys periodically
- Monitor API usage for anomalies

### Session Security

- Sessions stored server-side in database
- HTTP-only cookies (not accessible via JavaScript)
- Secure flag enabled in production
- SameSite=Lax to prevent CSRF
- 7-day expiration (configurable)

### User Data

- Passwords hashed with bcrypt (10 rounds)
- Conversations stored per user
- Users can only access their own data
- Admin panel requires admin role

## Application Features

### Authentication System

- Email + password authentication
- Server-side session management
- User roles: admin and user
- User status: active, suspended, disabled
- Admin panel for user management

### Conversation Management

- Server-side conversation persistence
- Conversation list with date/time
- Click to reopen conversations
- "New Chat" button for fresh context
- Auto-generated conversation titles
- Delete conversations
- Context truncation (last 20 messages)

### AI Response Format

All AI responses follow this structured format:

1. **Résumé** (2 lines)
2. **Analyse technique**
3. **Références normatives**
4. **Logique / Schéma** (texte)
5. **Solutions / Recommandations**
6. **Points de vigilance**
7. **Version courte** (if relevant)

This format is enforced via system prompts.

### Logging

- Request logging with timestamps
- Error logging with stack traces
- Conversation creation/deletion logs
- User authentication logs
- All logs include user IDs for traceability

## Support

For issues or questions:

1. Check the [Troubleshooting](#troubleshooting) section
2. Review application logs:
   ```bash
   # PM2 logs
   pm2 logs ai-assistant

   # Docker logs
   docker-compose logs -f

   # Direct logs (if running with npm)
   # Check terminal output
   ```

3. Check database connection:
   ```bash
   npx prisma studio
   ```

4. Verify environment variables:
   ```bash
   node -e "console.log(process.env.DATABASE_URL ? 'DB configured' : 'DB not configured')"
   ```

## License

[Your License Here]

---

**Last Updated:** December 2025
**Version:** 1.0.0
