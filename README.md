# Socially

A modern social media application built with Next.js 14, featuring user authentication, post creation with images, likes, comments, follows, and notifications.

**Note:** This application was originally developed as a tutorial project by Burak. Copyright © 2024 Burak. Licensed under MIT License.

![Demo App](/public/screenshot-for-readme.png)

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Status](#project-status)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Environment Variables](#environment-variables)
- [Documentation](#documentation)
- [Architecture](#architecture)
- [Known Issues](#known-issues)
- [License](#license)

## Features

### Implemented Features

- ✅ **User Authentication** - Sign up, login, logout via Clerk
- ✅ **User Profiles** - Customizable profiles with name, bio, location, website
- ✅ **Text Posts** - Create and share text-based posts
- ✅ **Image Posts** - Upload and share images via UploadThing
- ✅ **Feed** - Chronological feed of all posts
- ✅ **Likes** - Like and unlike posts with notifications
- ✅ **Comments** - Comment on posts with notifications
- ✅ **Follow System** - Follow and unfollow other users
- ✅ **Notifications** - Real-time notifications for likes, comments, and follows
- ✅ **Dark Mode** - Toggle between light and dark themes
- ✅ **Responsive Design** - Mobile-friendly interface
- ✅ **User Suggestions** - "Who to Follow" sidebar

### Not Implemented

The following features are commonly found in social media platforms but are **NOT** present in this application:

- ❌ Video uploads (only images supported)
- ❌ Search functionality
- ❌ Direct messages
- ❌ Stories
- ❌ Reposts/shares
- ❌ Content moderation tools
- ❌ Admin panel
- ❌ User blocking
- ❌ Subscriptions/payments

## Tech Stack

### Frontend
- **Framework:** Next.js 14.2.15 (App Router)
- **Language:** TypeScript 5
- **UI Library:** React 18
- **Styling:** Tailwind CSS 3.4.1
- **Components:** shadcn/ui (Radix UI primitives)
- **Icons:** Lucide React
- **Theming:** next-themes
- **Date Formatting:** date-fns
- **Notifications:** react-hot-toast

### Backend
- **Framework:** Next.js Server Actions and API Routes
- **Database ORM:** Prisma 6.1.0
- **Database:** PostgreSQL

### Third-Party Services
- **Authentication:** Clerk (@clerk/nextjs 6.9.6)
- **File Storage:** UploadThing (@uploadthing/react 7.1.4, uploadthing 7.4.2)

## Project Status

**Current Status:** Ready for environment configuration and testing

**IMPORTANT:**
- ❌ Database migrations have NOT been generated
- ❌ Application has NOT been tested with live services
- ❌ Environment variables are NOT configured
- ❌ Application build has NOT been verified

**Required Before Running:**
1. Configure environment variables (Clerk, PostgreSQL, UploadThing)
2. Run database migrations: `npx prisma migrate dev --name init`
3. Install dependencies: `npm install`
4. Test locally: `npm run dev`

For detailed setup instructions, see [docs/LOCAL_DEVELOPMENT.md](docs/LOCAL_DEVELOPMENT.md).

## Prerequisites

Before running this application, you need:

1. **Node.js** - Version 18 or higher
2. **PostgreSQL Database** - A running PostgreSQL instance (local or cloud)
3. **Clerk Account** - Free account at https://clerk.com
4. **UploadThing Account** - Free account at https://uploadthing.com

## Quick Start

### 1. Install Dependencies

```bash
npm install
```

### 2. Configure Environment Variables

Create a `.env` file in the project root:

```env
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=your_clerk_publishable_key
CLERK_SECRET_KEY=your_clerk_secret_key
DATABASE_URL=your_database_url
UPLOADTHING_TOKEN=your_uploadthing_token
```

For detailed instructions on obtaining these values, see [docs/ENVIRONMENT_SETUP.md](docs/ENVIRONMENT_SETUP.md).

### 3. Run Database Migrations

**CRITICAL:** The database schema exists but migrations have not been generated.

```bash
npx prisma migrate dev --name init
```

This will create the necessary database tables.

### 4. Start Development Server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

## Environment Variables

The application requires the following environment variables:

| Variable | Purpose | Source |
|----------|---------|--------|
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | Clerk public key | Clerk Dashboard |
| `CLERK_SECRET_KEY` | Clerk secret key | Clerk Dashboard |
| `DATABASE_URL` | PostgreSQL connection string | Your PostgreSQL instance |
| `UPLOADTHING_TOKEN` | UploadThing API token | UploadThing Dashboard |

For detailed setup instructions, see [docs/ENVIRONMENT_SETUP.md](docs/ENVIRONMENT_SETUP.md).

## Documentation

Comprehensive documentation is available in the `docs/` directory:

### Getting Started
- [docs/PROJECT_OVERVIEW.md](docs/PROJECT_OVERVIEW.md) - Beginner-friendly project overview
- [docs/ENVIRONMENT_SETUP.md](docs/ENVIRONMENT_SETUP.md) - Environment variable configuration
- [docs/LOCAL_DEVELOPMENT.md](docs/LOCAL_DEVELOPMENT.md) - Local development setup

### Architecture
- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) - Complete system architecture
- [docs/FRONTEND_GUIDE.md](docs/FRONTEND_GUIDE.md) - Frontend architecture and components
- [docs/BACKEND_GUIDE.md](docs/BACKEND_GUIDE.md) - Backend architecture and Server Actions
- [docs/DATABASE_GUIDE.md](docs/DATABASE_GUIDE.md) - Database structure and relationships

### Features
- [docs/AUTHENTICATION_GUIDE.md](docs/AUTHENTICATION_GUIDE.md) - Clerk authentication and authorization
- [docs/MEDIA_STORAGE_GUIDE.md](docs/MEDIA_STORAGE_GUIDE.md) - UploadThing file storage
- [docs/API_REFERENCE.md](docs/API_REFERENCE.md) - Server Actions and API endpoints

### Operations
- [docs/SECURITY.md](docs/SECURITY.md) - Security practices and known issues
- [docs/BACKUP_AND_RECOVERY.md](docs/BACKUP_AND_RECOVERY.md) - Backup and recovery procedures
- [docs/DEPLOYMENT_GUIDE.md](docs/DEPLOYMENT_GUIDE.md) - Deployment instructions
- [docs/GITHUB_WORKFLOW.md](docs/GITHUB_WORKFLOW.md) - Git and GitHub workflow
- [docs/TESTING_GUIDE.md](docs/TESTING_GUIDE.md) - Testing procedures
- [docs/KNOWN_ISSUES.md](docs/KNOWN_ISSUES.md) - Known issues and limitations

## Architecture

### System Overview

```mermaid
graph TB
    subgraph "Client Side"
        Browser[Browser]
        NextJS[Next.js Frontend]
    end

    subgraph "Server Side"
        Server[Next.js Server]
        ServerActions[Server Actions]
        APIRoutes[API Routes]
    end

    subgraph "External Services"
        Clerk[Clerk Auth]
        UploadThing[UploadThing Storage]
        PostgreSQL[(PostgreSQL Database)]
    end

    Browser --> NextJS
    NextJS --> Server
    Server --> ServerActions
    Server --> APIRoutes
    ServerActions --> PostgreSQL
    APIRoutes --> UploadThing
    Server --> Clerk
    Clerk --> Server
```

### Database Schema

The application uses PostgreSQL with the following tables:

- **User** - User accounts synced from Clerk
- **Post** - User posts with optional images
- **Comment** - Comments on posts
- **Like** - Likes on posts
- **Follows** - Follow relationships between users
- **Notification** - Notifications for likes, comments, and follows

For detailed schema information, see [docs/DATABASE_GUIDE.md](docs/DATABASE_GUIDE.md).

## Available Scripts

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run linter
npm run lint

# Generate Prisma Client (automatic postinstall)
npx prisma generate

# Create database migration
npx prisma migrate dev --name description

# Apply migrations in production
npx prisma migrate deploy

# Open Prisma Studio (database GUI)
npx prisma studio
```

## Known Issues

### Critical Issues

1. **Database Not Initialized** - Migrations have not been generated. Application cannot run without running `npx prisma migrate dev --name init`.

2. **Orphaned File Storage** - Deleting a post does not delete the associated file from UploadThing, potentially causing storage costs.

3. **No Rate Limiting** - No rate limiting on API endpoints or Server Actions, making the application vulnerable to abuse.

4. **Application Build Status** - Application has not been verified to build successfully.

For a complete list of known issues, see [docs/KNOWN_ISSUES.md](docs/KNOWN_ISSUES.md).

## Security Considerations

### Positive Security Findings

✅ No exposed secrets in source code
✅ Proper .gitignore excludes environment files
✅ Clerk handles authentication securely
✅ Prisma ORM prevents SQL injection
✅ Authorization checks in place for sensitive operations

### Security Gaps

⚠️ No rate limiting
⚠️ No input sanitization beyond ORM
⚠️ No content moderation tools
⚠️ No automated security scanning

For detailed security information, see [docs/SECURITY.md](docs/SECURITY.md).

## License

MIT License

Copyright (c) 2024 Burak

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

## Acknowledgments

- Tutorial by [Burak](https://youtu.be/vUYopHWOURg)
- Built with [Next.js](https://nextjs.org/)
- Authentication by [Clerk](https://clerk.com/)
- Database by [Prisma](https://www.prisma.io/)
- File Storage by [UploadThing](https://uploadthing.com/)
- UI Components by [shadcn/ui](https://ui.shadcn.com/)

