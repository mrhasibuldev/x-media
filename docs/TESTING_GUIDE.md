# Testing Guide

## Overview

This guide explains testing procedures for Socially.

## Current Testing Status

**NOT IMPLEMENTED:** No automated tests exist in the codebase.

**Test Files:** None found (*.test.ts, *.spec.ts, *.test.tsx, *.spec.tsx)

**Test Configuration:** No test configuration files found (jest.config.js, vitest.config.ts, etc.)

**Test Scripts:** No test scripts in package.json

---

## Manual Testing Procedures

Since automated tests are not implemented, manual testing is required.

### Pre-Testing Checklist

Before testing, ensure:
- [ ] Dependencies installed (`npm install`)
- [ ] Environment variables configured
- [ ] Database migrations applied
- [ ] Development server running (`npm run dev`)
- [ ] Clerk account configured
- [ ] UploadThing account configured
- [ ] PostgreSQL database running

---

### Authentication Testing

#### Test Case 1: Sign Up

**Steps:**
1. Navigate to http://localhost:3000
2. Click "Sign Up" button
3. Enter email and password
4. Submit form
5. Verify email (if required)
6. Verify redirect to home page
7. Verify user is authenticated (Clerk session active)

**Expected Result:** User account created, signed in, redirected to home page.

**Test Data:** Use test email: test@example.com

---

#### Test Case 2: Sign In

**Steps:**
1. Navigate to http://localhost:3000
2. Click "Sign In" button
3. Enter email and password
4. Submit form
5. Verify redirect to home page
6. Verify user is authenticated

**Expected Result:** User signed in, redirected to home page.

---

#### Test Case 3: Sign Out

**Steps:**
1. Sign in to application
2. Click user menu (UserButton)
3. Click "Sign Out"
4. Verify redirect to home page
5. Verify user is not authenticated

**Expected Result:** User signed out, redirected to home page, unauthenticated state.

---

#### Test Case 4: Protected Routes

**Steps:**
1. Sign out of application
2. Try to access protected features (e.g., create post)
3. Verify authentication required

**Expected Result:** Unauthenticated users cannot create posts; sign in prompt displayed.

---

### User Profile Testing

#### Test Case 5: View Own Profile

**Steps:**
1. Sign in to application
2. Click profile link in navbar
3. Verify profile page loads
4. Verify user information displayed correctly
5. Verify stats (followers, following, posts) displayed

**Expected Result:** Profile page loads with correct user information and stats.

---

#### Test Case 6: Edit Profile

**Steps:**
1. Navigate to own profile
2. Click "Edit Profile" button
3. Update name, bio, location, website
4. Click "Save Changes"
5. Verify success toast
6. Verify profile updated

**Expected Result:** Profile information updated successfully.

**Test Data:**
- Name: "Test User"
- Bio: "This is a test bio"
- Location: "Test City"
- Website: "https://example.com"

---

#### Test Case 7: View Other User's Profile

**Steps:**
1. Sign in to application
2. Click on another user's name in a post
3. Verify profile page loads
4. Verify "Edit Profile" button not shown
5. Verify "Follow" button shown

**Expected Result:** Other user's profile displayed correctly without edit option.

---

#### Test Case 8: Follow User

**Steps:**
1. Navigate to another user's profile
2. Click "Follow" button
3. Verify button changes to "Unfollow"
4. Verify success toast
5. Verify follower count increases

**Expected Result:** User followed successfully, button state updated.

---

#### Test Case 9: Unfollow User

**Steps:**
1. Navigate to followed user's profile
2. Click "Unfollow" button
3. Verify button changes to "Follow"
4. Verify success toast
5. Verify follower count decreases

**Expected Result:** User unfollowed successfully, button state updated.

---

### Post Testing

#### Test Case 10: Create Text Post

**Steps:**
1. Sign in to application
2. Navigate to home page
3. Enter text in "What's on your mind?" textarea
4. Click "Post" button
5. Verify success toast
6. Verify post appears in feed
7. Verify post content correct

**Expected Result:** Text post created and displayed in feed.

**Test Data:** "This is a test post"

---

#### Test Case 11: Create Post with Image

**Steps:**
1. Sign in to application
2. Navigate to home page
3. Click "Photo" button
4. Upload an image (under 4MB)
5. Enter text (optional)
6. Click "Post" button
7. Verify success toast
8. Verify post appears in feed
9. Verify image displayed correctly

**Expected Result:** Post with image created and displayed correctly.

**Test Data:**
- Image: Any image file under 4MB
- Text: "Test post with image"

---

#### Test Case 12: Create Empty Post

**Steps:**
1. Sign in to application
2. Navigate to home page
3. Leave textarea empty
4. Do not upload image
5. Click "Post" button
6. Verify button is disabled

**Expected Result:** Post button disabled when no content.

---

#### Test Case 13: Delete Own Post

**Steps:**
1. Create a test post
2. Click delete icon on post
3. Verify confirmation dialog appears
4. Click "Delete"
5. Verify success toast
6. Verify post removed from feed

**Expected Result:** Post deleted successfully.

---

#### Test Case 14: Cannot Delete Other's Post

**Steps:**
1. Sign in as User A
2. Create a post
3. Sign out
4. Sign in as User B
5. Try to delete User A's post
6. Verify delete button not shown

**Expected Result:** Delete button not shown for other user's posts.

---

### Interaction Testing

#### Test Case 15: Like Post

**Steps:**
1. Navigate to home page
2. Click like button on a post
3. Verify like count increases
4. Verify heart icon filled
5. Verify success toast

**Expected Result:** Post liked successfully, count updated.

---

#### Test Case 16: Unlike Post

**Steps:**
1. Like a post
2. Click like button again
3. Verify like count decreases
4. Verify heart icon not filled
5. Verify success toast

**Expected Result:** Post unliked successfully, count updated.

---

#### Test Case 17: Cannot Like Own Post

**Steps:**
1. Create a post
2. Try to like own post
3. Verify like button works (self-likes allowed)

**Note:** Current implementation allows self-likes. This may be intentional or a bug.

---

#### Test Case 18: Comment on Post

**Steps:**
1. Navigate to home page
2. Click comment button on a post
3. Enter comment text
4. Click "Comment" button
5. Verify success toast
6. Verify comment appears
7. Verify comment count increases

**Expected Result:** Comment added successfully, displayed correctly.

**Test Data:** "This is a test comment"

---

#### Test Case 19: View Comments

**Steps:**
1. Navigate to home page
2. Click comment button on a post
3. Verify comment section expands
4. Verify existing comments displayed
5. Verify comment authors displayed

**Expected Result:** Comments displayed correctly with author information.

---

#### Test Case 20: Empty Comment

**Steps:**
1. Click comment button on a post
2. Leave comment textarea empty
3. Click "Comment" button
4. Verify button disabled

**Expected Result:** Comment button disabled when empty.

---

### Notification Testing

#### Test Case 21: Receive Like Notification

**Steps:**
1. Sign in as User A
2. Create a post
3. Sign out
4. Sign in as User B
5. Like User A's post
6. Sign out
7. Sign in as User A
8. Click notifications icon
9. Verify like notification displayed
10. Verify notification shows correct user

**Expected Result:** Like notification received and displayed correctly.

---

#### Test Case 22: Receive Comment Notification

**Steps:**
1. Sign in as User A
2. Create a post
3. Sign out
4. Sign in as User B
5. Comment on User A's post
6. Sign out
7. Sign in as User A
8. Click notifications icon
9. Verify comment notification displayed
10. Verify notification shows comment text

**Expected Result:** Comment notification received and displayed correctly.

---

#### Test Case 23: Receive Follow Notification

**Steps:**
1. Sign in as User A
2. Sign out
3. Sign in as User B
4. Follow User A
5. Sign out
6. Sign in as User A
7. Click notifications icon
8. Verify follow notification displayed

**Expected Result:** Follow notification received and displayed correctly.

---

#### Test Case 24: Mark Notifications as Read

**Steps:**
1. Receive notifications
2. Click notifications icon
3. Verify unread count
4. Close notifications page
5. Reopen notifications
6. Verify marked as read

**Expected Result:** Notifications marked as read automatically.

---

### File Upload Testing

#### Test Case 25: Upload Valid Image

**Steps:**
1. Click "Photo" button in CreatePost
2. Select a valid image file (JPEG, PNG, etc.)
3. Verify upload progress
4. Verify image preview displayed
5. Verify URL generated

**Expected Result:** Image uploaded successfully, preview displayed.

**Test Data:** Any valid image under 4MB

---

#### Test Case 26: Upload Oversized File

**Steps:**
1. Click "Photo" button in CreatePost
2. Select a file over 4MB
3. Verify upload fails
4. Verify error message displayed

**Expected Result:** Upload rejected with error message.

**Test Data:** Any file over 4MB

---

#### Test Case 27: Upload Invalid File Type

**Steps:**
1. Click "Photo" button in CreatePost
2. Select a non-image file (e.g., PDF, EXE)
3. Verify upload fails
4. Verify error message displayed

**Expected Result:** Upload rejected with error message.

**Test Data:** Any non-image file

---

#### Test Case 28: Delete Image Before Posting

**Steps:**
1. Upload an image
2. Click delete (X) button on image preview
3. Verify image removed
4. Verify upload area reappears

**Expected Result:** Image removed successfully, can upload new image.

---

### UI/UX Testing

#### Test Case 29: Dark Mode Toggle

**Steps:**
1. Click theme toggle button
2. Verify theme changes to dark
3. Click theme toggle again
4. Verify theme changes to light
5. Verify system theme detection works

**Expected Result:** Dark mode toggle works correctly.

---

#### Test Case 30: Responsive Design

**Steps:**
1. Open application in browser
2. Resize browser to mobile width (< 1024px)
3. Verify sidebar hidden
4. Verify mobile navbar appears
5. Verify hamburger menu works
6. Resize to desktop width (≥ 1024px)
7. Verify sidebar appears
8. Verify desktop navbar appears

**Expected Result:** Responsive design works correctly on mobile and desktop.

---

#### Test Case 31: Loading States

**Steps:**
1. Create a post
2. Verify loading spinner during post creation
3. Like a post
4. Verify button state during like action
5. Comment on a post
6. Verify loading state during comment

**Expected Result:** Loading states displayed during async operations.

---

#### Test Case 32: Error States

**Steps:**
1. Disconnect from internet
2. Try to create a post
3. Verify error toast displayed
4. Reconnect to internet
5. Verify application recovers

**Expected Result:** Error states handled gracefully with user feedback.

---

### Navigation Testing

#### Test Case 33: Home Navigation

**Steps:**
1. Click "Home" in navbar
2. Verify redirected to home page
3. Verify feed displayed

**Expected Result:** Home navigation works correctly.

---

#### Test Case 34: Profile Navigation

**Steps:**
1. Click profile link in navbar
2. Verify redirected to own profile
3. Verify profile displayed

**Expected Result:** Profile navigation works correctly.

---

#### Test Case 35: Notifications Navigation

**Steps:**
1. Click notifications icon in navbar
2. Verify redirected to notifications page
3. Verify notifications displayed

**Expected Result:** Notifications navigation works correctly.

---

### Database Testing

#### Test Case 36: Data Persistence

**Steps:**
1. Create a post
2. Refresh page
3. Verify post still exists
4. Sign out
5. Sign in
6. Verify post still exists

**Expected Result:** Data persists across page refreshes and sessions.

---

#### Test Case 37: Cascade Delete

**Steps:**
1. Create a post with comments and likes
2. Delete the post
3. Verify comments deleted
4. Verify likes deleted
5. Verify notifications deleted

**Expected Result:** Cascade delete works correctly.

---

## Validation Commands

### Package Scripts

**Available Scripts (from package.json):**

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
```

**Note:** No test scripts are defined.

---

### Type Checking

**Command:**
```bash
npx tsc --noEmit
```

**Purpose:** Check TypeScript types without emitting output.

**Status:** NOT TESTED - Should be run to verify type safety.

---

### Linting

**Command:**
```bash
npm run lint
```

**Purpose:** Check code quality and style.

**Status:** NOT TESTED - Should be run to verify code quality.

---

### Build Test

**Command:**
```bash
npm run build
```

**Purpose:** Verify application builds without errors.

**Status:** NOT TESTED - Should be run to verify build succeeds.

---

### Prisma Validation

**Command:**
```bash
npx prisma validate
```

**Purpose:** Verify Prisma schema is valid.

**Status:** NOT TESTED - Should be run to verify schema.

---

## Testing Environment

### Development Environment

**Current Status:** Application not tested in development environment.

**Requirements:**
- Local PostgreSQL database
- Clerk account
- UploadThing account
- Environment variables configured

**Setup:** See [LOCAL_DEVELOPMENT.md](./LOCAL_DEVELOPMENT.md)

---

### Production Environment

**Current Status:** Application not deployed to production.

**Requirements:**
- Production PostgreSQL database
- Production Clerk application
- Production UploadThing application
- Production environment variables
- Domain and SSL

**Setup:** See [DEPLOYMENT_GUIDE.md](./DEPLOYMENT_GUIDE.md)

---

## Automated Testing (Future)

### Recommended Test Framework

**Options:**
- Jest + React Testing Library
- Vitest + React Testing Library
- Playwright (E2E testing)

### Recommended Test Structure

```
src/
├── __tests__/
│   ├── actions/
│   │   ├── post.action.test.ts
│   │   ├── user.action.test.ts
│   │   └── profile.action.test.ts
│   ├── components/
│   │   ├── PostCard.test.tsx
│   │   ├── CreatePost.test.tsx
│   │   └── Navbar.test.tsx
│   └── app/
│       ├── page.test.tsx
│       └── profile.test.tsx
```

### Test Types to Implement

1. **Unit Tests:**
   - Server Actions
   - Utility functions
   - Individual components

2. **Integration Tests:**
   - Component interactions
   - API routes
   - Database operations

3. **E2E Tests:**
   - User flows (sign up, create post, etc.)
   - Cross-page interactions
   - Critical user journeys

---

## Test Reporting

### Bug Reporting Template

**Title:** [Bug Type] Brief description

**Description:**
- What happened
- What should have happened
- Steps to reproduce

**Environment:**
- Browser: Chrome/Firefox/Safari
- OS: Windows/macOS/Linux
- Node.js version:

**Screenshots:** (if applicable)

**Priority:** High/Medium/Low

---

## Best Practices

### Do

- Test all user-facing features
- Test both happy path and error cases
- Test on multiple browsers
- Test on mobile and desktop
- Document test cases
- Report bugs with reproduction steps
- Test after code changes

### Don't

- Don't skip testing
- Don't assume code works without testing
- Don't test only happy path
- Don't ignore browser-specific issues
- Don't forget to test responsive design

---

## Next Steps

1. Complete manual testing checklist
2. Fix any bugs found
3. Implement automated tests
4. Set up CI/CD with automated testing
5. Add test coverage reporting

## Resources

- **React Testing Library:** https://testing-library.com/react
- **Jest:** https://jestjs.io
- **Vitest:** https://vitest.dev
- **Playwright:** https://playwright.dev
