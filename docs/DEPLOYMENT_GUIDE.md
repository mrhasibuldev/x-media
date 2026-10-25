# Deployment Guide

## Overview

This guide explains how to deploy Socially to production.

## Prerequisites

Before deploying, ensure you have:

1. **Completed Testing:** Application tested locally
2. **Environment Variables:** Production credentials configured
3. **Database:** Production PostgreSQL database
4. **Clerk:** Production Clerk application
5. **UploadThing:** Production UploadThing application
6. **Domain:** Custom domain (optional but recommended)
7. **Git Repository:** Code pushed to GitHub

---

## Recommended Platform: Vercel

Vercel is the recommended deployment platform for Next.js applications.

**Why Vercel:**
- Built for Next.js
- Automatic HTTPS
- Global CDN
- Zero configuration deployment
- Environment variable management
- Preview deployments
- Free tier available

---

## Step 1: Prepare for Deployment

### 1.1 Build Application Locally

**Command:**
```bash
npm run build
```

**Purpose:** Verify application builds without errors.

**Expected Output:**
```
✓ Compiled successfully
✓ Linting and checking validity of types
✓ Collecting page data
✓ Generating static pages
✓ Finalizing page optimization
```

**If Build Fails:**
- Fix TypeScript errors
- Fix linting errors
- Fix build errors
- Retry build

---

### 1.2 Verify Environment Variables

Create `.env.production` file (local only, not committed):

```env
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_live_...
CLERK_SECRET_KEY=sk_live_...
DATABASE_URL=postgresql://user:password@host:port/database
UPLOADTHING_TOKEN=sk_live_...
```

**Note:** Production environment variables will be configured in Vercel, not in this file.

---

### 1.3 Push Code to GitHub

**If not already done:**
```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/mrhasibuldev/x-media.git
git branch -M main
git push -u origin main
```

---

## Step 2: Deploy to Vercel

### 2.1 Create Vercel Account

1. Go to https://vercel.com
2. Click "Sign Up"
3. Sign up with GitHub, GitLab, or email
4. Verify email

---

### 2.2 Import Repository

1. Log in to Vercel
2. Click "Add New" → "Project"
3. Select GitHub (or your Git provider)
4. Authorize Vercel to access your repository
5. Select `x-media` repository
6. Click "Import"

---

### 2.3 Configure Project

**Project Settings:**

**Framework Preset:** Next.js (auto-detected)

**Root Directory:** `./` (leave as default)

**Build Command:** `npm run build` (auto-detected)

**Output Directory:** `.next` (auto-detected)

**Install Command:** `npm ci` (auto-detected)

**Click "Deploy"**

---

### 2.4 Configure Environment Variables

**After initial deployment:**

1. Go to project settings in Vercel
2. Navigate to "Settings" → "Environment Variables"
3. Add each variable:

**Variables to Add:**

| Name | Value | Environment |
|------|-------|-------------|
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | Your Clerk publishable key | Production, Preview, Development |
| `CLERK_SECRET_KEY` | Your Clerk secret key | Production, Preview, Development |
| `DATABASE_URL` | Your PostgreSQL connection string | Production, Preview, Development |
| `UPLOADTHING_TOKEN` | Your UploadThing token | Production, Preview, Development |

**Note:** Use production keys for Production environment, test keys for Preview/Development.

---

### 2.5 Redeploy

After adding environment variables:

1. Go to "Deployments" tab
2. Click "Redeploy"
3. Wait for deployment to complete

---

## Step 3: Configure Production Database

### Option 1: Supabase (Recommended)

#### 3.1 Create Supabase Project

1. Go to https://supabase.com
2. Click "New Project"
3. Sign up or log in
4. Create new organization
5. Create new project
6. Wait for database to be provisioned

#### 3.2 Get Connection String

1. Navigate to project settings
2. Go to "Database" section
3. Find "Connection string"
4. Copy "URI" format
5. Replace `[YOUR-PASSWORD]` with your database password

#### 3.3 Run Migrations

**Option A: Using Vercel CLI:**
```bash
# Install Vercel CLI
npm i -g vercel

# Login to Vercel
vercel login

# Run migration in production
vercel env pull .env.production
npx prisma migrate deploy
```

**Option B: Using Direct Connection:**
```bash
# Set DATABASE_URL
export DATABASE_URL="postgresql://postgres:[YOUR-PASSWORD]@db.[PROJECT-REF].supabase.co:5432/postgres"

# Run migrations
npx prisma migrate deploy
```

---

### Option 2: Neon

#### 3.1 Create Neon Project

1. Go to https://neon.tech
2. Click "Create a project"
3. Sign up or log in
4. Choose region
5. Create project
6. Copy connection string

#### 3.2 Run Migrations

```bash
export DATABASE_URL="postgres://user:password@ep-cool-region.aws.neon.tech/dbname?sslmode=require"
npx prisma migrate deploy
```

---

### Option 3: Railway

#### 3.1 Create Railway Project

1. Go to https://railway.app
2. Click "New Project"
3. Select "Provision PostgreSQL"
4. Wait for database to be created
5. Copy connection string

#### 3.2 Run Migrations

```bash
export DATABASE_URL="postgresql://user:password@host.railway.app:5432/railway"
npx prisma migrate deploy
```

---

## Step 4: Configure Clerk for Production

### 4.1 Update Clerk Application

1. Go to Clerk dashboard
2. Navigate to your application
3. Go to "API Keys"
4. Copy production keys:
   - Publishable Key → `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`
   - Secret Key → `CLERK_SECRET_KEY`

### 4.2 Configure Allowed Origins

1. In Clerk dashboard
2. Go to "Domains"
3. Add your production domain
4. Add Vercel preview URL

### 4.3 Update Environment Variables

Add production Clerk keys to Vercel environment variables.

---

## Step 5: Configure UploadThing for Production

### 5.1 Update UploadThing App

1. Go to UploadThing dashboard
2. Navigate to your app
3. Go to "API Keys"
4. Copy production token

### 5.2 Configure Allowed Domains

1. In UploadThing dashboard
2. Configure allowed origins
3. Add your production domain
4. Add Vercel preview URL

### 5.3 Update Environment Variables

Add production UploadThing token to Vercel environment variables.

---

## Step 6: Configure Custom Domain (Optional)

### 6.1 Purchase Domain

Purchase domain from:
- Namecheap
- GoDaddy
- Google Domains
- Cloudflare Registrar

### 6.2 Add Domain to Vercel

1. Go to Vercel project settings
2. Navigate to "Domains"
3. Click "Add Domain"
4. Enter your domain name
5. Click "Add"

### 6.3 Update DNS Records

Vercel will provide DNS records to add to your domain registrar.

**Typical Records:**
```
A Record: @ → 76.76.21.21
CNAME Record: www → cname.vercel-dns.com
```

### 6.4 Enable SSL

Vercel automatically provisions SSL certificates for custom domains.

---

## Step 7: Verify Deployment

### 7.1 Check Deployment Status

1. Go to Vercel "Deployments" tab
2. Verify latest deployment shows "Ready"
3. Click deployment to view logs

### 7.2 Test Application

1. Open production URL
2. Test sign up/sign in
3. Test create post
4. Test file upload
5. Test notifications
6. Test profile editing

### 7.3 Check Console Errors

1. Open browser DevTools
2. Check console for errors
3. Check network tab for failed requests

---

## Step 8: Monitor and Maintain

### 8.1 Set Up Monitoring

**Vercel Analytics:**
- Enabled by default
- View in Vercel dashboard
- Monitor page views, performance

**Error Monitoring:**
- Consider integrating Sentry or similar
- Track production errors
- Set up alerts

### 8.2 Regular Maintenance

**Weekly:**
- Check deployment logs
- Monitor error rates
- Review database usage

**Monthly:**
- Update dependencies
- Review security advisories
- Check storage usage
- Review costs

---

## Alternative Deployment Platforms

### Netlify

**Pros:**
- Easy deployment
- Good free tier
- Automatic HTTPS

**Cons:**
- Not optimized for Next.js (though supported)
- Different build process

**Steps:**
1. Connect Netlify to GitHub
2. Configure build settings
3. Add environment variables
4. Deploy

---

### Railway

**Pros:**
- Simple deployment
- Includes database
- Good for small projects

**Cons:**
- Not as optimized for Next.js
- Limited free tier

**Steps:**
1. Connect Railway to GitHub
2. Configure build settings
3. Add environment variables
4. Deploy

---

### Self-Hosted (VPS)

**Pros:**
- Full control
- Potentially lower cost at scale
- Custom configuration

**Cons:**
- Requires DevOps knowledge
- Must manage SSL
- Must handle scaling
- More maintenance

**Steps:**
1. Set up VPS (DigitalOcean, AWS EC2, etc.)
2. Install Node.js
3. Install PostgreSQL
4. Clone repository
5. Install dependencies
6. Build application
7. Configure PM2 or similar
8. Configure Nginx
9. Configure SSL (Let's Encrypt)
10. Start application

---

## Deployment Checklist

### Pre-Deployment

- [ ] Application tested locally
- [ ] Build succeeds locally
- [ ] Database migrations created
- [ ] Production database ready
- [ ] Production Clerk app configured
- [ ] Production UploadThing app configured
- [ ] Environment variables documented
- [ ] Code pushed to GitHub

### Deployment

- [ ] Repository imported to Vercel
- [ ] Build configuration verified
- [ ] Environment variables added
- [ ] Database migrations deployed
- [ ] Deployment successful
- [ ] Custom domain configured (optional)
- [ ] SSL enabled

### Post-Deployment

- [ ] Application accessible
- [ ] Authentication working
- [ ] Database connected
- [ ] File uploads working
- [ ] All features tested
- [ ] Monitoring configured
- [ ] Error tracking set up

---

## Troubleshooting

### Common Issues

**Issue: Build fails on Vercel**

**Solution:**
- Check build logs for errors
- Ensure all dependencies are in package.json
- Verify TypeScript version
- Check environment variables

**Issue: Database connection fails**

**Solution:**
- Verify DATABASE_URL is correct
- Check database is accessible from Vercel
- Ensure SSL is enabled for remote databases
- Check database credentials

**Issue: Environment variables not loading**

**Solution:**
- Verify variable names match exactly
- Check environment (Production vs Preview)
- Redeploy after adding variables
- Check Vercel logs

**Issue: Clerk authentication not working**

**Solution:**
- Verify Clerk keys are correct
- Check allowed origins in Clerk dashboard
- Ensure ClerkProvider wraps application
- Check middleware configuration

**Issue: File uploads failing**

**Solution:**
- Verify UploadThing token is correct
- Check file size limits
- Ensure user is authenticated
- Check UploadThing dashboard for errors

---

## Cost Considerations

### Vercel

**Free Tier:**
- 100GB bandwidth per month
- Unlimited deployments
- 1000 serverless function invocations per month
- 10GB logs per month

**Pro Tier ($20/month):**
- 1TB bandwidth per month
- Unlimited serverless function invocations
- 100GB logs per month
- Priority support

### Database Costs

**Supabase:**
- Free tier: 500MB database, 1GB file storage
- Pro tier: $25/month for 8GB database, 1GB file storage

**Neon:**
- Free tier: 0.5GB storage, 200 hours compute
- Pro tier: $19/month for 32GB storage, 200 hours compute

**Railway:**
- Free tier: $5 credit per month
- Pro tier: $5/month per service

### UploadThing

**Free Tier:**
- 2GB storage
- 10GB bandwidth per month

**Pro Tier:**
- Pricing based on storage and bandwidth

**Estimated Monthly Cost:**
- Free tier: $0 (within limits)
- Pro tier: $20-$60/month (Vercel + database + UploadThing)

---

## Scaling Considerations

### When to Scale

**Signs you need to scale:**
- Slow page load times
- High database usage
- Frequent timeouts
- High error rates

### Scaling Options

**Vercel:**
- Upgrade to Pro tier
- Increase serverless function limits
- Use Edge functions for static content

**Database:**
- Upgrade database plan
- Add read replicas
- Use connection pooling
- Implement caching

**UploadThing:**
- Upgrade storage plan
- Implement CDN caching
- Optimize image sizes

---

## Security in Production

### HTTPS

**Status:** Automatic with Vercel and custom domains.

**Verification:**
- Check for lock icon in browser
- Ensure all resources load over HTTPS

### Environment Variables

**Best Practices:**
- Never commit secrets to Git
- Use different keys for dev/prod
- Rotate keys regularly
- Use Vercel's environment variable management

### Firewall

**Vercel:** Managed by Vercel

**Self-Hosted:** Configure firewall rules to restrict access

### Monitoring

**Set up alerts for:**
- High error rates
- Failed deployments
- Database connection failures
- High response times

---

## Rollback Procedure

### Vercel Rollback

**If deployment fails:**

1. Go to "Deployments" tab
2. Find previous successful deployment
3. Click "..."
4. Select "Promote to Production"

**Or use Git:**
```bash
# Revert to previous commit
git revert HEAD
git push
```

### Database Rollback

**If migration fails:**
```bash
# Rollback migration
npx prisma migrate resolve --rolled-back <migration-name>
```

**Or restore from backup:**
```bash
psql socially < backup.sql
```

---

## Next Steps

1. Complete pre-deployment checklist
2. Deploy to Vercel
3. Configure production database
4. Configure production services
5. Test all features
6. Set up monitoring
7. Configure custom domain (optional)

## Resources

- **Vercel Documentation:** https://vercel.com/docs
- **Supabase Documentation:** https://supabase.com/docs
- **Neon Documentation:** https://neon.tech/docs
- **Railway Documentation:** https://docs.railway.app
- **Clerk Documentation:** https://clerk.com/docs
- **UploadThing Documentation:** https://uploadthing.com/docs
