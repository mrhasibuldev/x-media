# Local Development Guide

## Overview

This guide explains how to set up and run Socially for local development.

## Prerequisites

Before starting, ensure you have:

1. **Node.js:** Version 18 or higher
   - Download from https://nodejs.org
   - Verify with `node --version`

2. **PostgreSQL Database:** A running PostgreSQL instance
   - Local installation OR
   - Cloud database (Supabase, Neon, Railway)
   - See [ENVIRONMENT_SETUP.md](./ENVIRONMENT_SETUP.md) for options

3. **Clerk Account:** Free account at https://clerk.com
   - For authentication

4. **UploadThing Account:** Free account at https://uploadthing.com
   - For file uploads

5. **Git:** For version control (optional but recommended)
   - Download from https://git-scm.com

## Installation Steps

### Step 1: Clone or Extract Project

If you have the project as a zip file:
1. Extract to desired location
2. Navigate to project directory

If cloning from Git:
```bash
git clone <repository-url>
cd nextjs-course-master
```

### Step 2: Install Dependencies

```bash
npm install
```

**What this does:**
- Installs all dependencies from `package.json`
- Runs `prisma generate` automatically (postinstall script)
- Creates `node_modules/` directory

**Estimated time:** 2-5 minutes

**Common Issues:**
- If `npm install` fails, try clearing cache: `npm cache clean --force`
- If permissions error, try with sudo (Linux/macOS) or run as administrator (Windows)

### Step 3: Configure Environment Variables

Create a `.env` file in the project root with the following variables:

```env
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=your_clerk_publishable_key
CLERK_SECRET_KEY=your_clerk_secret_key
DATABASE_URL=your_database_url
UPLOADTHING_TOKEN=your_uploadthing_token
```

**For detailed instructions on obtaining these values, see [ENVIRONMENT_SETUP.md](./ENVIRONMENT_SETUP.md).**

### Step 4: Run Database Migrations

**CRITICAL:** The database schema exists but migrations have not been generated.

```bash
npx prisma migrate dev --name init
```

**What this does:**
- Creates `prisma/migrations/` directory
- Generates initial migration SQL
- Applies migration to database
- Creates all tables and indexes

**Expected Output:**
```
✔ Generated Prisma Client
✔ The following migration has been created and applied from new schema

migrations/
  └─ 20240101000000_init/
    └─ migration.sql
```

**Troubleshooting:**
- If "Can't reach database server", verify DATABASE_URL is correct
- If "Table already exists", drop and recreate database
- If permissions error, check database user permissions

### Step 5: Verify Database Setup

```bash
npx prisma studio
```

This opens Prisma Studio (GUI) to view database contents.

**Expected:** Empty database with tables: User, Post, Comment, Like, Follows, Notification

Press `Ctrl+C` to close Prisma Studio.

### Step 6: Start Development Server

```bash
npm run dev
```

**What this does:**
- Starts Next.js development server
- Runs on http://localhost:3000
- Enables hot-reloading
- Compiles TypeScript on-the-fly

**Expected Output:**
```
▲ Next.js 14.2.15
- Local:        http://localhost:3000
- Environments: .env.local
```

### Step 7: Open Application

Open http://localhost:3000 in your browser.

**Expected:**
- Application loads
- Sign In / Sign Up buttons visible (Clerk components)
- No posts yet (empty database)

## Development Workflow

### Making Changes

1. **Edit Source Code:**
   - Edit files in `src/` directory
   - Changes hot-reload automatically

2. **Edit Schema:**
   - Edit `prisma/schema.prisma`
   - Run `npx prisma migrate dev --name description`
   - Changes applied to database

3. **Edit Environment Variables:**
   - Edit `.env` file
   - Restart development server

### Common Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run linter
npm run lint

# Generate Prisma Client
npx prisma generate

# Create migration
npx prisma migrate dev --name description

# Apply migrations (production)
npx prisma migrate deploy

# Open Prisma Studio
npx prisma studio

# Validate Prisma schema
npx prisma validate

# Format Prisma schema
npx prisma format
```

### Database Management

**View Database:**
```bash
npx prisma studio
```

**Reset Database (WARNING: Deletes all data):**
```bash
npx prisma migrate reset
```

**Seed Database (if seed script exists):**
```bash
npx prisma db seed
```

**Pull Schema from Database:**
```bash
npx prisma db pull
```

## Testing the Application

### Manual Testing Checklist

**Authentication:**
- [ ] Sign up with email
- [ ] Sign in with existing account
- [ ] Sign out
- [ ] Access protected routes (should redirect to sign in)

**User Profile:**
- [ ] View own profile
- [ ] Edit profile (name, bio, location, website)
- [ ] View other user's profile
- [ ] Follow other user
- [ ] Unfollow user

**Posts:**
- [ ] Create text-only post
- [ ] Create post with image
- [ ] View posts in feed
- [ ] Delete own post
- [ ] Cannot delete other's post

**Interactions:**
- [ ] Like post
- [ ] Unlike post
- [ ] Comment on post
- [ ] View comments
- [ ] Like count updates

**Notifications:**
- [ ] Receive notification for like
- [ ] Receive notification for comment
- [ ] Receive notification for follow
- [ ] Mark notifications as read
- [ ] Notification count displays

**File Uploads:**
- [ ] Upload image
- [ ] Image displays in post
- [ ] Image has valid URL
- [ ] Cannot upload non-image file
- [ ] Cannot upload file > 4MB

**UI/UX:**
- [ ] Dark mode toggle works
- [ ] Responsive design (mobile vs desktop)
- [ ] Navigation works
- [ ] Loading states display
- [ ] Error messages display

## Troubleshooting

### Common Issues

**Issue: Port 3000 already in use**

**Solution:**
```bash
# Kill process on port 3000 (Linux/macOS)
lsof -ti:3000 | xargs kill -9

# Or use different port
npm run dev -- -p 3001
```

**Issue: "Module not found" errors**

**Solution:**
```bash
rm -rf node_modules package-lock.json
npm install
```

**Issue: TypeScript errors**

**Solution:**
```bash
# Check TypeScript version
npx tsc --version

# Reinstall dependencies
npm install
```

**Issue: Prisma Client not generated**

**Solution:**
```bash
npx prisma generate
```

**Issue: Database connection failed**

**Solution:**
- Verify DATABASE_URL in `.env`
- Ensure PostgreSQL is running
- Test connection: `npx prisma db pull`

**Issue: Clerk authentication not working**

**Solution:**
- Verify Clerk keys in `.env`
- Check Clerk dashboard for application status
- Ensure ClerkProvider wraps application
- Check browser console for errors

**Issue: UploadThing upload failed**

**Solution:**
- Verify UPLOADTHING_TOKEN in `.env`
- Ensure user is authenticated
- Check file size is under 4MB
- Check UploadThing dashboard for errors

**Issue: Changes not reflecting**

**Solution:**
- Restart development server
- Clear browser cache
- Check for browser console errors

### Debug Mode

**Enable Verbose Logging:**

Add to `.env`:
```env
DEBUG=*
```

**Enable Prisma Query Logging:**

Edit `src/lib/prisma.ts`:
```typescript
const prisma = new PrismaClient({
  log: ["query", "error", "warn"],
});
```

## Development Tools

### Recommended VS Code Extensions

- **ESLint:** Linting
- **Prettier:** Code formatting
- **Prisma:** Prisma syntax highlighting
- **Tailwind CSS IntelliSense:** Tailwind autocomplete
- **TypeScript Vue Plugin (Volar):** TypeScript support

### Browser DevTools

Use browser DevTools to debug:
- **Console:** View errors and logs
- **Network:** Inspect API requests
- **Application:** View cookies and local storage
- **Elements:** Inspect DOM and styles

### Prisma Studio

GUI for database management:
```bash
npx prisma studio
```

Features:
- View and edit data
- Run queries
- View relationships
- Filter and sort

## Performance Tips

### Keep Development Fast

- Use `.env.local` for local variables
- Close unnecessary browser tabs
- Restart dev server periodically
- Clear node_modules and reinstall if slow

### Database Performance

- Use indexes for frequently queried fields
- Avoid N+1 queries (use `include` in Prisma)
- Use `select` to fetch only needed fields
- Monitor query performance in Prisma Studio

## Best Practices

### Do

- Commit `.env.example` to Git (not `.env`)
- Use meaningful commit messages
- Run linter before committing
- Test changes before committing
- Keep dependencies updated

### Don't

- Don't commit `.env` files
- Don't commit `node_modules/`
- Don't commit `.next/` build folder
- Don't push to main branch without testing
- Don't ignore TypeScript errors

## Resetting Development Environment

If development environment becomes corrupted:

```bash
# 1. Stop development server (Ctrl+C)

# 2. Remove dependencies
rm -rf node_modules package-lock.json

# 3. Reinstall dependencies
npm install

# 4. Reset database (WARNING: deletes all data)
npx prisma migrate reset

# 5. Restart development server
npm run dev
```

## Next Steps

After local development is working:

1. Test all features thoroughly
2. Fix any bugs or issues
3. Prepare for deployment
4. See [DEPLOYMENT_GUIDE.md](./DEPLOYMENT_GUIDE.md)

## Resources

- **Next.js:** https://nextjs.org/docs
- **Prisma:** https://www.prisma.io/docs
- **Clerk:** https://clerk.com/docs
- **UploadThing:** https://uploadthing.com/docs
- **Tailwind CSS:** https://tailwindcss.com/docs
