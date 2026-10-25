# Project Overview

## What is Socially?

Socially is a modern social media application built with Next.js 14, allowing users to create posts, share images, follow other users, like and comment on posts, and receive notifications. The application was originally developed as a tutorial project by Burak.

**IMPORTANT:** The application is named "Socially" in the codebase. The target GitHub repository is named "x-media". This is a naming discrepancy that should be resolved before production deployment.

## Current Status

**VERIFIED:** The application code is complete and ready for environment configuration and testing.

**UNVERIFIED:** The application has not been tested with a live database, Clerk authentication, or UploadThing file storage. These external services must be configured before the application can run.

## Key Features

### Implemented Features

- **User Authentication:** Sign up, login, and logout managed by Clerk
- **User Profiles:** Customizable profiles with name, bio, location, and website
- **Text Posts:** Create and share text-based posts
- **Image Posts:** Upload and share images (via UploadThing)
- **Feed:** Chronological feed of all posts from all users
- **Likes:** Like and unlike posts with notification to post author
- **Comments:** Comment on posts with notification to post author
- **Follow System:** Follow and unfollow other users
- **Notifications:** Real-time notifications for likes, comments, and follows
- **Dark Mode:** Toggle between light and dark themes
- **Responsive Design:** Mobile-friendly interface with adaptive layouts
- **User Suggestions:** "Who to Follow" sidebar with random user recommendations

### Features Not Implemented

The following features are commonly found in social media platforms but are **NOT** present in this application:

- Video uploads (only images are supported)
- Search functionality (no user or post search)
- Direct messages or chat
- Stories or ephemeral content
- Reposts or shares
- Content moderation tools
- Admin panel
- User blocking or muting
- Subscriptions or payments
- Analytics or insights
- Bookmarking or saving posts

### Demo/Placeholder Features

- **Tasks Page:** Located at `/tasks`, this is a demo page with in-memory storage (not connected to the database). It demonstrates API route patterns but is not part of the core social media functionality.

## Technology Stack

### Frontend
- **Framework:** Next.js 14.2.15 (App Router)
- **Language:** TypeScript 5
- **UI Library:** React 18
- **Styling:** Tailwind CSS 3.4.1
- **Components:** shadcn/ui (Radix UI primitives)
- **Icons:** Lucide React
- **Theming:** next-themes for dark mode
- **Date Formatting:** date-fns
- **Notifications:** react-hot-toast

### Backend
- **Framework:** Next.js Server Actions and API Routes
- **Database ORM:** Prisma 6.1.0
- **Database:** PostgreSQL

### Third-Party Services
- **Authentication:** Clerk (@clerk/nextjs 6.9.6)
- **File Storage:** UploadThing (@uploadthing/react 7.1.4, uploadthing 7.4.2)

## Project Structure

```
nextjs-course-master/
├── prisma/
│   └── schema.prisma          # Database schema definition
├── public/
│   ├── avatar.png             # Default avatar image
│   └── screenshot-for-readme.png  # Demo screenshot
├── src/
│   ├── actions/               # Server Actions (business logic)
│   │   ├── notification.action.ts
│   │   ├── post.action.ts
│   │   ├── profile.action.ts
│   │   └── user.action.ts
│   ├── app/
│   │   ├── api/
│   │   │   ├── tasks/         # Demo tasks API
│   │   │   └── uploadthing/  # UploadThing configuration
│   │   ├── fonts/             # Custom fonts
│   │   ├── notifications/    # Notifications page
│   │   ├── profile/[username]/ # User profile pages
│   │   ├── tasks/             # Demo tasks page
│   │   ├── globals.css        # Global styles
│   │   ├── layout.tsx         # Root layout
│   │   └── page.tsx           # Home page
│   ├── components/
│   │   ├── ui/                # shadcn/ui components
│   │   ├── CreatePost.tsx
│   │   ├── DeleteAlertDialog.tsx
│   │   ├── DesktopNavbar.tsx
│   │   ├── FollowButton.tsx
│   │   ├── ImageUpload.tsx
│   │   ├── MobileNavbar.tsx
│   │   ├── ModeToggle.tsx
│   │   ├── Navbar.tsx
│   │   ├── NotificationSkeleton.tsx
│   │   ├── PostCard.tsx
│   │   ├── Sidebar.tsx
│   │   ├── ThemeProvider.tsx
│   │   └── WhoToFollow.tsx
│   ├── lib/
│   │   ├── prisma.ts          # Prisma client singleton
│   │   ├── uploadthing.ts     # UploadThing components
│   │   └── utils.ts           # Utility functions
│   └── middleware.ts          # Clerk authentication middleware
├── .gitignore
├── components.json            # shadcn/ui configuration
├── LICENSE                    # MIT License
├── next.config.mjs            # Next.js configuration
├── package.json               # Dependencies and scripts
├── postcss.config.mjs         # PostCSS configuration
├── tailwind.config.ts         # Tailwind CSS configuration
└── tsconfig.json              # TypeScript configuration
```

## How Users Interact with the Application

### For New Users

1. **Sign Up:** Users click "Sign Up" to create an account via Clerk
2. **Profile Creation:** Clerk account information is automatically synced to the local database
3. **Profile Setup:** Users can edit their profile (name, bio, location, website)
4. **Posting:** Users can create text posts and upload images
5. **Discovery:** Users see all posts in the chronological feed
6. **Following:** Users can follow other users via "Who to Follow" suggestions or profile pages
7. **Engagement:** Users can like and comment on posts
8. **Notifications:** Users receive notifications for likes, comments, and follows

### For Returning Users

1. **Login:** Users sign in via Clerk
2. **Feed:** Home page shows latest posts from all users
3. **Create Post:** Users can create new posts with optional images
4. **Interact:** Users can like, comment, and follow
5. **Notifications:** Bell icon shows unread notification count
6. **Profile:** Users can view and edit their own profile
7. **Other Profiles:** Users can view other users' profiles and their posts/likes

## Important Notes for Beginners

### Prerequisites

Before running this application, you will need:

1. **Node.js:** Version 18 or higher (for Next.js 14)
2. **PostgreSQL Database:** A running PostgreSQL instance (local or cloud-hosted)
3. **Clerk Account:** Free account at https://clerk.com
4. **UploadThing Account:** Free account at https://uploadthing.com

### Database Setup

The application uses Prisma ORM with PostgreSQL. The schema is defined in `prisma/schema.prisma`, but **no migrations have been generated yet**. You must run:

```bash
npx prisma migrate dev --name init
```

This will create the necessary database tables.

### Environment Variables

The application requires the following environment variables in a `.env` file:

- `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` - Clerk public key
- `CLERK_SECRET_KEY` - Clerk secret key
- `DATABASE_URL` - PostgreSQL connection string
- `UPLOADTHING_TOKEN` - UploadThing API token

See [ENVIRONMENT_SETUP.md](./ENVIRONMENT_SETUP.md) for detailed instructions.

### Testing Status

**NOT TESTED:** This application has not been verified to build or run successfully. The following validation steps have NOT been performed:

- Dependency installation (`npm install`)
- Type checking (`npx tsc --noEmit`)
- Linting (`npm run lint`)
- Production build (`npm run build`)
- Database connection
- Authentication flow
- File upload flow

These steps should be performed after environment configuration.

## Known Limitations

1. **No Search:** Users cannot search for other users or posts
2. **No Video Support:** Only image uploads are supported
3. **No Moderation:** No tools to report or remove inappropriate content
4. **No Admin Panel:** No administrative interface
5. **Orphaned Files:** Deleting a post does not delete the associated file from UploadThing (may cause storage costs)
6. **No Rate Limiting:** API endpoints and server actions have no rate limiting
7. **No Testing:** No automated tests exist
8. **In-Memory Demo:** The `/tasks` page uses in-memory storage and is not persistent

## Next Steps

1. Set up external services (Clerk, UploadThing, PostgreSQL)
2. Configure environment variables
3. Run database migrations
4. Install dependencies and test locally
5. Deploy to production (e.g., Vercel)

For detailed setup instructions, see:
- [ENVIRONMENT_SETUP.md](./ENVIRONMENT_SETUP.md)
- [LOCAL_DEVELOPMENT.md](./LOCAL_DEVELOPMENT.md)
- [DEPLOYMENT_GUIDE.md](./DEPLOYMENT_GUIDE.md)
