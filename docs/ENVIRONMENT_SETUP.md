# Environment Setup Guide

## Overview

This guide explains how to configure the required environment variables for Socially.

## Required Environment Variables

The application requires the following environment variables in a `.env` file:

```env
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...
DATABASE_URL=postgresql://user:password@host:port/database
UPLOADTHING_TOKEN=...
```

## Environment Variable Details

### NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY

**Purpose:** Clerk public key for client-side authentication.

**Prefix:** `NEXT_PUBLIC_` - Makes the variable available on the client side.

**Format:** Starts with `pk_test_` (development) or `pk_live_` (production).

**How to Obtain:**
1. Create account at https://clerk.com
2. Create a new application
3. Navigate to API Keys section
4. Copy the Publishable Key

**Example:**
```
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_abc123xyz456
```

### CLERK_SECRET_KEY

**Purpose:** Clerk secret key for server-side authentication.

**Prefix:** No prefix - Kept server-side only (not exposed to client).

**Format:** Starts with `sk_test_` (development) or `sk_live_` (production).

**How to Obtain:**
1. Navigate to Clerk application dashboard
2. Go to API Keys section
3. Copy the Secret Key

**Security:** Never commit this key to Git. Keep it secret.

**Example:**
```
CLERK_SECRET_KEY=sk_test_xyz789abc123
```

### DATABASE_URL

**Purpose:** PostgreSQL database connection string.

**Format:** Standard PostgreSQL connection string format.

**Format Template:**
```
postgresql://username:password@host:port/database_name
```

**How to Obtain:**
- **Local PostgreSQL:** `postgresql://postgres:password@localhost:5432/socially`
- **Supabase:** Get from Supabase project settings
- **Neon:** Get from Neon dashboard
- **Railway:** Get from Railway service settings

**Local Example:**
```
DATABASE_URL=postgresql://postgres:mypassword@localhost:5432/socially
```

**Supabase Example:**
```
DATABASE_URL=postgresql://postgres:[YOUR-PASSWORD]@db.[PROJECT-REF].supabase.co:5432/postgres
```

**Security:** Never commit this to Git. Contains database credentials.

### UPLOADTHING_TOKEN

**Purpose:** UploadThing API token for file uploads.

**Format:** Token string provided by UploadThing.

**How to Obtain:**
1. Create account at https://uploadthing.com
2. Create a new app
3. Navigate to API Keys section
4. Copy the API token

**Example:**
```
UPLOADTHING_TOKEN=sk_live_abc123xyz456
```

**Security:** Never commit this to Git. Keep it secret.

## Creating the .env File

### Step 1: Create .env File

Create a new file named `.env` in the project root:

```bash
# In project root
touch .env
```

### Step 2: Add Environment Variables

Add the required variables to `.env`:

```env
# Clerk Authentication
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=your_clerk_publishable_key_here
CLERK_SECRET_KEY=your_clerk_secret_key_here

# Database
DATABASE_URL=your_database_url_here

# File Storage
UPLOADTHING_TOKEN=your_uploadthing_token_here
```

### Step 3: Verify .gitignore

Ensure `.env` is in `.gitignore` to prevent committing secrets:

```gitignore
# local env files
.env*.local
.env
```

**Current Status:** The `.gitignore` already excludes `.env` files. ✅

## Clerk Setup

### Step 1: Create Clerk Account

1. Go to https://clerk.com
2. Click "Sign Up"
3. Sign up with email, Google, or GitHub

### Step 2: Create Application

1. Navigate to Clerk dashboard
2. Click "Add Application"
3. Choose "Next.js" as framework
4. Give your application a name (e.g., "Socially")

### Step 3: Configure Authentication

**Sign Up Methods:**
- Email/password (recommended)
- Optional: Google, GitHub, etc.

**Email Verification:**
- Recommended: Require email verification
- Customize email template if desired

**User Attributes:**
- Enable: First name, Last name, Username, Profile image
- These fields are synced to local database

### Step 4: Get API Keys

1. Navigate to your application in Clerk dashboard
2. Go to "API Keys" section
3. Copy:
   - Publishable Key → `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`
   - Secret Key → `CLERK_SECRET_KEY`

### Step 5: Add to .env

```env
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...
```

## PostgreSQL Setup

### Option 1: Local PostgreSQL

#### Install PostgreSQL

**Windows:**
1. Download from https://www.postgresql.org/download/windows/
2. Run installer
3. Set password for postgres user
4. Complete installation

**macOS:**
```bash
brew install postgresql
brew services start postgresql
```

**Linux (Ubuntu):**
```bash
sudo apt-get install postgresql postgresql-contrib
sudo service postgresql start
```

#### Create Database

```bash
# Access PostgreSQL
psql -U postgres

# Create database
CREATE DATABASE socially;

# Exit
\q
```

#### Add to .env

```env
DATABASE_URL=postgresql://postgres:your_password@localhost:5432/socially
```

### Option 2: Docker PostgreSQL

#### Run PostgreSQL Container

```bash
docker run --name socially-db \
  -e POSTGRES_PASSWORD=mypassword \
  -e POSTGRES_DB=socially \
  -p 5432:5432 \
  -d postgres
```

#### Add to .env

```env
DATABASE_URL=postgresql://postgres:mypassword@localhost:5432/socially
```

### Option 3: Supabase (Recommended for Production)

#### Create Supabase Project

1. Go to https://supabase.com
2. Click "Start your project"
3. Sign up or log in
4. Create new organization
5. Create new project
6. Wait for database to be provisioned

#### Get Connection String

1. Navigate to project settings
2. Go to "Database" section
3. Find "Connection string"
4. Copy "URI" format
5. Replace `[YOUR-PASSWORD]` with your database password

#### Add to .env

```env
DATABASE_URL=postgresql://postgres:[YOUR-PASSWORD]@db.[PROJECT-REF].supabase.co:5432/postgres
```

### Option 4: Neon

#### Create Neon Project

1. Go to https://neon.tech
2. Click "Create a project"
3. Sign up or log in
4. Choose region
5. Create project
6. Copy connection string

#### Add to .env

```env
DATABASE_URL=postgres://user:password@ep-cool-region.aws.neon.tech/dbname?sslmode=require
```

### Option 5: Railway

#### Create Railway Project

1. Go to https://railway.app
2. Click "New Project"
3. Select "Provision PostgreSQL"
4. Wait for database to be created
5. Copy connection string

#### Add to .env

```env
DATABASE_URL=postgresql://user:password@host.railway.app:5432/railway
```

## UploadThing Setup

### Step 1: Create UploadThing Account

1. Go to https://uploadthing.com
2. Click "Start for free"
3. Sign up with email, Google, or GitHub

### Step 2: Create App

1. Navigate to UploadThing dashboard
2. Click "New App"
3. Give your app a name (e.g., "Socially")
4. Choose "Next.js" as framework

### Step 3: Configure File Types

**Allowed File Types:**
- Images (JPEG, PNG, GIF, WebP, etc.)

**File Size Limit:**
- Maximum 4MB (configured in code)

### Step 4: Get API Token

1. Navigate to your app in UploadThing dashboard
2. Go to "API Keys" section
3. Copy the API token

### Step 5: Add to .env

```env
UPLOADTHING_TOKEN=sk_live_...
```

## Example .env File

```env
# Clerk Authentication
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_abc123xyz456
CLERK_SECRET_KEY=sk_test_xyz789abc123

# Database (Local PostgreSQL Example)
DATABASE_URL=postgresql://postgres:mypassword@localhost:5432/socially

# File Storage
UPLOADTHING_TOKEN=sk_live_def456ghi789
```

## Environment-Specific Files

### Development (.env.local)

For local development, you can use `.env.local`:

```env
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...
DATABASE_URL=postgresql://postgres:password@localhost:5432/socially
UPLOADTHING_TOKEN=sk_test_...
```

### Production (.env.production)

For production deployment, use production keys:

```env
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_live_...
CLERK_SECRET_KEY=sk_live_...
DATABASE_URL=postgresql://user:password@production-host:5432/socially
UPLOADTHING_TOKEN=sk_live_...
```

**Note:** Production environment variables are typically configured in your hosting platform (e.g., Vercel environment variables), not in a `.env` file.

## Verifying Environment Variables

### Check Variables are Loaded

In development, Next.js automatically loads `.env` files.

To verify, you can add temporary logging:

```typescript
// In any server component
console.log("DATABASE_URL:", process.env.DATABASE_URL);
```

### Test Database Connection

After setting up DATABASE_URL, test the connection:

```bash
npx prisma db pull
```

If successful, Prisma will connect to the database.

### Test Clerk Connection

Start the development server and visit the application. If Clerk is configured correctly, you should see sign in/sign up buttons.

## Common Issues

### Issue: Environment variables not loading

**Solution:**
- Ensure `.env` file is in project root
- Restart development server after adding variables
- Check variable names match exactly (case-sensitive)

### Issue: Clerk authentication not working

**Solution:**
- Verify NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY starts with `pk_`
- Verify CLERK_SECRET_KEY starts with `sk_`
- Ensure ClerkProvider wraps the application
- Check Clerk dashboard for application status

### Issue: Database connection failed

**Solution:**
- Verify DATABASE_URL format is correct
- Ensure PostgreSQL is running
- Check database name exists
- Verify username and password are correct
- Test connection with `npx prisma db pull`

### Issue: UploadThing upload failed

**Solution:**
- Verify UPLOADTHING_TOKEN is correct
- Ensure user is authenticated
- Check file size is under 4MB
- Verify file is an image type

## Security Best Practices

### Do

- Keep all secret keys in `.env` files
- Add `.env` to `.gitignore`
- Use different keys for development and production
- Rotate keys if compromised
- Use strong passwords for database

### Don't

- Don't commit `.env` files to Git
- Don't share secret keys publicly
- Don't use production keys in development
- Don't hardcode credentials in source code
- Don't use weak passwords

## Production Environment Variables

### Vercel (Recommended)

For Vercel deployment:

1. Go to Vercel project settings
2. Navigate to "Environment Variables"
3. Add each variable:
   - `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`
   - `CLERK_SECRET_KEY`
   - `DATABASE_URL`
   - `UPLOADTHING_TOKEN`
4. Select appropriate environment (Production, Preview, Development)
5. Redeploy to apply changes

### Other Platforms

For other hosting platforms, configure environment variables according to their documentation.

## Next Steps

After configuring environment variables:

1. Run database migrations: `npx prisma migrate dev --name init`
2. Install dependencies: `npm install`
3. Start development server: `npm run dev`
4. Test authentication flow
5. Test file upload flow

For detailed setup instructions, see [LOCAL_DEVELOPMENT.md](./LOCAL_DEVELOPMENT.md).

## Resources

- **Clerk:** https://clerk.com/docs
- **PostgreSQL:** https://www.postgresql.org/docs/
- **Supabase:** https://supabase.com/docs
- **Neon:** https://neon.tech/docs
- **UploadThing:** https://uploadthing.com/docs
