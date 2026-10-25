# Media Upload Capabilities and Roadmap

## 1. Document Purpose

This document explains the current state of image and video upload capabilities in the Socially application. It is written for junior developers who want to understand how media uploading works today and how to implement video support in the future.

**What this document covers:**
- What media features currently exist
- How image uploading works
- Why video uploading is not implemented
- What would need to change to support video
- Step-by-step implementation roadmap

**What this document does NOT cover:**
- How to set up Clerk authentication (see docs/AUTHENTICATION_GUIDE.md)
- How to set up the database (see docs/DATABASE_GUIDE.md)
- General application architecture (see docs/ARCHITECTURE.md)

**How to use this document:**
1. Read the "Current Feature Status" section to understand what exists
2. Read "Current Image Upload Workflow" to understand how images work
3. Read "Recommended Database Design" to choose an approach for video
4. Follow the "Step-by-Step Implementation Roadmap" to implement changes

---

## 2. Project Overview

### Technology Stack

The Socially application uses:

**Frontend:**
- Next.js 14.2.15 (App Router)
- React 18
- TypeScript 5
- Tailwind CSS for styling

**Backend:**
- Next.js Server Actions (server-side functions)
- API Routes (for external integrations)

**Database:**
- PostgreSQL (the database that stores data)
- Prisma 6.1.0 (the tool that helps us talk to PostgreSQL)

**Authentication:**
- Clerk (@clerk/nextjs 6.9.6) - handles user sign up, sign in, sign out

**File Storage:**
- UploadThing (@uploadthing/react 7.1.4, uploadthing 7.4.2) - stores uploaded files

### How Media Uploading Fits In

When a user wants to share an image with their post, the application needs to:

1. Accept the file from the user's device
2. Send the file to a storage service (UploadThing)
3. Get back a URL (web address) for the file
4. Save that URL in the database
5. Display the image in the feed using the URL

The storage service (UploadThing) handles the hard work of:
- Storing the file
- Making it accessible via URL
- Providing a CDN for fast delivery
- Managing file security

---

## 3. Current Feature Status

### Media Feature Table

| Feature | Status | Evidence | Notes |
|---------|--------|----------|-------|
| Text-only posts | Implemented and verified in source code | src/actions/post.action.ts createPost accepts content parameter | Users can create posts without images |
| Image uploads | Implemented and verified in source code | src/app/api/uploadthing/core.ts configures image endpoint | Users can upload images to posts |
| Video uploads | Not implemented | No video endpoint in UploadThing configuration | Video uploads are not supported |
| Multiple images in one post | Not implemented | maxFileCount: 1 in UploadThing config | Only 1 image per post allowed |
| Multiple videos in one post | Not implemented | No video support at all | N/A - video not supported |
| Mixed image-and-video posts | Not implemented | No video support at all | N/A - video not supported |

### Key Findings

**What Works:**
- ✅ Users can create text-only posts
- ✅ Users can upload ONE image per post
- ✅ Images are stored on UploadThing
- ✅ Image URLs are saved in the database
- ✅ Images display in the feed

**What Doesn't Work:**
- ❌ No video upload capability
- ❌ Cannot upload multiple images per post
- ❌ Cannot upload multiple videos per post
- ❌ Cannot mix images and videos in one post

---

## 4. Current Image Upload Workflow

### Step-by-Step Flow

Let's trace what happens when a user uploads an image to a post:

**Step 1: User Clicks "Photo" Button**

**File:** `src/components/CreatePost.tsx` (lines 75-85)

When the user clicks the "Photo" button in the post composer:
```typescript
<Button onClick={() => setShowImageUpload(!showImageUpload)}>
  <ImageIcon className="size-4 mr-2" />
  Photo
</Button>
```

This sets `showImageUpload` to `true`, which shows the upload component.

**Step 2: ImageUpload Component Displays**

**File:** `src/components/ImageUpload.tsx` (lines 12-38)

The `ImageUpload` component uses UploadThing's `UploadDropzone`:
```typescript
<UploadDropzone
  endpoint="postImage"
  onClientUploadComplete={(res) => {
    onChange(res?.[0].url);
  }}
  onUploadError={(error: Error) => {
    console.log(error);
  }}
/>
```

This shows a drag-and-drop area where the user can select an image file.

**Step 3: User Selects an Image File**

When the user selects an image file:
- UploadThing automatically validates the file type (must be an image)
- UploadThing automatically validates the file size (must be under 4MB)
- UploadThing uploads the file to their cloud storage
- UploadThing generates a URL for the file

**Step 4: Upload Completes and URL Returned**

**File:** `src/app/api/uploadthing/core.ts` (lines 22-28)

When the upload completes:
```typescript
.onUploadComplete(async ({ metadata, file }) => {
  try {
    return { fileUrl: file.url };
  } catch (error) {
    console.error("Error in onUploadComplete:", error);
    throw error;
  }
})
```

UploadThing returns `file.url` - the web address where the image is stored.

**Step 5: URL Saved in Component State**

**File:** `src/components/ImageUpload.tsx` (line 31)

The URL is passed to the parent component:
```typescript
onClientUploadComplete={(res) => {
  onChange(res?.[0].url);
}}
```

**Step 6: URL Saved in CreatePost State**

**File:** `src/components/CreatePost.tsx` (lines 65-68)

The URL is saved in the component's state:
```typescript
onChange={(url) => {
  setImageUrl(url);
  if (!url) setShowImageUpload(false);
}}
```

**Step 7: User Submits Post**

**File:** `src/components/CreatePost.tsx` (lines 21-41)

When the user clicks "Post":
```typescript
const result = await createPost(content, imageUrl);
```

The `imageUrl` (UploadThing URL) is sent to the server.

**Step 8: Server Action Saves URL in Database**

**File:** `src/actions/post.action.ts` (lines 7-27)

The `createPost` Server Action saves the URL in the database:
```typescript
const post = await prisma.post.create({
  data: {
    content,
    image,  // This is the UploadThing URL
    authorId: userId,
  },
});
```

**Step 9: Image Displayed in Feed**

**File:** `src/components/PostCard.tsx` (lines 111-115)

When displaying posts in the feed:
```typescript
{post.image && (
  <div className="rounded-lg overflow-hidden">
    <img src={post.image} alt="Post content" className="w-full h-auto object-cover" />
  </div>
)}
```

The image is loaded using the URL from the database.

### Summary

The flow is:
1. User selects image → UploadThing uploads → URL returned → URL saved in database → Image displayed using URL

**Key Point:** The actual image file is stored on UploadThing's servers. Only the URL is stored in the database.

---

## 5. Current Video Upload Status

### Verification

**Video Upload Status:** NOT IMPLEMENTED

**Evidence:**

**File:** `src/app/api/uploadthing/core.ts` (lines 8-12)

The UploadThing configuration ONLY defines an image endpoint:
```typescript
postImage: f({
  image: {
    maxFileSize: "4MB",
    maxFileCount: 1,
  },
})
```

There is NO `postVideo` endpoint or any video configuration.

**What This Means:**
- The application cannot accept video files
- Even if a user tries to upload a video, UploadThing will reject it
- No UI component exists for video selection
- No database field exists for video URLs
- No video playback UI exists

**Why It's Not Implemented:**
- Video files are much larger than images
- Video storage costs more
- Video requires special handling (transcoding, streaming)
- The original tutorial focused on images only

---

## 6. Existing File Storage Architecture

### Current Architecture Components

**1. UploadThing (Storage Provider)**

**Purpose:** Stores the actual files (images) and provides URLs

**Location:** Cloud service (managed by UploadThing)

**What It Does:**
- Accepts file uploads from the application
- Stores files on their servers
- Generates URLs for accessing files
- Provides CDN for fast delivery
- Handles file validation (type, size)

**Configuration:**
- File types: Images only
- Max file size: 4MB
- Max file count: 1 per post
- Authentication: Required (Clerk middleware)

**File:** `src/app/api/uploadthing/core.ts`

**2. Application Server (Next.js)**

**Purpose:** Mediates between the user and UploadThing

**Location:** The application's server (or serverless functions)

**What It Does:**
- Receives upload requests from the frontend
- Validates user authentication (Clerk)
- Forwards requests to UploadThing
- Receives URLs from UploadThing
- Returns URLs to the frontend

**File:** `src/app/api/uploadthing/route.ts`

**3. Prisma (Database ORM)**

**Purpose:** Helps the application talk to PostgreSQL

**Location:** Runs on the application server

**What It Does:**
- Provides a clean way to write database queries
- Handles database connections
- Validates data types

**File:** `src/lib/prisma.ts`

**4. PostgreSQL (Database)**

**Purpose:** Stores data persistently

**Location:** Database server (could be local or cloud)

**What It Stores:**
- User accounts
- Posts (including image URLs)
- Comments
- Likes
- Follows
- Notifications

**What It Does NOT Store:**
- The actual image files (only the URLs)

**File:** `prisma/schema.prisma`

### Storing File vs. Storing URL

**Storing the File:**
- Means: The actual file data (bytes) is saved in the database
- Pros: Everything in one place
- Cons: Database becomes very large, slow, expensive
- Not recommended for large files like images and videos

**Storing the URL:**
- Means: Only the web address (string) is saved in the database
- Pros: Database stays small and fast, files handled by storage provider
- Cons: Need to keep storage provider happy (pay bills, don't lose files)
- Recommended for images and videos

**Current Approach:** Storing URLs only (recommended approach)

---

## 7. Current Database Design

### Post Model Structure

**File:** `prisma/schema.prisma` (lines 42-55)

```prisma
model Post{
  id        String    @id @default(cuid())
  authorId String
  content String?
  image String?        // <-- Only ONE image URL field
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt

  // Relations
  author User @relation(fields: [authorId],references: [id], onDelete: Cascade)
  comments Comment[]
  likes Like[]
  notifications Notification[]
}
```

### Current Design Analysis

**Current Fields:**
- `content` - Optional text content (String?)
- `image` - Optional image URL (String?)

**What This Means:**
- The Post model can store EITHER text OR an image OR both
- There is ONE field for media: `image`
- This field is a String (stores a URL)
- It is optional (String? means it can be null)

**Can This Represent Video Without Schema Changes?**

**NO.** Here's why:

1. **Wrong Name:** The field is called `image`, not `media` or `file`
2. **Semantic Confusion:** If we stored a video URL in the `image` field, it would be confusing
3. **No Video-Specific Fields:** No fields for video duration, thumbnail, resolution, etc.
4. **No Multiple Media:** Only ONE field for media, so we can't have multiple images or videos

**What Would Need to Change for Video Support:**

**Option A: Simple Approach (Not Recommended)**
- Rename `image` field to `mediaUrl`
- Accept that it could be an image OR video URL
- Add a `mediaType` field to distinguish between image and video
- Problem: Still only ONE media item per post

**Option B: Separate Media Model (Recommended)**
- Create a new `Media` model in Prisma
- Change Post to have a relation to Media (one-to-many)
- Each Media record represents one image or video
- This allows multiple images and/or videos per post
- This is the recommended approach (explained in Section 11)

---

## 8. Existing Upload Restrictions

### Verified Restrictions

**File Type Restrictions:**

**File:** `src/app/api/uploadthing/core.ts` (lines 8-12)

```typescript
postImage: f({
  image: {
    maxFileSize: "4MB",
    maxFileCount: 1,
  },
})
```

**Restrictions:**
- **File Types:** Images only (JPEG, PNG, GIF, WebP, etc.)
- **File Size:** Maximum 4MB per file
- **File Count:** Maximum 1 file per post

**How It's Enforced:**
- UploadThing validates file type automatically
- UploadThing validates file size automatically
- Invalid files are rejected before upload completes

**Authentication Restrictions:**

**File:** `src/app/api/uploadthing/core.ts` (lines 14-20)

```typescript
.middleware(async () => {
  const { userId } = await auth();
  if (!userId) throw new Error("Unauthorized");
  return { userId };
})
```

**Restrictions:**
- **Authentication Required:** User must be signed in via Clerk
- **Authorization:** Only authenticated users can upload
- Unauthenticated uploads are rejected with "Unauthorized" error

**How It's Enforced:**
- Clerk middleware checks for valid session
- If no valid session, upload is rejected

### Summary of Restrictions

| Restriction | Value | Enforced By |
|-------------|-------|--------------|
| File Type | Images only | UploadThing |
| File Size | Max 4MB | UploadThing |
| File Count | Max 1 per post | UploadThing |
| Authentication | Required | Clerk middleware |

---

## 9. Current Limitations and Risks

### Missing Video Support

**Status:** Not implemented

**Impact:** Users cannot share videos in posts

**Future Consideration:** Would require significant changes (explained in Section 10-17)

### File Cleanup When Posts Are Deleted

**Status:** NOT IMPLEMENTED

**Evidence:**

**File:** `src/actions/post.action.ts` (lines 193-215)

The `deletePost` function:
```typescript
export async function deletePost(postId: string) {
  // ... authorization check ...
  await prisma.post.delete({
    where: { id: postId },
  });
  // NOTE: File is NOT deleted from UploadThing
  revalidatePath("/");
  return { success: true };
}
```

**Problem:** When a post is deleted:
- The database record is removed
- The image URL is removed from the database
- BUT the actual image file remains on UploadThing servers

**Impact:**
- Orphaned files accumulate
- Storage costs increase over time
- Storage quota may be reached

**Future Consideration:** Need to implement file deletion using UploadThing's deletion API

### Missing Upload Verification

**Status:** Runtime behavior not verified

**Evidence:** Application has not been run locally

**What This Means:**
- We can see the code that handles uploads
- We cannot verify that uploads actually work in practice
- We cannot verify error handling in production

**Future Consideration:** Must test upload flow when application is running

### Other Limitations

**No Multiple Images:**
- Only 1 image per post allowed
- Users cannot share multiple images in one post

**No Upload Cancellation:**
- Once upload starts, user cannot cancel it
- No progress indicator (besides UploadThing's default)

**No Image Editing:**
- Users cannot crop, resize, or edit images before uploading
- Images are uploaded as-is

**No File Metadata:**
- No metadata stored about images (dimensions, size, etc.)
- Only the URL is stored

---

## 10. Recommended Future Architecture

### Modular Storage Architecture

**Goal:** Make it easy to switch between storage providers without rewriting the application.

**Current Problem:**
- UploadThing is tightly coupled throughout the codebase
- Components directly use UploadThing components
- Hard to switch to another provider (AWS S3, Cloudinary, etc.)

**Recommended Solution: Storage Abstraction Layer**

**Concept:** Create an interface that defines storage operations, then implement it for different providers.

**Interface Example:**

```typescript
// File: src/lib/storage/types.ts

interface StorageProvider {
  upload(file: File, options: UploadOptions): Promise<StorageResult>;
  delete(url: string): Promise<void>;
  getUrl(id: string): Promise<string>;
  validate(file: File): boolean;
}

interface UploadOptions {
  maxFileSize: number;
  allowedTypes: string[];
  folder?: string;
}

interface StorageResult {
  url: string;
  id: string;
  metadata?: Record<string, any>;
}
```

**Provider Implementations:**

```typescript
// File: src/lib/storage/providers/uploadthing.ts

class UploadThingProvider implements StorageProvider {
  async upload(file: File, options: UploadOptions): Promise<StorageResult> {
    // Call UploadThing API
    // Return result in standard format
  }

  async delete(url: string): Promise<void> {
    // Call UploadThing deletion API
  }

  async getUrl(id: string): Promise<string> {
    // Return URL for given file ID
  }

  validate(file: File): boolean {
    // Validate file against options
  }
}
```

```typescript
// File: src/lib/storage/providers/s3.ts (future)

class S3Provider implements StorageProvider {
  async upload(file: File, options: UploadOptions): Promise<StorageResult> {
    // Call AWS S3 API
    // Return result in standard format
  }

  // ... other methods
}
```

**Benefits:**
- Easy to switch providers (just change which provider is instantiated)
- Consistent API for all storage operations
- Easier to test (can mock storage provider)
- Reduces vendor lock-in

**Implementation Phases:**
1. Create storage abstraction layer
2. Implement UploadThing adapter
3. Refactor existing code to use abstraction
4. Test with UploadThing
5. Implement additional providers as needed

**Note:** This is a recommendation, not an implemented feature. It requires significant refactoring.

---

## 11. Recommended Database Design

### Approach A: Simple URL Structure on Post

**Description:** Keep a simple structure where Post has fields for media URLs.

**Example Schema:**

```prisma
model Post {
  id        String   @id @default(cuid())
  authorId String
  content   String?
  mediaUrl  String?    // Renamed from 'image'
  mediaType String?    // 'image' or 'video'
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  author User @relation(fields: [authorId], references: [id], onDelete: Cascade)
  comments Comment[]
  likes Like[]
  notifications Notification[]
}
```

**Advantages:**
- Simple and easy to understand
- Minimal schema changes
- Fast to implement
- Good for basic use cases

**Disadvantages:**
- Only ONE media item per post
- Cannot have multiple images and/or videos
- Limited for rich media posts
- Hard to add metadata (duration, dimensions, etc.)

**When This Approach is Appropriate:**
- If you only need ONE image OR ONE video per post
- If you want to get started quickly
- If you don't need rich media features

### Approach B: Separate Media Model (Recommended)

**Description:** Create a dedicated Media model to represent each media item, related to Post.

**Example Schema:**

```prisma
model Post {
  id        String   @id @default(cuid())
  authorId String
  content   String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  author User @relation(fields: [authorId], references: [id], onDelete: Cascade)
  media Media[]  // One-to-many: one post can have many media items
  comments Comment[]
  likes Like[]
  notifications Notification[]
}

model Media {
  id        String   @id @default(cuid())
  postId    String
  type      MediaType // 'image' or 'video'
  url       String   // UploadThing or S3 URL
  thumbnail String?  // For videos, a thumbnail image URL
  duration  Int?     // For videos, duration in seconds
  width     Int?     // Image/video width
  height    Int?     // Image/video height
  size      Int?     // File size in bytes
  createdAt DateTime @default(now())

  // Relations
  post Post @relation(fields: [postId], references: [id], onDelete: Cascade)
}

enum MediaType {
  IMAGE
  VIDEO
}
```

**Advantages:**
- Multiple images and/or videos per post
- Rich metadata for each media item
- Can support mixed image-and-video posts
- Flexible for future features
- Can add thumbnail for videos
- Can store video duration
- Can add multiple versions (resolutions) later

**Disadvantages:**
- More complex schema
- Requires more database queries
- More development time
- Slightly slower queries (joins required)

**When This Approach is Appropriate:**
- If you want multiple images per post
- If you want video support
- If you want mixed image-and-video posts
- If you want rich media features (thumbnails, metadata)
- If you plan to scale and add more media features

### Recommendation

**Recommended Approach: Approach B (Separate Media Model)**

**Reasons:**
1. The long-term vision includes video support
2. Social media platforms typically allow multiple images
3. Rich metadata is valuable for optimization
4. Flexibility for future features
5. Better scalability

**Note:** This is a recommendation, not an implemented change. Schema changes require careful planning and migration.

---

## 12. Frontend Implementation Plan

### Image Selection and Preview

**Current State:**
- UploadThing provides built-in dropzone UI
- Image preview after upload (in ImageUpload component)
- Can delete image before posting

**Future Changes for Video:**

**Video Selection:**
- Update UploadThing configuration to accept videos
- Update `ImageUpload` component to `MediaUpload` component
- Accept both image and video file types

**Video Preview:**
- Add video preview component (using HTML5 `<video>` tag)
- Show video player with controls
- Allow playback before posting

**Code Example:**

```typescript
// File: src/components/MediaUpload.tsx (future)

{mediaType === 'video' && value && (
  <video
    src={value}
    controls
    className="rounded-md size-40 object-cover"
  />
)}

{mediaType === 'image' && value && (
  <img
    src={value}
    alt="Upload"
    className="rounded-md size-40 object-cover"
  />
)}
```

### Upload Progress

**Current State:**
- UploadThing provides built-in progress indicator
- No custom progress bar in the application

**Future Changes:**
- Could add custom progress bar for better UX
- Could show upload percentage
- Could show upload speed

**Note:** This is optional, not required.

### Upload Cancellation

**Current State:**
- No cancellation capability
- Once upload starts, it must complete

**Future Changes:**
- Implement upload cancellation (if UploadThing supports it)
- Add cancel button
- Handle cancellation in UI

**Note:** This requires UploadThing API support.

### Validation and Error Messages

**Current State:**
- UploadThing validates file type and size
- Errors logged to console (ImageUpload.tsx line 34)
- No user-friendly error messages

**Future Changes:**
- Display error messages to user
- Show validation errors before upload
- Show helpful error messages for unsupported formats

**Code Example:**

```typescript
onUploadError={(error: Error) => {
  toast.error("Upload failed: " + error.message);
}}
```

### Responsive Media Display

**Current State:**
- Images display with `w-full h-auto object-cover` (responsive)
- No video display

**Future Changes for Video:**
- Use responsive video player
- Ensure videos work on mobile and desktop
- Handle different aspect ratios
- Add poster image (thumbnail) for videos

**Code Example:**

```typescript
<video
  src={media.url}
  poster={media.thumbnail}
  controls
  className="w-full h-auto"
  playsinline
/>
```

### Video Playback in Feed

**Current State:**
- No video playback (no videos exist)

**Future Changes:**
- Display video player in PostCard
- Add play/pause controls
- Show video duration
- Add thumbnail before playback
- Handle large videos (lazy loading)

**Code Example:**

```typescript
// File: src/components/PostCard.tsx (future)

{post.media[0].type === 'video' && (
  <video
    src={post.media[0].url}
    poster={post.media[0].thumbnail}
    controls
    className="w-full h-auto rounded-lg"
  />
)}
```

### Mobile Compatibility

**Current State:**
- Responsive design exists
- UploadThing dropzone works on mobile
- Images display correctly on mobile

**Future Changes for Video:**
- Ensure video player works on mobile
- Test on iOS and Android
- Handle mobile-specific video formats
- Optimize for mobile bandwidth

---

## 13. Backend Implementation Plan

### Secure Upload Authorization

**Current State:**
- Authentication required (Clerk middleware)
- Only authenticated users can upload

**File:** `src/app/api/uploadthing/core.ts` (lines 14-20)

**Future Changes:**
- Keep existing authentication
- Add role-based authorization (if needed)
- Add rate limiting to prevent abuse

**Note:** Current implementation is good, just needs rate limiting.

### File-Type Validation

**Current State:**
- UploadThing validates file types (images only)
- No additional validation in application

**Future Changes for Video:**
- Update UploadThing configuration to accept videos
- Add application-level validation as defense in depth
- Validate video formats (MP4, WebM, etc.)

**Code Example:**

```typescript
// File: src/app/api/uploadthing/core.ts (future)

postMedia: f({
  video: {
    maxFileSize: "100MB",
    maxFileCount: 1,
  },
})
```

### File-Size Validation

**Current State:**
- UploadThing validates file size (4MB max)
- No additional validation in application

**Future Changes for Video:**
- Increase file size limit for videos (e.g., 100MB)
- Add application-level validation
- Warn users about large file upload times

### Post Creation

**Current State:**
- `createPost` accepts content and image URL
- Stores in Post model

**File:** `src/actions/post.action.ts` (lines 7-27)

**Future Changes:**
- Accept array of media items
- Create Media records instead of storing URL directly
- Link Media records to Post

**Code Example:**

```typescript
// File: src/actions/post.action.ts (future)

export async function createPost(content: string, mediaItems: MediaInput[]) {
  const userId = await getDbUserId();
  if (!userId) return;

  const post = await prisma.post.create({
    data: {
      content,
      authorId: userId,
      media: {
        create: mediaItems.map(item => ({
          type: item.type,
          url: item.url,
          // ... other fields
        }))
      }
    },
  });

  return { success: true, post };
}
```

### Media Metadata

**Current State:**
- No metadata stored
- Only URL is stored

**Future Changes:**
- Store file size
- Store dimensions (width, height)
- Store duration (for videos)
- Store thumbnail URL (for videos)

**Code Example:**

```typescript
// File: src/lib/media/extractMetadata.ts (future)

async function extractMetadata(file: File): Promise<MediaMetadata> {
  // Use browser APIs to extract metadata
  const video = document.createElement('video');
  video.src = URL.createObjectURL(file);
  
  return {
    duration: video.duration,
    width: video.videoWidth,
    height: video.videoHeight,
    size: file.size,
  };
}
```

### Media URL Persistence

**Current State:**
- URL stored as string in Post.image field
- No additional URL management

**Future Changes:**
- Store URLs in Media model
- Consider URL expiration (if provider supports)
- Consider CDN configuration

### Deleting Associated Files Safely

**Current State:**
- Files NOT deleted when posts deleted
- Orphaned files accumulate

**Future Changes:**
- Implement file deletion in deletePost action
- Call UploadThing deletion API for each media item
- Handle deletion errors gracefully

**Code Example:**

```typescript
// File: src/actions/post.action.ts (future)

export async function deletePost(postId: string) {
  const userId = await getDbUserId();
  
  const post = await prisma.post.findUnique({
    where: { id: postId },
    include: { media: true }, // Include media items
  });

  if (!post) throw new Error("Post not found");
  if (post.authorId !== userId) throw new Error("Unauthorized");

  // Delete files from UploadThing
  for (const media of post.media) {
    await deleteFromUploadThing(media.url);
  }

  // Delete post (cascade deletes media)
  await prisma.post.delete({ where: { id: postId } });

  return { success: true };
}
```

### Error Handling

**Current State:**
- Errors logged to console
- Some errors shown as toast messages
- No comprehensive error handling

**Future Changes:**
- Add comprehensive error handling
- Log errors to structured logging service
- Show user-friendly error messages
- Implement retry logic for transient errors

---

## 14. Video Processing Considerations

### Why Larger Videos Require Special Handling

**File Size:**
- Images: Typically 100KB - 5MB
- Videos: Can be 10MB - 500MB (or larger)
- Large files take longer to upload
- Large files take longer to process

**Processing Requirements:**

**1. Background Processing**
- Videos may take time to upload
- Server may time out during upload
- Need to handle uploads asynchronously
- May need progress tracking

**2. Thumbnail Generation**
- Videos need thumbnail images for preview
- Thumbnails improve UX (users see preview before playback)
- Thumbnails reduce bandwidth (smaller than full video)

**3. Video Transcoding**
- Different devices support different video formats
- Need to convert to standard format (MP4)
- May need multiple resolutions (360p, 720p, 1080p)
- Transcoding takes significant processing time

**4. Multiple Resolutions**
- Adaptive bitrate streaming
- Automatically adjust quality based on bandwidth
- Better for mobile users with slow connections
- Requires storing multiple versions of each video

**5. Streaming vs. Progressive Download**
- Progressive download: User must download entire video before seeking
- Streaming: User can seek to any point without downloading entire video
- Streaming requires special server configuration (HLS, DASH)

### Basic First Version vs. Advanced Features

**Basic First Version (What You Need First):**

**Must Have:**
- Basic video upload
- Basic video playback (HTML5 `<video>` tag)
- File size limit (e.g., 100MB)
- Simple thumbnail (optional but recommended)

**Nice to Have (Can Wait):**
- Background processing
- Multiple resolutions
- Adaptive streaming
- Advanced transcoding
- Video editing features

**Recommendation:**
- Start with basic upload and playback
- Use UploadThing's built-in video support
- Use HTML5 `<video>` tag for playback
- Add advanced features as platform grows

---

## 15. Security Considerations

### Upload Authorization

**Current State:**
- Authentication required (Clerk middleware)
- Only authenticated users can upload

**File:** `src/app/api/uploadthing/core.ts` (lines 14-20)

**Future Considerations:**
- Keep authentication required
- Consider rate limiting per user
- Consider IP-based rate limiting

### File Validation

**Current State:**
- UploadThing validates file type (images only)
- UploadThing validates file size (4MB max)
- No additional validation in application

**Future Considerations:**
- Add application-level validation (defense in depth)
- Validate file signature (magic bytes) to prevent file type spoofing
- Scan for malware (virus scanning) if handling uploads from users

### Access Permissions

**Current State:**
- URLs are public (anyone with URL can access)
- No access control on stored files

**Future Considerations:**
- Consider private storage (signed URLs)
- Consider access control on media
- Consider expiration of URLs

### Abuse Prevention

**Current State:**
- No rate limiting
- No upload limits per user

**Future Considerations:**
- Implement rate limiting on uploads
- Limit uploads per user per day
- Monitor for suspicious upload patterns
- Implement CAPTCHA for spam prevention

### Storage Limits

**Current State:**
- No storage limit per user
- No total storage limit

**Future Considerations:**
- Implement storage quotas per user
- Monitor total storage usage
- Alert when approaching limits
- Implement cleanup of old files

### Safe Deletion

**Current State:**
- Files NOT deleted when posts deleted
- Orphaned files accumulate

**Future Considerations:**
- Implement file deletion when posts deleted
- Implement cleanup of orphaned files
- Implement retention policies (auto-delete old files)

---

## 16. Testing Plan

### Practical Tests for Image Uploads

**Test 1: Upload Valid Image**
- Select a valid image file (JPEG, PNG, etc.)
- Verify upload succeeds
- Verify preview displays
- Verify URL is returned

**Test 2: Upload Oversized Image**
- Select an image larger than 4MB
- Verify upload fails
- Verify error message displayed

**Test 3: Upload Invalid File Type**
- Select a non-image file (PDF, EXE, etc.)
- Verify upload fails
- Verify error message displayed

**Test 4: Upload Without Authentication**
- Sign out of application
- Try to upload image
- Verify upload is rejected
- Verify error message displayed

**Test 5: Delete Post with Image**
- Create post with image
- Delete the post
- Verify post is removed from feed
- Verify image is deleted from storage (if implemented)

### Practical Tests for Video Uploads (Future)

**Test 6: Upload Valid Video**
- Select a valid video file (MP4, WebM, etc.)
- Verify upload succeeds
- Verify preview displays
- Verify URL is returned

**Test 7: Upload Oversized Video**
- Select a video larger than size limit
- Verify upload fails
- Verify error message displayed

**Test 8: Upload Invalid Video Format**
- Select an invalid video file
- Verify upload fails
- Verify error message displayed

**Test 9: Video Playback in Feed**
- Upload video to post
- Navigate to feed
- Verify video player displays
- Verify playback works
- Verify controls work (play, pause, seek)

**Test 10: Video on Mobile**
- Test on mobile device
- Verify video player works
- Verify responsive behavior
- Verify playback performance

### Upload Failure Tests

**Test 11: Network Error During Upload**
- Simulate network disconnection during upload
- Verify error handling
- Verify user can retry

**Test 12: Server Error During Upload**
- Simulate server error during upload
- Verify error handling
- Verify user is informed

### Feed Playback Tests

**Test 13: Image Feed Loading**
- Navigate to feed
- Verify images load correctly
- Verify images display in correct order
- Verify responsive behavior

**Test 14: Video Feed Loading (Future)**
- Navigate to feed with videos
- Verify videos load correctly
- Verify video players display
- Verify playback performance

### Deletion Tests

**Test 15: Image Deletion with Post**
- Create post with image
- Delete post
- Verify post removed
- Verify image deleted from storage (if implemented)

**Test 16: Video Deletion with Post (Future)**
- Create post with video
- Delete post
- Verify post removed
- Verify video deleted from storage (if implemented)

---

## 17. Step-by-Step Implementation Roadmap

### Phase 1: Foundation (No Code Changes)

**Objective:** Understand current implementation

**Tasks:**
1. Read this entire document
2. Review existing upload code
3. Review Prisma schema
4. Understand UploadThing configuration

**Deliverables:**
- Understanding of current architecture
- No code changes

**Validation Criteria:**
- Can explain current upload flow
- Can identify current limitations

**Risks:** None

**Estimated Time:** 1 hour

---

### Phase 2: File Deletion Fix (Quick Win)

**Objective:** Delete files from UploadThing when posts are deleted

**Prerequisites:**
- Phase 1 complete
- UploadThing account configured

**Files to Change:**
- `src/actions/post.action.ts`

**Steps:**
1. Research UploadThing deletion API
2. Modify `deletePost` function to call deletion API
3. Test deletion locally
4. Verify files are deleted

**Code Changes:**

```typescript
// File: src/actions/post.action.ts

export async function deletePost(postId: string) {
  const userId = await getDbUserId();
  
  const post = await prisma.post.findUnique({
    where: { id: postId },
    select: { image: true },
  });

  if (!post) throw new Error("Post not found");
  if (post.authorId !== userId) throw new Error("Unauthorized");

  // Delete file from UploadThing
  if (post.image) {
    await deleteFromUploadThing(post.image);
  }

  await prisma.post.delete({
    where: { id: postId },
  });

  revalidatePath("/");
  return { success: true };
}
```

**Deliverables:**
- File deletion implemented
- Orphaned files no longer accumulate

**Validation Criteria:**
- Deleting post also deletes file from UploadThing
- No orphaned files in storage

**Risks:**
- Medium - UploadThing API may change
- Need to test deletion thoroughly

**Estimated Time:** 2-3 hours

---

### Phase 3: Rate Limiting (Security Improvement)

**Objective:** Add rate limiting to prevent abuse

**Prerequisites:**
- Phase 2 complete
- Application running locally

**Files to Change:**
- `src/middleware.ts` (or create new middleware)
- Package.json (add rate limiting library)

**Steps:**
1. Choose rate limiting library (e.g., upstash/ratelimit)
2. Install library
3. Implement rate limiting middleware
4. Add rate limits to upload endpoint
5. Test rate limiting

**Deliverables:**
- Rate limiting implemented
- Abuse prevention in place

**Validation Criteria:**
- Rate limiting prevents rapid repeated uploads
- Legitimate users not blocked

**Risks:**
- Medium - May block legitimate users if too strict
- Need to tune rate limits

**Estimated Time:** 3-4 hours

---

### Phase 4: Schema Design for Video Support

**Objective:** Design database schema for video support

**Prerequisites:**
- Phase 3 complete
- Owner approval for schema changes

**Files to Change:**
- `prisma/schema.prisma`

**Steps:**
1. Choose between Approach A (simple) or Approach B (separate Media model)
2. Recommended: Approach B (separate Media model)
3. Design Media model
4. Update Post model to relate to Media
5. Document migration strategy
6. Get approval

**Code Changes (Approach B):**

```prisma
// File: prisma/schema.prisma

model Post {
  id        String   @id @default(cuid())
  authorId String
  content   String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  author User @relation(fields: [authorId], references: [id], onDelete: Cascade)
  media Media[]  // NEW: One-to-many relation
  comments Comment[]
  likes Like[]
  notifications Notification[]
}

model Media {
  id        String   @id @default(cuid())
  postId    String
  type      MediaType // NEW: 'image' or 'video'
  url       String
  thumbnail String?  // NEW: For videos
  duration  Int?     // NEW: For videos (seconds)
  width     Int?     // NEW: Width in pixels
  height    Int?     // NEW: Height in pixels
  size      Int?     // NEW: File size in bytes
  createdAt DateTime @default(now())

  // Relations
  post Post @relation(fields: [postId], references: [id], onDelete: Cascade)
}

enum MediaType {
  IMAGE
  VIDEO
}
```

**Deliverables:**
- Database schema designed
- Migration strategy documented
- Owner approval obtained

**Validation Criteria:**
- Schema supports images and videos
- Schema supports multiple media per post
- Migration strategy clear

**Risks:**
- High - Schema changes require careful migration
- High - Existing data migration needed
- High - Testing required

**Estimated Time:** 4-6 hours (including migration)

---

### Phase 5: Backend Video Upload Implementation

**Objective:** Implement video upload in backend

**Prerequisites:**
- Phase 4 complete
- Schema migrated

**Files to Change:**
- `src/app/api/uploadthing/core.ts`
- `src/actions/post.action.ts`

**Steps:**
1. Add video endpoint to UploadThing configuration
2. Update `createPost` to accept media array
3. Implement Media creation in Prisma
4. Test video upload locally

**Code Changes:**

```typescript
// File: src/app/api/uploadthing/core.ts

export const ourFileRouter = {
  postImage: f({
    image: {
      maxFileSize: "4MB",
      maxFileCount: 1,
    },
  }),
  postVideo: f({  // NEW: Video endpoint
    video: {
      maxFileSize: "100MB",
      maxFileCount: 1,
    },
  })
    .middleware(async () => {
      const { userId } = await auth();
      if (!userId) throw new Error("Unauthorized");
      return { userId };
    })
    .onUploadComplete(async ({ metadata, file }) => {
      return { fileUrl: file.url };
    }),
} satisfies FileRouter;
```

**Deliverables:**
- Video upload endpoint configured
- Server action updated
- Video upload works locally

**Validation Criteria:**
- Video upload succeeds
- Video URL stored in database
- Video displays in feed

**Risks:**
- Medium - UploadThing video support needs verification
- Medium - Larger files may timeout

**Estimated Time:** 4-6 hours

---

### Phase 6: Frontend Video Upload Implementation

**Objective:** Implement video upload in frontend

**Prerequisites:**
- Phase 5 complete
- Backend video upload working

**Files to Change:**
- `src/components/CreatePost.tsx`
- `src/components/ImageUpload.tsx` (rename to MediaUpload.tsx)
- `src/components/PostCard.tsx`

**Steps:**
1. Rename ImageUpload to MediaUpload
2. Add video file type support
3. Add video preview component
4. Update CreatePost to handle media array
5. Update PostCard to display videos
6. Test video upload and playback

**Deliverables:**
- Video upload UI implemented
- Video preview works
- Video playback in feed works

**Validation Criteria:**
- Can select video file
- Video preview displays
- Video upload succeeds
- Video plays in feed

**Risks:**
- Medium - Video playback on mobile needs testing
- Medium - Video format compatibility

**Estimated Time:** 6-8 hours

---

### Phase 7: Thumbnail Generation (Optional)

**Objective:** Generate thumbnails for videos

**Prerequisites:**
- Phase 6 complete
- Video upload working

**Files to Change:**
- New: `src/lib/video/thumbnail.ts`
- `src/app/api/uploadthing/core.ts`

**Steps:**
1. Choose thumbnail generation service (FFmpeg, Cloudinary, etc.)
2. Implement thumbnail generation on upload
3. Store thumbnail URL in Media model
4. Display thumbnail in feed

**Deliverables:**
- Thumbnails generated for videos
- Thumbnails display in feed

**Validation Criteria:**
- Thumbnails generated automatically
- Thumbnails display correctly

**Risks:**
- High - May require external service
- High - May increase costs
- High - Additional processing time

**Estimated Time:** 8-12 hours

---

### Phase 8: Testing and Validation

**Objective:** Test all media features

**Prerequisites:**
- All previous phases complete

**Files to Change:**
- New: Tests (if implementing automated tests)

**Steps:**
1. Test image upload manually
2. Test video upload manually
3. Test post deletion
4. Test file deletion
5. Test on mobile
6. Test error handling

**Deliverables:**
- All features tested
- Bugs fixed
- Documentation updated

**Validation Criteria:**
- All tests pass
- No critical bugs

**Risks:**
- Low - Testing may reveal bugs

**Estimated Time:** 4-6 hours

---

## 18. Files Likely to Change

### Existing Files That Would Need Review or Changes

**Phase 2 (File Deletion):**
- `src/actions/post.action.ts` - Add file deletion logic

**Phase 3 (Rate Limiting):**
- `src/middleware.ts` - Add rate limiting middleware
- `package.json` - Add rate limiting library

**Phase 4 (Schema Design):**
- `prisma/schema.prisma` - Add Media model, update Post model
- Need to create migration file in `prisma/migrations/`

**Phase 5 (Backend Video Upload):**
- `src/app/api/uploadthing/core.ts` - Add video endpoint
- `src/actions/post.action.ts` - Update to handle media array
- `src/lib/uploadthing.ts` - Update type definitions

**Phase 6 (Frontend Video Upload):**
- `src/components/CreatePost.tsx` - Update to handle media array
- `src/components/ImageUpload.tsx` - Rename to MediaUpload.tsx, add video support
- `src/components/PostCard.tsx` - Add video display

**Phase 7 (Thumbnail Generation):**
- `src/app/api/uploadthing/core.ts` - Add thumbnail generation
- New: `src/lib/video/thumbnail.ts` - Thumbnail generation logic

### New Files to Create

**Phase 3 (Rate Limiting):**
- New: `src/lib/rate-limit.ts` - Rate limiting configuration

**Phase 7 (Thumbnail Generation):**
- New: `src/lib/video/thumbnail.ts` - Thumbnail generation

---

## 19. Cost and Scalability Considerations

### Storage Costs

**Current (Images Only):**
- UploadThing has free tier limits
- Beyond free tier, charges based on storage and bandwidth
- Images are relatively small (4MB max)
- Cost grows with number of users and posts

**Future (With Videos):**
- Videos are much larger than images
- Storage costs will be significantly higher
- Bandwidth costs will be higher (video streaming)
- Need to monitor usage closely

**Note:** Specific pricing depends on UploadThing's current pricing. Check their documentation for current rates.

### Video Processing Costs

**Thumbnail Generation:**
- May require external service (e.g., Cloudinary)
- May have per-video processing fees
- Can add significant cost

**Video Transcoding:**
- Requires significant processing power
- May need specialized service (AWS Elemental, Mux, etc.)
- Can be expensive
- May not be needed for first version

### Bandwidth Costs

**Current (Images):**
- Minimal bandwidth (images are small)
- CDN costs are low

**Future (With Videos):**
- Bandwidth costs significantly higher
- Video streaming requires more bandwidth
- Adaptive streaming helps but still costs more

### Scalability Considerations

**Current Limitations:**
- UploadThing has quotas
- Database has performance limits
- File size limits restrict media quality

**Future Scalability:**
- Need to monitor storage quotas
- Need to optimize database queries
- May need to add caching
- May need to implement CDN

**Recommendation:**
- Start small, optimize later
- Monitor costs closely
- Set up alerts for unusual activity

---

## 20. Known Unknowns

### What Cannot Be Verified Without Credentials

**UploadThing Configuration:**
- Exact file type restrictions (beyond "images")
- Exact file size validation behavior
- Whether UploadThing supports video uploads
- UploadThing's current pricing and quotas

**Without Runtime Testing:**
- Whether uploads actually work in practice
- Error handling in production
- Performance of uploads with large files
- Mobile browser compatibility

**Without Provider Account Access:**
- UploadThing dashboard features
- UploadThing API documentation for deletion
- UploadThing's video support capabilities

### What Requires Runtime Testing

**Image Upload Flow:**
- End-to-end upload process
- Error messages display
- Preview functionality
- Image display in feed

**Video Upload Flow:**
- Video upload capabilities (if implemented)
- Video playback in browser
- Mobile video compatibility
- Video performance

---

## 21. Final Summary

### What Works According to Source Code

**Implemented and Verified in Source Code:**
- ✅ Text-only posts (src/actions/post.action.ts)
- ✅ Image uploads (src/app/api/uploadthing/core.ts)
- ✅ One image per post (maxFileCount: 1)
- ✅ Image URLs stored in database (src/actions/post.action.ts)
- ✅ Images displayed in feed (src/components/PostCard.tsx)
- ✅ Authentication required for uploads (src/app/api/uploadthing/core.ts)
- ✅ File type validation (images only, via UploadThing)
- ✅ File size validation (4MB max, via UploadThing)

### What Is Missing

**Not Implemented:**
- ❌ Video uploads (no video endpoint in UploadThing configuration)
- ❌ Multiple images per post (maxFileCount: 1)
- ❌ Multiple videos per post (no video support)
- ❌ Mixed image-and-video posts (no video support)
- ❌ File deletion when posts are deleted (orphaned files)
- ❌ Rate limiting on uploads
- ❌ Upload cancellation
- ❌ Custom progress indicators
- ❌ Rich error messages

### What Should Be Implemented Next

**Recommended Priority Order:**

1. **Phase 2: File Deletion Fix** (Quick Win)
   - Implement file deletion when posts are deleted
   - Prevents orphaned files and storage costs
   - Low risk, high value

2. **Phase 3: Rate Limiting** (Security)
   - Add rate limiting to prevent abuse
   - Protects platform from spam
   - Medium risk, high value

3. **Phase 4: Schema Design** (Planning)
   - Design Media model for video support
   - Get owner approval
   - High risk (schema change), necessary for video

4. **Phase 5: Backend Video Upload** (Implementation)
   - Implement video upload endpoint
   - Update server actions
   - Medium risk, high value

5. **Phase 6: Frontend Video Upload** (Implementation)
   - Implement video upload UI
   - Implement video playback
   - Medium risk, high value

6. **Phase 7: Thumbnail Generation** (Optional)
   - Generate thumbnails for videos
   - Improves UX
   - High risk (cost), medium value

7. **Phase 8: Testing** (Validation)
   - Test all features
   - Fix bugs
   - Low risk, high value

### Notes for Junior Developers

**Starting Point:**
- Current implementation is simple and functional for images
- Don't try to implement everything at once
- Follow the phases sequentially
- Test each phase before moving to the next

**Learning Resources:**
- UploadThing documentation: https://uploadthing.com/docs
- Prisma documentation: https://www.prisma.io/docs
- Next.js documentation: https://nextjs.org/docs

**Key Takeaway:**
- Start with file deletion fix (Phase 2)
- Build from there
- Don't skip testing
- Ask for help when stuck

---

**Document Version:** 1.0
**Last Updated:** 2026-10-25
**Repository:** https://github.com/mrhasibuldev/x-media
