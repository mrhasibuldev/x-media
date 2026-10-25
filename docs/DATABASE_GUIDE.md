# Database Guide

## Overview

This guide explains the database structure, relationships, and migration procedures for Socially.

## Database Technology

- **Database:** PostgreSQL
- **ORM:** Prisma 6.1.0
- **Schema Location:** `prisma/schema.prisma`
- **Migration Status:** **NOT INITIALIZED** - No migrations folder exists

## Prisma Schema

### Schema File

**Location:** `prisma/schema.prisma`

The schema defines 6 models:
1. User
2. Post
3. Comment
4. Like
5. Follows
6. Notification

### Database Configuration

```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

**DATABASE_URL Environment Variable:**
Required for database connection. Format:
```
postgresql://username:password@host:port/database
```

## Database Models

### User Model

**Purpose:** Store user account information synced from Clerk.

**Fields:**

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | String | @id, @default(cuid()) | Primary key |
| email | String | @unique | User email address |
| username | String | @unique | Username for profile URLs |
| clerkId | String | @unique | Clerk user ID for authentication |
| name | String? | Optional | Display name |
| bio | String? | Optional | User biography |
| image | String? | Optional | Profile image URL |
| location | String? | Optional | User location |
| website | String? | Optional | Personal website URL |
| createdAt | DateTime | @default(now()) | Account creation timestamp |
| updatedAt | DateTime | @updatedAt | Last update timestamp |

**Relationships:**
- One-to-many to Post (as author)
- One-to-many to Comment (as author)
- One-to-many to Like
- Self-referencing many-to-many via Follows (followers and following)
- One-to-many to Notification (as recipient - "userNotifications")
- One-to-many to Notification (as creator - "notificationCreator")

**Indexes:**
- Unique on email
- Unique on username
- Unique on clerkId

**Prisma Definition:**
```prisma
model User {
  id String @id @default(cuid())
  email String @unique
  username String @unique
  clerkId String @unique
  name String?
  bio String?
  image String?
  location String?
  website String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  posts Post[]
  comments Comment[]
  likes Like[]
  followers Follows[] @relation("following")
  following Follows[] @relation("follower")
  notifications Notification[] @relation("userNotifications")
  notificationsCreated Notification[] @relation("notificationCreator")
}
```

### Post Model

**Purpose:** Store user posts with optional images.

**Fields:**

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | String | @id, @default(cuid()) | Primary key |
| authorId | String | Foreign key to User | User who created the post |
| content | String? | Optional | Post text content |
| image | String? | Optional | UploadThing image URL |
| createdAt | DateTime | @default(now()) | Post creation timestamp |
| updatedAt | DateTime | @updatedAt | Last update timestamp |

**Relationships:**
- Many-to-one to User (author)
- One-to-many to Comment
- One-to-many to Like
- One-to-many to Notification

**Cascade Delete:** Deleting a user deletes all their posts.

**Prisma Definition:**
```prisma
model Post {
  id        String    @id @default(cuid())
  authorId String
  content String?
  image String?
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt

  author User @relation(fields: [authorId], references: [id], onDelete: Cascade)
  comments Comment[]
  likes Like[]
  notifications Notification[]
}
```

### Comment Model

**Purpose:** Store comments on posts.

**Fields:**

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | String | @id, @default(cuid()) | Primary key |
| content | String | Required | Comment text |
| authorId | String | Foreign key to User | User who wrote the comment |
| postId | String | Foreign key to Post | Post being commented on |
| createdAt | DateTime | @default(now()) | Comment creation timestamp |

**Relationships:**
- Many-to-one to User (author)
- Many-to-one to Post
- One-to-many to Notification

**Indexes:**
- Composite index on (authorId, postId) for performance

**Cascade Delete:** Deleting post or user deletes comments.

**Prisma Definition:**
```prisma
model Comment {
  id String @id @default(cuid())
  content String
  authorId String
  postId String
  createdAt DateTime @default(now())

  author User @relation(fields: [authorId], references: [id], onDelete: Cascade)
  post Post @relation(fields: [postId], references: [id], onDelete: Cascade)
  notifications Notification[]

  @@index([authorId, postId])
}
```

### Like Model

**Purpose:** Track which users liked which posts.

**Fields:**

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | String | @id, @default(cuid()) | Primary key |
| postId | String | Foreign key to Post | Post being liked |
| userId | String | Foreign key to User | User who liked |
| createdAt | DateTime | @default(now()) | Like timestamp |

**Relationships:**
- Many-to-one to User
- Many-to-one to Post

**Indexes:**
- Composite index on (userId, postId)
- Unique constraint on (userId, postId) - prevents duplicate likes

**Cascade Delete:** Deleting post or user removes likes.

**Prisma Definition:**
```prisma
model Like {
  id        String   @id @default(cuid())
  postId    String
  userId    String
  createdAt DateTime @default(now())

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
  post Post @relation(fields: [postId], references: [id], onDelete: Cascade)

  @@index([userId, postId])
  @@unique([userId, postId])
}
```

### Follows Model

**Purpose:** Track follow relationships between users.

**Fields:**

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| followerId | String | Foreign key to User, Primary Key | User who follows |
| followingId | String | Foreign key to User, Primary Key | User being followed |
| createdAt | DateTime | @default(now()) | Follow timestamp |

**Relationships:**
- Self-referencing: Both fields reference User table
- followerId → User (as "follower")
- followingId → User (as "following")

**Primary Key:** Composite on (followerId, followingId) - prevents duplicate follows

**Indexes:**
- Composite index on (followerId, followingId)

**Cascade Delete:** Deleting a user removes their follow relationships.

**Prisma Definition:**
```prisma
model Follows {
  followerId String
  followingId String
  createdAt DateTime @default(now())

  follower    User     @relation("follower", fields: [followerId], references: [id], onDelete: Cascade)
  following   User     @relation("following", fields: [followingId], references: [id], onDelete: Cascade)

  @@index([followerId, followingId])
  @@id([followerId, followingId])
}
```

### Notification Model

**Purpose:** Store notifications for likes, comments, and follows.

**Fields:**

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | String | @id, @default(cuid()) | Primary key |
| userId | String | Foreign key to User | Notification recipient |
| creatorId | String | Foreign key to User | User who triggered notification |
| type | NotificationType | Enum | LIKE, COMMENT, or FOLLOW |
| read | Boolean | @default(false) | Whether notification has been read |
| postId | String? | Foreign key to Post, Optional | Related post |
| commentId | String? | Foreign key to Comment, Optional | Related comment |
| createdAt | DateTime | @default(now()) | Notification creation timestamp |

**Relationships:**
- Many-to-one to User (recipient - "userNotifications")
- Many-to-one to User (creator - "notificationCreator")
- Many-to-one to Post (optional)
- Many-to-one to Comment (optional)

**Indexes:**
- Composite index on (userId, createdAt) for efficient fetching

**Cascade Delete:** Deleting related entities removes notifications.

**Prisma Definition:**
```prisma
model Notification {
  id        String   @id @default(cuid())
  userId    String
  creatorId String
  type      NotificationType
  read      Boolean  @default(false)
  postId    String?
  commentId String?
  createdAt DateTime @default(now())

  user      User     @relation("userNotifications", fields: [userId], references: [id], onDelete: Cascade)
  creator   User     @relation("notificationCreator", fields: [creatorId], references: [id], onDelete: Cascade)
  post      Post?    @relation(fields: [postId], references: [id], onDelete: Cascade)
  comment   Comment? @relation(fields: [commentId], references: [id], onDelete: Cascade)

  @@index([userId, createdAt])
}

enum NotificationType {
  LIKE
  COMMENT
  FOLLOW
}
```

## Entity Relationship Diagram

```mermaid
erDiagram
    User ||--o{ Post : creates
    User ||--o{ Comment : writes
    User ||--o{ Like : gives
    User ||--o{ Follows : "has followers"
    User ||--o{ Follows : "follows"
    User ||--o{ Notification : receives
    User ||--o{ Notification : creates
    Post ||--o{ Comment : has
    Post ||--o{ Like : receives
    Post ||--o{ Notification : triggers
    Comment ||--o{ Notification : triggers
```

## Database Relationships

### User to Post (One-to-Many)

One user can create many posts. Deleting a user deletes all their posts.

**Query Example:**
```typescript
// Get user's posts
const posts = await prisma.post.findMany({
  where: { authorId: userId },
});

// Get post's author
const post = await prisma.post.findUnique({
  where: { id: postId },
  include: { author: true },
});
```

### User to Comment (One-to-Many)

One user can write many comments. Deleting a user deletes all their comments.

**Query Example:**
```typescript
// Get user's comments
const comments = await prisma.comment.findMany({
  where: { authorId: userId },
});

// Get comment's author
const comment = await prisma.comment.findUnique({
  where: { id: commentId },
  include: { author: true },
});
```

### User to Like (One-to-Many)

One user can like many posts. Deleting a user removes all their likes.

**Query Example:**
```typescript
// Get user's likes
const likes = await prisma.like.findMany({
  where: { userId: userId },
});

// Get post's likes
const likes = await prisma.like.findMany({
  where: { postId: postId },
});
```

### User to Follows (Many-to-Many)

Users can follow other users. This is a self-referencing many-to-many relationship implemented via the Follows table.

**Query Example:**
```typescript
// Get user's followers
const user = await prisma.user.findUnique({
  where: { id: userId },
  include: { followers: true },
});

// Get users that user follows
const user = await prisma.user.findUnique({
  where: { id: userId },
  include: { following: true },
});

// Check if user follows another user
const follow = await prisma.follows.findUnique({
  where: {
    followerId_followingId: {
      followerId: currentUserId,
      followingId: targetUserId,
    },
  },
});
```

### Post to Comment (One-to-Many)

One post can have many comments. Deleting a post deletes all its comments.

**Query Example:**
```typescript
// Get post's comments
const comments = await prisma.comment.findMany({
  where: { postId: postId },
});

// Get comment's post
const comment = await prisma.comment.findUnique({
  where: { id: commentId },
  include: { post: true },
});
```

### Post to Like (One-to-Many)

One post can have many likes. Deleting a post removes all its likes.

**Query Example:**
```typescript
// Get post's likes
const likes = await prisma.like.findMany({
  where: { postId: postId },
});

// Check if user liked post
const like = await prisma.like.findUnique({
  where: {
    userId_postId: {
      userId: userId,
      postId: postId,
    },
  },
});
```

### Notification Relationships

Notifications can be triggered by:
- Like on post (userId, creatorId, postId)
- Comment on post (userId, creatorId, postId, commentId)
- Follow (userId, creatorId)

**Query Example:**
```typescript
// Get user's notifications
const notifications = await prisma.notification.findMany({
  where: { userId: userId },
  include: {
    creator: true,
    post: true,
    comment: true,
  },
});

// Get unread notifications
const unread = await prisma.notification.findMany({
  where: {
    userId: userId,
    read: false,
  },
});
```

## Migration Status

### Current Status

**CRITICAL:** No migrations have been generated or applied.

The Prisma schema exists in `prisma/schema.prisma`, but:
- No `prisma/migrations/` directory exists
- Database tables have not been created
- Application cannot run without database setup

### Migration Workflow

**Step 1: Generate Initial Migration**

```bash
npx prisma migrate dev --name init
```

This will:
- Create `prisma/migrations/` directory
- Generate migration SQL files
- Apply migration to database
- Create all tables and indexes

**Step 2: Verify Migration**

```bash
npx prisma studio
```

This opens Prisma Studio to view database contents.

**Step 3: Generate Prisma Client**

```bash
npx prisma generate
```

This generates the Prisma Client based on the schema.

**Note:** The `postinstall` script in `package.json` automatically runs `prisma generate` after `npm install`.

### Migration Files

**Location:** `prisma/migrations/`

Migration files contain SQL statements to create and modify database structure.

**Example Migration File Structure:**
```
prisma/migrations/
├── 20240101000000_init/
│   └── migration.sql
└── migration_lock.toml
```

**Why Migrations Should Be Version-Controlled:**

1. **Team Collaboration:** All developers can recreate the same database structure
2. **Deployment:** Production database can be updated with same migrations
3. **Rollback:** Can revert to previous schema versions
4. **History:** Track changes to database structure over time
5. **Reproducibility:** New team members can set up database easily

**.gitignore Note:** The current `.gitignore` does not exclude `prisma/migrations/`. This is correct - migrations should be committed to Git.

## Database Setup

### Prerequisites

Before setting up the database, you need:

1. **PostgreSQL Database:** A running PostgreSQL instance
   - Local: PostgreSQL installed on your machine
   - Cloud: Supabase, Neon, Railway, or similar

2. **DATABASE_URL:** Connection string for your PostgreSQL database

### Local PostgreSQL Setup

**Option 1: Install PostgreSQL Locally**

1. Download and install PostgreSQL from https://www.postgresql.org/download/
2. Create a database:
   ```bash
   createdb socially
   ```
3. Set DATABASE_URL in `.env`:
   ```
   DATABASE_URL="postgresql://postgres:password@localhost:5432/socially"
   ```

**Option 2: Use Docker**

```bash
docker run --name socially-db -e POSTGRES_PASSWORD=password -e POSTGRES_DB=socially -p 5432:5432 -d postgres
```

Then set DATABASE_URL:
```
DATABASE_URL="postgresql://postgres:password@localhost:5432/socially"
```

### Cloud PostgreSQL Setup

**Supabase (Recommended)**

1. Create account at https://supabase.com
2. Create a new project
3. Get connection string from project settings
4. Set DATABASE_URL in `.env`

**Neon**

1. Create account at https://neon.tech
2. Create a new project
3. Get connection string
4. Set DATABASE_URL in `.env`

**Railway**

1. Create account at https://railway.app
2. Add PostgreSQL service
3. Get connection string
4. Set DATABASE_URL in `.env`

### Running Migrations

Once DATABASE_URL is configured:

```bash
# Generate and apply initial migration
npx prisma migrate dev --name init

# Or if migrations already exist
npx prisma migrate dev
```

### Verifying Database Setup

```bash
# Open Prisma Studio to view database
npx prisma studio

# Or validate schema
npx prisma validate
```

## Prisma Commands

### Common Commands

```bash
# Generate Prisma Client
npx prisma generate

# Create and apply migration
npx prisma migrate dev --name migration_name

# Apply migrations in production
npx prisma migrate deploy

# Reset database (WARNING: deletes all data)
npx prisma migrate reset

# Open Prisma Studio (GUI)
npx prisma studio

# Validate schema
npx prisma validate

# Format schema file
npx prisma format

# Pull database schema (reverse engineer)
npx prisma db pull

# Push schema changes (not recommended for production)
npx prisma db push
```

### Development vs Production

**Development (`prisma migrate dev`):**
- Creates migration files
- Applies migration to database
- Recommended for development

**Production (`prisma migrate deploy`):**
- Applies existing migrations
- Does not create new migration files
- Recommended for production

## Database Seeding

**NOT IMPLEMENTED:** No seed script exists.

**Optional:** You can create a seed script to populate the database with sample data:

1. Create `prisma/seed.ts`:
```typescript
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

async function main() {
  // Create sample users, posts, etc.
}

main()
  .catch((e) => console.error(e))
  .finally(() => prisma.$disconnect());
```

2. Add to `package.json`:
```json
"prisma": {
  "seed": "ts-node prisma/seed.ts"
}
```

3. Run seed:
```bash
npx prisma db seed
```

## Database Backup and Recovery

### Backup

**Using pg_dump:**
```bash
pg_dump socially > backup.sql
```

**Using Prisma (not recommended for full backup):**
```bash
npx prisma db pull  # Only pulls schema, not data
```

### Recovery

**Using psql:**
```bash
psql socially < backup.sql
```

**Note:** See [BACKUP_AND_RECOVERY.md](./BACKUP_AND_RECOVERY.md) for detailed backup procedures.

## Performance Considerations

### Indexes

The following indexes are defined for performance:

1. **Comment:** Composite index on (authorId, postId)
   - Speeds up queries for comments by user or post

2. **Like:** Composite index on (userId, postId)
   - Speeds up queries for likes by user or post
   - Unique constraint prevents duplicate likes

3. **Follows:** Composite index on (followerId, followingId)
   - Speeds up follow relationship queries
   - Primary key prevents duplicate follows

4. **Notification:** Composite index on (userId, createdAt)
   - Speeds up notification fetching by user
   - Enables efficient chronological ordering

### Query Optimization

**Use Select:**
```typescript
// Good - fetches only needed fields
const user = await prisma.user.findUnique({
  where: { id: userId },
  select: { name: true, username: true },
});

// Avoid - fetches all fields
const user = await prisma.user.findUnique({
  where: { id: userId },
});
```

**Use Include:**
```typescript
// Good - fetches related data in single query
const post = await prisma.post.findUnique({
  where: { id: postId },
  include: { author: true, comments: true },
});

// Avoid - N+1 query problem
const post = await prisma.post.findUnique({
  where: { id: postId },
});
const author = await prisma.user.findUnique({
  where: { id: post.authorId },
});
```

**Pagination (Not Currently Implemented):**
```typescript
// Future improvement for large datasets
const posts = await prisma.post.findMany({
  take: 10,
  skip: 0,
  orderBy: { createdAt: "desc" },
});
```

## Cascade Delete Behavior

The following cascade deletes are configured:

1. **User Deleted:**
   - All user's posts are deleted
   - All user's comments are deleted
   - All user's likes are removed
   - All follow relationships are removed
   - All notifications are removed

2. **Post Deleted:**
   - All post's comments are deleted
   - All post's likes are removed
   - All notifications for the post are removed

3. **Comment Deleted:**
   - All notifications for the comment are removed

**Warning:** Cascade deletes are irreversible. Use with caution.

## Troubleshooting

### Common Issues

**Issue:** "Can't reach database server"
**Solution:** Verify DATABASE_URL is correct and database is running

**Issue:** "Migration failed"
**Solution:** Check database permissions and connection string

**Issue:** "Prisma Client not generated"
**Solution:** Run `npx prisma generate`

**Issue:** "Table doesn't exist"
**Solution:** Run `npx prisma migrate dev --name init`

**Issue:** "Unique constraint violation"
**Solution:** Check for duplicate data or conflicts

### Debugging

**Enable Query Logging:**
```typescript
// In prisma.ts
const prisma = new PrismaClient({
  log: ["query", "error", "warn"],
});
```

**View Database with Prisma Studio:**
```bash
npx prisma studio
```

## Best Practices

### Do

- Always use migrations for schema changes
- Commit migration files to Git
- Use indexes for frequently queried fields
- Use select to fetch only needed fields
- Test migrations in development first
- Backup database before major changes

### Don't

- Don't use `prisma db push` in production
- Don't manually modify database structure
- Don't commit sensitive DATABASE_URL to Git
- Don't forget to cascade delete implications
- Don't ignore migration conflicts

## Next Steps

For more information on:
- Frontend architecture: See [FRONTEND_GUIDE.md](./FRONTEND_GUIDE.md)
- Backend architecture: See [BACKEND_GUIDE.md](./BACKEND_GUIDE.md)
- Authentication: See [AUTHENTICATION_GUIDE.md](./AUTHENTICATION_GUIDE.md)
- Environment setup: See [ENVIRONMENT_SETUP.md](./ENVIRONMENT_SETUP.md)
