# Frontend Guide

## Overview

This guide explains the frontend architecture of Socially, including pages, components, routing, and UI behavior.

## Technology Stack

- **Framework:** Next.js 14.2.15 (App Router)
- **Language:** TypeScript 5
- **UI Library:** React 18
- **Styling:** Tailwind CSS 3.4.1
- **Component Library:** shadcn/ui (Radix UI primitives)
- **Icons:** Lucide React
- **Theme:** next-themes for dark mode
- **Date Formatting:** date-fns
- **Toast Notifications:** react-hot-toast

## Rendering Strategy

Socially uses Next.js App Router with hybrid rendering:

- **Server Components (Default):** Render on the server for better performance and SEO
- **Client Components:** Render in the browser for interactivity (forms, state management)

### Server Components

Server components are identified by the absence of `"use client"` directive.

**Examples:**
- `src/app/layout.tsx` - Root layout
- `src/app/page.tsx` - Home page
- `src/app/profile/[username]/page.tsx` - Profile page
- `src/components/Sidebar.tsx` - User sidebar
- `src/components/Navbar.tsx` - Navigation
- `src/components/WhoToFollow.tsx` - User suggestions

**Benefits:**
- Reduced client-side JavaScript
- Better SEO
- Direct database access
- Faster initial page load

### Client Components

Client components use the `"use client"` directive at the top of the file.

**Examples:**
- `src/app/notifications/page.tsx` - Notifications page
- `src/app/profile/[username]/ProfilePageClient.tsx` - Profile client logic
- `src/components/CreatePost.tsx` - Post creation form
- `src/components/PostCard.tsx` - Post display with interactions
- `src/components/FollowButton.tsx` - Follow button
- `src/components/ImageUpload.tsx` - Image upload
- `src/components/MobileNavbar.tsx` - Mobile navigation
- `src/components/ModeToggle.tsx` - Theme toggle

**Use Cases:**
- Event handlers (onClick, onChange)
- State management (useState, useEffect)
- Browser APIs (window, localStorage)
- Interactive forms

## Page Structure

### Root Layout

**File:** `src/app/layout.tsx`

The root layout wraps every page with:

1. **ClerkProvider** - Authentication context
2. **ThemeProvider** - Dark mode context
3. **Navbar** - Navigation bar
4. **Sidebar** - User profile sidebar (desktop only)
5. **Toaster** - Toast notification container

**Layout Structure:**
```
┌─────────────────────────────────────┐
│           Navbar                    │
├──────────┬──────────────────────────┤
│          │                          │
│ Sidebar  │     Page Content         │
│ (Desktop)│                          │
│          │                          │
└──────────┴──────────────────────────┘
```

### Home Page

**File:** `src/app/page.tsx`

**Route:** `/`

**Purpose:** Display chronological feed of all posts with "Who to Follow" suggestions.

**Components:**
- `CreatePost` - Post creation form (only if user is authenticated)
- `PostCard` - Individual post display (repeated for each post)
- `WhoToFollow` - User suggestions sidebar (desktop only)

**Layout:**
```
┌─────────────────────────────────────┐
│          Create Post                 │
├─────────────────────────────────────┤
│          Post Card 1                │
├─────────────────────────────────────┤
│          Post Card 2                │
├─────────────────────────────────────┤
│          ...                        │
└─────────────────────────────────────┘
```

**Data Fetching:**
- `getPosts()` - Fetches all posts with author, comments, and likes
- `getDbUserId()` - Gets current user's database ID
- `currentUser()` - Gets current Clerk user

### Notifications Page

**File:** `src/app/notifications/page.tsx`

**Route:** `/notifications`

**Purpose:** Display user notifications for likes, comments, and follows.

**Components:**
- `NotificationSkeleton` - Loading state skeleton
- Custom notification card component

**Layout:**
```
┌─────────────────────────────────────┐
│  Notifications         [Unread: X] │
├─────────────────────────────────────┤
│  User liked your post               │
│  [Post preview]                     │
├─────────────────────────────────────┤
│  User commented on your post        │
│  [Post preview]                     │
│  [Comment text]                     │
├─────────────────────────────────────┤
│  User started following you         │
└─────────────────────────────────────┘
```

**Data Fetching:**
- `getNotifications()` - Fetches user notifications via useEffect
- `markNotificationsAsRead()` - Marks notifications as read automatically

**Notification Types:**
- **LIKE:** User liked your post
- **COMMENT:** User commented on your post
- **FOLLOW:** User started following you

### Profile Page

**Files:**
- `src/app/profile/[username]/page.tsx` (Server Component)
- `src/app/profile/[username]/ProfilePageClient.tsx` (Client Component)

**Route:** `/profile/[username]`

**Purpose:** Display user profile with posts and liked posts.

**Components:**
- Profile header with avatar, name, bio, stats
- Edit profile dialog (if viewing own profile)
- Follow/Unfollow button (if viewing other's profile)
- Tabs for "Posts" and "Likes"
- `PostCard` components for posts

**Layout:**
```
┌─────────────────────────────────────┐
│         [Avatar]                    │
│         User Name                   │
│         @username                   │
│         Bio text                    │
│                                     │
│  Following: X  Followers: Y  Posts: Z│
│                                     │
│  [Edit Profile / Follow Button]     │
│                                     │
│  Location: City                     │
│  Website: example.com               │
│  Joined: Month Year                 │
├─────────────────────────────────────┤
│  [Posts]  [Likes]                   │
├─────────────────────────────────────┤
│  Post Card 1                        │
├─────────────────────────────────────┤
│  Post Card 2                        │
└─────────────────────────────────────┘
```

**Data Fetching:**
- `getProfileByUsername()` - Fetches user profile data
- `getUserPosts()` - Fetches user's posts
- `getUserLikedPosts()` - Fetches posts user has liked
- `isFollowing()` - Checks if current user follows this user

**Dynamic Metadata:**
Page title and description are generated based on user profile.

### Tasks Page (Demo)

**File:** `src/app/tasks/page.tsx`

**Route:** `/tasks`

**Purpose:** Demo page showing API route patterns (not part of core features).

**Status:** Uses in-memory storage, not connected to database.

**Note:** This is a placeholder for demonstration purposes and can be removed in production.

## Components

### Navigation Components

#### Navbar

**File:** `src/components/Navbar.tsx`

**Type:** Server Component

**Purpose:** Top navigation bar with links and user menu.

**Features:**
- Application logo/title ("Socially")
- Desktop navigation links
- Mobile menu trigger
- User authentication status display

**Sub-components:**
- `DesktopNavbar` - Desktop navigation links
- `MobileNavbar` - Mobile slide-out menu

#### DesktopNavbar

**File:** `src/components/DesktopNavbar.tsx`

**Type:** Server Component

**Purpose:** Desktop navigation links.

**Links:**
- Home (`/`)
- Notifications (`/notifications`) - if authenticated
- Profile (`/profile/[username]`) - if authenticated
- Sign In button - if not authenticated
- User menu - if authenticated

#### MobileNavbar

**File:** `src/components/MobileNavbar.tsx`

**Type:** Client Component

**Purpose:** Mobile slide-out navigation menu.

**Features:**
- Hamburger menu trigger
- Slide-out sheet with navigation links
- Theme toggle
- Sign out button

### Sidebar Components

#### Sidebar

**File:** `src/components/Sidebar.tsx`

**Type:** Server Component

**Purpose:** Display current user's profile information.

**Features:**
- User avatar and name
- Username
- Bio
- Follower/following counts
- Location and website
- Sign in/sign up buttons (if not authenticated)

**Visibility:** Hidden on mobile, visible on desktop (lg breakpoint).

#### WhoToFollow

**File:** `src/components/WhoToFollow.tsx`

**Type:** Server Component

**Purpose:** Suggest users to follow.

**Features:**
- Displays 3 random users
- Excludes current user
- Excludes users already followed
- Shows follower count
- Follow button for each user

**Data Fetching:**
- `getRandomUsers()` - Fetches 3 random users

### Post Components

#### CreatePost

**File:** `src/components/CreatePost.tsx`

**Type:** Client Component

**Purpose:** Form for creating new posts.

**Features:**
- Text area for post content
- Image upload button
- Image preview with delete option
- Submit button with loading state

**State:**
- `content` - Post text
- `imageUrl` - Uploaded image URL
- `isPosting` - Loading state
- `showImageUpload` - Image upload UI visibility

**Server Action:**
- `createPost(content, image)` - Creates post in database

#### PostCard

**File:** `src/components/PostCard.tsx`

**Type:** Client Component

**Purpose:** Display individual post with interactions.

**Features:**
- Post author info (avatar, name, username, timestamp)
- Post content (text and optional image)
- Like button with count
- Comment button with count
- Comment section (expandable)
- Add comment form
- Delete button (for post author only)

**State:**
- `newComment` - Comment text input
- `isCommenting` - Comment submission loading state
- `isLiking` - Like action loading state
- `isDeleting` - Delete action loading state
- `hasLiked` - Whether current user liked the post
- `optimisticLikes` - Optimistic like count
- `showComments` - Comment section visibility

**Server Actions:**
- `toggleLike(postId)` - Like/unlike post
- `createComment(postId, content)` - Add comment
- `deletePost(postId)` - Delete post

**Optimistic Updates:**
Like count updates immediately before server response for better UX.

### User Components

#### FollowButton

**File:** `src/components/FollowButton.tsx`

**Type:** Client Component

**Purpose:** Follow/unfollow button for user suggestions.

**Features:**
- Follow button with loading state
- Success/error toast notifications

**Server Action:**
- `toggleFollow(targetUserId)` - Follow/unfollow user

#### ProfilePageClient

**File:** `src/app/profile/[username]/ProfilePageClient.tsx`

**Type:** Client Component

**Purpose:** Client-side logic for profile page.

**Features:**
- Edit profile dialog
- Follow/unfollow button
- Tab switching (Posts/Likes)
- Profile stats display

**State:**
- `showEditDialog` - Edit dialog visibility
- `isFollowing` - Follow status
- `isUpdatingFollow` - Follow action loading state
- `editForm` - Profile edit form data

**Server Actions:**
- `updateProfile(formData)` - Update user profile
- `toggleFollow(targetUserId)` - Follow/unfollow user

### Utility Components

#### ImageUpload

**File:** `src/components/ImageUpload.tsx`

**Type:** Client Component

**Purpose:** UploadThing image upload component.

**Features:**
- UploadThing dropzone
- Image preview
- Delete button
- Loading state

**Props:**
- `endpoint` - UploadThing endpoint name
- `onChange` - Callback when URL changes
- `value` - Current image URL

#### ModeToggle

**File:** `src/components/ModeToggle.tsx`

**Type:** Client Component

**Purpose:** Toggle between light and dark themes.

**Features:**
- Sun/moon icon toggle
- System theme detection

#### ThemeProvider

**File:** `src/components/ThemeProvider.tsx`

**Type:** Client Component

**Purpose:** Wrap application with next-themes provider.

**Configuration:**
- Attribute: class
- Default theme: system
- Enable system theme detection
- Disable transition on theme change

#### DeleteAlertDialog

**File:** `src/components/DeleteAlertDialog.tsx`

**Type:** Client Component

**Purpose:** Confirmation dialog for delete actions.

**Features:**
- Confirmation dialog
- Loading state during deletion
- Customizable title and description

#### NotificationSkeleton

**File:** `src/components/NotificationSkeleton.tsx`

**Type:** Server Component

**Purpose:** Loading skeleton for notifications page.

**Features:**
- 5 skeleton items
- Matches notification card layout

## UI Component Library (shadcn/ui)

The application uses shadcn/ui components built on Radix UI primitives.

### UI Components Location

All UI components are in `src/components/ui/`:

- `alert-dialog.tsx` - Confirmation dialogs
- `avatar.tsx` - User avatars
- `button.tsx` - Buttons
- `card.tsx` - Card containers
- `dialog.tsx` - Modal dialogs
- `input.tsx` - Text inputs
- `label.tsx` - Form labels
- `scroll-area.tsx` - Scrollable areas
- `separator.tsx` - Visual separators
- `sheet.tsx` - Slide-out panels
- `skeleton.tsx` - Loading skeletons
- `tabs.tsx` - Tab navigation
- `textarea.tsx` - Multi-line text inputs

### shadcn/ui Configuration

**File:** `components.json`

**Settings:**
- Style: "new-york"
- RSC: true (React Server Components)
- TypeScript: true
- Tailwind base color: neutral
- CSS variables: true
- Icon library: lucide

## Styling

### Tailwind CSS Configuration

**File:** `tailwind.config.ts`

**Content Paths:**
- `src/pages/**/*.{js,ts,jsx,tsx,mdx}`
- `src/components/**/*.{js,ts,jsx,tsx,mdx}`
- `src/app/**/*.{js,ts,jsx,tsx,mdx}`

**Theme:**
- Dark mode: class-based
- Custom colors via CSS variables
- Custom border radius

### Global Styles

**File:** `src/app/globals.css`

**Features:**
- Tailwind directives
- Custom font family
- CSS variables for light/dark themes
- Base styles

### CSS Variables

Light theme variables:
- `--background`, `--foreground`
- `--card`, `--card-foreground`
- `--primary`, `--primary-foreground`
- `--secondary`, `--secondary-foreground`
- `--muted`, `--muted-foreground`
- `--accent`, `--accent-foreground`
- `--destructive`, `--destructive-foreground`
- `--border`, `--input`, `--ring`
- Chart colors

Dark theme overrides for all variables.

## Responsive Design

### Breakpoints

The application uses Tailwind's default breakpoints:

- `sm`: 640px
- `md`: 768px
- `lg`: 1024px
- `xl`: 1280px
- `2xl`: 1536px

### Mobile vs Desktop

**Mobile (< 1024px):**
- Sidebar hidden
- Mobile navbar with hamburger menu
- Single-column layout
- Full-width content

**Desktop (≥ 1024px):**
- Sidebar visible (3 columns)
- Desktop navbar with inline links
- Multi-column layout
- "Who to Follow" sidebar visible

### Responsive Components

- **Navbar:** Desktop navbar hidden on mobile, mobile navbar hidden on desktop
- **Sidebar:** Hidden on mobile (`hidden lg:block`)
- **WhoToFollow:** Hidden on mobile (`hidden lg:block`)
- **Post Grid:** Single column on mobile, 10-column grid on desktop (6 for posts, 4 for sidebar)

## State Management

### Local Component State

Most state is managed at the component level using React hooks:

```typescript
const [state, setState] = useState(initialValue);
```

**Examples:**
- Form inputs (CreatePost, edit profile)
- UI toggles (show comments, show edit dialog)
- Loading states (isPosting, isLiking)
- Optimistic updates (like count)

### Server State

Server state is managed through Next.js Server Actions:

```typescript
const result = await serverAction();
```

**Benefits:**
- No separate API layer needed
- Automatic type safety
- Direct database access
- Cache invalidation with `revalidatePath`

### Cache Invalidation

Server Actions invalidate Next.js cache after mutations:

```typescript
revalidatePath("/"); // Invalidate home page
revalidatePath("/profile"); // Invalidate profile pages
```

## Routing

### File-Based Routing

Next.js App Router uses file-based routing in `src/app/`:

```
src/app/
├── layout.tsx          → / (root layout)
├── page.tsx            → / (home page)
├── notifications/
│   └── page.tsx        → /notifications
├── profile/
│   └── [username]/
│       ├── page.tsx    → /profile/:username
│       └── not-found.tsx → 404 for profiles
└── tasks/
    └── page.tsx        → /tasks
```

### Dynamic Routes

Dynamic routes use square brackets:

- `[username]` - Dynamic user profile route

### Navigation

Navigation uses Next.js Link component:

```typescript
import Link from "next/link";

<Link href="/profile/username">Profile</Link>
```

### Programmatic Navigation

Server Actions use `revalidatePath` for cache invalidation rather than programmatic navigation.

## Date Formatting

The application uses `date-fns` for date formatting:

```typescript
import { formatDistanceToNow } from "date-fns";

formatDistanceToNow(new Date(post.createdAt), { addSuffix: true });
// Output: "2 hours ago"
```

## Toast Notifications

The application uses `react-hot-toast` for user feedback:

```typescript
import toast from "react-hot-toast";

toast.success("Operation successful");
toast.error("Operation failed");
```

Toaster component is rendered in root layout:

```typescript
<Toaster />
```

## Icon Library

The application uses Lucide React icons:

```typescript
import { HeartIcon, MessageCircleIcon, UserIcon } from "lucide-react";
```

All icons are consistent in size and style.

## Fonts

The application uses custom fonts from the `src/app/fonts/` directory:

- `GeistVF.woff` - Primary font (Geist Sans)
- `GeistMonoVF.woff` - Monospace font (Geist Mono)

Fonts are loaded using Next.js `localFont`:

```typescript
const geistSans = localFont({
  src: "./fonts/GeistVF.woff",
  variable: "--font-geist-sans",
  weight: "100 900",
});
```

## Accessibility

The application includes basic accessibility features:

- Semantic HTML elements
- ARIA labels on interactive elements
- Keyboard navigation support (via Radix UI)
- Focus states on interactive elements
- Screen reader support (via Radix UI)

## Performance Optimizations

### Server Components

Using Server Components by default reduces client-side JavaScript bundle size.

### Image Optimization

Images are uploaded to UploadThing, which provides:
- Automatic optimization
- CDN delivery
- Responsive variants

### Code Splitting

Next.js automatically code-splits by route and component.

### Lazy Loading

Heavy components can be lazy-loaded (not currently implemented).

## Common Patterns

### Server Action Pattern

```typescript
"use server";

import prisma from "@/lib/prisma";
import { revalidatePath } from "next/cache";

export async function actionName(params: ParamType) {
  try {
    // 1. Get current user
    const userId = await getDbUserId();
    if (!userId) return;

    // 2. Perform database operation
    const result = await prisma.model.create({ data: params });

    // 3. Invalidate cache
    revalidatePath("/");

    // 4. Return success
    return { success: true, result };
  } catch (error) {
    console.error("Error message", error);
    return { success: false, error: "Error message" };
  }
}
```

### Client Component Pattern

```typescript
"use client";

import { useState } from "react";
import { actionName } from "@/actions/some.action";
import toast from "react-hot-toast";

function Component() {
  const [state, setState] = useState(initialValue);

  const handleSubmit = async () => {
    try {
      const result = await actionName(params);
      if (result?.success) {
        toast.success("Success");
        // Update state
      }
    } catch (error) {
      toast.error("Error");
    }
  };

  return <div>{/* JSX */}</div>;
}
```

## Troubleshooting

### Common Issues

**Issue:** Component not re-rendering after server action
**Solution:** Ensure `revalidatePath` is called in the server action

**Issue:** Client component not interactive
**Solution:** Ensure `"use client"` directive is at the top of the file

**Issue:** Styling not applying
**Solution:** Check Tailwind class names and ensure Tailwind is configured correctly

**Issue:** Dark mode not working
**Solution:** Ensure ThemeProvider wraps the component and is configured correctly

## Next Steps

For more information on:
- Backend architecture: See [BACKEND_GUIDE.md](./BACKEND_GUIDE.md)
- Database structure: See [DATABASE_GUIDE.md](./DATABASE_GUIDE.md)
- Authentication: See [AUTHENTICATION_GUIDE.md](./AUTHENTICATION_GUIDE.md)
- File uploads: See [MEDIA_STORAGE_GUIDE.md](./MEDIA_STORAGE_GUIDE.md)
