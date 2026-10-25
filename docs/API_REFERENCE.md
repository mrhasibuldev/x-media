# API Reference

## Overview

This guide documents all verified API endpoints and Server Actions in Socially.

## Server Actions

Server Actions are the primary backend API. They run on the server and can be called directly from client components.

### Post Actions

**File:** `src/actions/post.action.ts`

#### createPost

Creates a new post with optional image.

**Type:** Server Action

**Parameters:**
- `content` (string) - Post text content
- `image` (string) - UploadThing image URL

**Returns:**
```typescript
{
  success: boolean;
  post?: Post;
  error?: string;
}
```

**Example:**
```typescript
const result = await createPost("Hello world", "https://utfs.io/f/...");
if (result.success) {
  console.log(result.post);
}
```

**Cache Invalidation:** Revalidates `/`

---

#### getPosts

Fetches all posts with author, comments, and likes.

**Type:** Server Action

**Parameters:** None

**Returns:**
```typescript
Array<{
  id: string;
  authorId: string;
  content: string | null;
  image: string | null;
  createdAt: Date;
  updatedAt: Date;
  author: {
    id: string;
    name: string | null;
    image: string | null;
    username: string;
  };
  comments: Array<{
    id: string;
    content: string;
    authorId: string;
    postId: string;
    createdAt: Date;
    author: {
      id: string;
      username: string;
      image: string | null;
      name: string | null;
    };
  }>;
  likes: Array<{
    userId: string;
  }>;
  _count: {
    likes: number;
    comments: number;
  };
}>
```

**Ordering:** Chronological (newest first)

**Example:**
```typescript
const posts = await getPosts();
posts.forEach(post => {
  console.log(post.author.name);
  console.log(post._count.likes);
});
```

---

#### toggleLike

Likes or unlikes a post.

**Type:** Server Action

**Parameters:**
- `postId` (string) - Post ID to like/unlike

**Returns:**
```typescript
{
  success: boolean;
  error?: string;
}
```

**Behavior:**
- If already liked: Removes like
- If not liked: Adds like and creates notification (if not own post)

**Example:**
```typescript
const result = await toggleLike("post_123");
if (result.success) {
  console.log("Like toggled successfully");
}
```

**Cache Invalidation:** Revalidates `/`

---

#### createComment

Adds a comment to a post.

**Type:** Server Action

**Parameters:**
- `postId` (string) - Post ID
- `content` (string) - Comment text

**Returns:**
```typescript
{
  success: boolean;
  comment?: Comment;
  error?: string;
}
```

**Behavior:**
- Creates comment record
- Creates notification (if commenting on someone else's post)
- Uses transaction for atomicity

**Example:**
```typescript
const result = await createComment("post_123", "Great post!");
if (result.success) {
  console.log(result.comment);
}
```

**Cache Invalidation:** Revalidates `/`

---

#### deletePost

Deletes a post (only by author).

**Type:** Server Action

**Parameters:**
- `postId` (string) - Post ID to delete

**Returns:**
```typescript
{
  success: boolean;
  error?: string;
}
```

**Authorization:** Only post author can delete

**Cascade Delete:** Also deletes comments, likes, and notifications

**Example:**
```typescript
const result = await deletePost("post_123");
if (result.success) {
  console.log("Post deleted");
}
```

**Cache Invalidation:** Revalidates `/`

---

### User Actions

**File:** `src/actions/user.action.ts`

#### syncUser

Syncs Clerk user to local database.

**Type:** Server Action

**Parameters:** None (uses current Clerk session)

**Returns:**
```typescript
User | null
```

**Behavior:**
- If user exists: Returns existing user
- If not exists: Creates new user record

**Fields Synced:**
- `clerkId` - Clerk user ID
- `name` - First and last name
- `username` - Username or email prefix
- `email` - Primary email
- `image` - Profile image URL

**Example:**
```typescript
const user = await syncUser();
if (user) {
  console.log(user.username);
}
```

---

#### getUserByClerkId

Fetches user from database by Clerk ID.

**Type:** Server Action

**Parameters:**
- `clerkId` (string) - Clerk user ID

**Returns:**
```typescript
{
  id: string;
  email: string;
  username: string;
  clerkId: string;
  name: string | null;
  bio: string | null;
  image: string | null;
  location: string | null;
  website: string | null;
  createdAt: Date;
  updatedAt: Date;
  _count: {
    followers: number;
    following: number;
    posts: number;
  };
} | null
```

**Example:**
```typescript
const user = await getUserByClerkId("clerk_123");
if (user) {
  console.log(user._count.followers);
}
```

---

#### getDbUserId

Gets database user ID from Clerk session.

**Type:** Server Action

**Parameters:** None

**Returns:** `string | null`

**Behavior:**
- Gets Clerk user ID from session
- Fetches user from database by clerkId
- Returns database user ID

**Example:**
```typescript
const userId = await getDbUserId();
if (userId) {
  console.log("Database user ID:", userId);
}
```

---

#### getRandomUsers

Gets 3 random users to suggest for following.

**Type:** Server Action

**Parameters:** None

**Returns:**
```typescript
Array<{
  id: string;
  name: string | null;
  username: string;
  image: string | null;
  _count: {
    followers: number;
  };
}>
```

**Filters:**
- Excludes current user
- Excludes users already followed

**Example:**
```typescript
const users = await getRandomUsers();
users.forEach(user => {
  console.log(user.name, user._count.followers);
});
```

---

#### toggleFollow

Follows or unfollows a user.

**Type:** Server Action

**Parameters:**
- `targetUserId` (string) - User ID to follow/unfollow

**Returns:**
```typescript
{
  success: boolean;
  error?: string;
}
```

**Behavior:**
- If following: Removes follow
- If not following: Adds follow and creates notification

**Restriction:** Cannot follow self

**Example:**
```typescript
const result = await toggleFollow("user_456");
if (result.success) {
  console.log("Follow toggled");
}
```

**Cache Invalidation:** Revalidates `/`

---

### Profile Actions

**File:** `src/actions/profile.action.ts`

#### getProfileByUsername

Fetches user profile by username.

**Type:** Server Action

**Parameters:**
- `username` (string) - Username

**Returns:**
```typescript
{
  id: string;
  name: string | null;
  username: string;
  bio: string | null;
  image: string | null;
  location: string | null;
  website: string | null;
  createdAt: Date;
  _count: {
    followers: number;
    following: number;
    posts: number;
  };
} | null
```

**Example:**
```typescript
const profile = await getProfileByUsername("johndoe");
if (profile) {
  console.log(profile.bio);
}
```

---

#### getUserPosts

Fetches all posts by a specific user.

**Type:** Server Action

**Parameters:**
- `userId` (string) - User ID

**Returns:** Array of post objects (same structure as getPosts)

**Ordering:** Chronological (newest first)

**Example:**
```typescript
const posts = await getUserPosts("user_123");
posts.forEach(post => {
  console.log(post.content);
});
```

---

#### getUserLikedPosts

Fetches all posts liked by a specific user.

**Type:** Server Action

**Parameters:**
- `userId` (string) - User ID

**Returns:** Array of post objects (same structure as getPosts)

**Ordering:** Chronological (newest first)

**Example:**
```typescript
const likedPosts = await getUserLikedPosts("user_123");
likedPosts.forEach(post => {
  console.log(post.content);
});
```

---

#### updateProfile

Updates user profile information.

**Type:** Server Action

**Parameters:**
- `formData` (FormData) - Form data with profile fields

**Fields:**
- `name` - Display name
- `bio` - Biography
- `location` - Location
- `website` - Website URL

**Returns:**
```typescript
{
  success: boolean;
  user?: User;
  error?: string;
}
```

**Example:**
```typescript
const formData = new FormData();
formData.append("name", "John Doe");
formData.append("bio", "Developer");
const result = await updateProfile(formData);
if (result.success) {
  console.log(result.user);
}
```

**Cache Invalidation:** Revalidates `/profile`

---

#### isFollowing

Checks if current user follows a specific user.

**Type:** Server Action

**Parameters:**
- `userId` (string) - User ID to check

**Returns:** `boolean`

**Example:**
```typescript
const following = await isFollowing("user_456");
if (following) {
  console.log("User is following this user");
}
```

---

### Notification Actions

**File:** `src/actions/notification.action.ts`

#### getNotifications

Fetches all notifications for current user.

**Type:** Server Action

**Parameters:** None

**Returns:**
```typescript
Array<{
  id: string;
  userId: string;
  creatorId: string;
  type: "LIKE" | "COMMENT" | "FOLLOW";
  read: boolean;
  postId: string | null;
  commentId: string | null;
  createdAt: Date;
  creator: {
    id: string;
    name: string | null;
    username: string;
    image: string | null;
  };
  post: {
    id: string;
    content: string | null;
    image: string | null;
  } | null;
  comment: {
    id: string;
    content: string;
    createdAt: Date;
  } | null;
}>
```

**Ordering:** Chronological (newest first)

**Example:**
```typescript
const notifications = await getNotifications();
notifications.forEach(notif => {
  console.log(notif.type, notif.creator.name);
});
```

---

#### markNotificationsAsRead

Marks notifications as read.

**Type:** Server Action

**Parameters:**
- `notificationIds` (string[]) - Array of notification IDs

**Returns:**
```typescript
{
  success: boolean;
}
```

**Example:**
```typescript
const result = await markNotificationsAsRead(["notif_1", "notif_2"]);
if (result.success) {
  console.log("Notifications marked as read");
}
```

---

## API Routes

API Routes handle external service integrations and demo endpoints.

### UploadThing Route

**File:** `src/app/api/uploadthing/route.ts`

**Route:** `/api/uploadthing`

**Methods:** GET, POST

**Purpose:** UploadThing file upload handler.

**Authentication:** Required (Clerk middleware)

**Configuration:** See [MEDIA_STORAGE_GUIDE.md](./MEDIA_STORAGE_GUIDE.md)

---

### Tasks Route (Demo)

**File:** `src/app/api/tasks/route.ts`

**Route:** `/api/tasks`

**Methods:** GET, POST, DELETE

**Purpose:** Demo API route with in-memory storage.

**Status:** Not part of core features. Can be removed in production.

#### GET /api/tasks

Fetches all tasks.

**Returns:**
```typescript
Array<{
  id: number;
  title: string;
  completed: boolean;
}>
```

**Example:**
```typescript
const response = await fetch("/api/tasks");
const tasks = await response.json();
```

---

#### POST /api/tasks

Creates a new task.

**Request Body:**
```typescript
{
  title: string;
}
```

**Returns:**
```typescript
{
  id: number;
  title: string;
  completed: boolean;
}
```

**Example:**
```typescript
const response = await fetch("/api/tasks", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ title: "New task" }),
});
const task = await response.json();
```

---

#### DELETE /api/tasks

Deletes a task by ID.

**Query Parameters:**
- `id` (number) - Task ID

**Returns:**
```typescript
{
  message: string;
}
```

**Example:**
```typescript
const response = await fetch("/api/tasks?id=1", {
  method: "DELETE",
});
const result = await response.json();
```

---

## Error Responses

All Server Actions return error objects on failure:

```typescript
{
  success: false;
  error: string; // User-friendly error message
}
```

**Common Errors:**
- "Failed to create post"
- "Failed to toggle like"
- "Failed to create comment"
- "Failed to delete post"
- "Unauthorized - no delete permission"
- "User not found"
- "Post not found"
- "Content is required"

## Authentication

All Server Actions require authentication via Clerk. If user is not authenticated:

- `getDbUserId()` returns `null`
- Server Actions return early without performing operations
- Client components should handle unauthenticated state

## Rate Limiting

**NOT IMPLEMENTED:** No rate limiting is currently configured on Server Actions or API routes.

## Pagination

**NOT IMPLEMENTED:** No pagination is currently implemented. All queries return all results.

Future improvement: Add `take` and `skip` parameters to Server Actions.

## Next Steps

For more information on:
- Backend architecture: See [BACKEND_GUIDE.md](./BACKEND_GUIDE.md)
- Database structure: See [DATABASE_GUIDE.md](./DATABASE_GUIDE.md)
- Authentication: See [AUTHENTICATION_GUIDE.md](./AUTHENTICATION_GUIDE.md)
