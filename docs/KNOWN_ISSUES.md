# Known Issues

## Overview

This document lists verified issues, limitations, and unresolved questions in Socially.

## Critical Issues

### 1. Database Not Initialized

**Status:** CRITICAL - Application cannot run without database setup

**Description:**
- Prisma schema exists in `prisma/schema.prisma`
- No migrations folder exists
- Database tables have not been created
- Application will fail without manual migration

**Impact:** Application cannot function

**Evidence:** No `prisma/migrations/` directory found in project

**Resolution Required:**
```bash
npx prisma migrate dev --name init
```

**Priority:** P0 - Must be resolved before application can run

---

### 2. Orphaned File Storage

**Status:** HIGH - Storage cost and data accumulation issue

**Description:**
- When a post is deleted, only the database record is removed
- The associated file remains on UploadThing servers
- This results in orphaned files and unnecessary storage costs

**Impact:** Financial cost over time; data accumulation

**Evidence:** `src/actions/post.action.ts` deletePost function only removes database record (lines 205-207)

**Example:**
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

**Resolution Required:**
Implement file deletion in deletePost action using UploadThing's deletion API.

**Priority:** P1 - Should be resolved to prevent storage costs

---

## High Priority Issues

### 3. No Rate Limiting

**Status:** HIGH - Vulnerability to abuse

**Description:**
- No rate limiting configured on API endpoints or Server Actions
- Application is vulnerable to spam, abuse, or DoS attacks

**Impact:** Service degradation or abuse

**Evidence:** No rate limiting middleware found in codebase

**Affected Endpoints:**
- createPost
- createComment
- toggleLike
- toggleFollow
- All API routes

**Resolution Required:**
Implement rate limiting using Next.js middleware or dedicated service.

**Priority:** P1 - Should be resolved before production deployment

---

### 4. No Input Sanitization Beyond Database ORM

**Status:** MEDIUM - Potential XSS risk

**Description:**
- No XSS protection for user-generated content beyond React's automatic escaping
- User content is displayed directly without additional sanitization

**Impact:** Potential cross-site scripting attacks

**Evidence:** Content displayed directly in PostCard without additional sanitization (line 106 in PostCard.tsx)

**Mitigation:**
- React provides some automatic escaping
- Additional sanitization recommended

**Resolution Required:**
Implement content sanitization library (e.g., DOMPurify).

**Priority:** P2 - Should be resolved for production security

---

### 5. No Content Moderation

**Status:** MEDIUM - Legal and reputational risk

**Description:**
- No tools to report or remove inappropriate content
- No admin interface for moderation

**Impact:** Platform could host harmful content

**Evidence:** No moderation features found in codebase

**Resolution Required:**
Implement content reporting system and admin moderation tools.

**Priority:** P2 - Should be resolved for production use

---

## Medium Priority Issues

### 6. No Automated Security Scanning

**Status:** LOW - Security monitoring gap

**Description:**
- No dependency scanning or SAST tools configured
- Vulnerable dependencies may go undetected

**Impact:** Potential security vulnerabilities in dependencies

**Evidence:** No GitHub Actions, npm audit, or security tools found

**Resolution Required:**
Add GitHub Actions with dependency scanning or integrate Snyk.

**Priority:** P3 - Should be resolved for production security

---

### 7. Error Logging to Console

**Status:** LOW - Operational difficulty

**Description:**
- Errors logged to console rather than structured logging service
- Difficult to debug production issues
- No security event logging

**Impact:** Difficult to debug production issues; no security monitoring

**Evidence:** console.error throughout server actions

**Resolution Required:**
Implement structured logging (e.g., Sentry, LogRocket).

**Priority:** P3 - Should be resolved for production monitoring

---

### 8. No Backup Strategy Documented

**Status:** LOW - Data loss risk

**Description:**
- No database backup procedures documented
- No backup configuration found

**Impact:** Potential data loss if database fails

**Evidence:** No backup configuration found

**Resolution Required:**
Document and implement automated database backups.

**Priority:** P3 - Should be resolved for production data safety

---

## Feature Limitations

### 9. No Search Functionality

**Status:** NOT IMPLEMENTED

**Description:**
- Users cannot search for other users
- Users cannot search for posts
- No search interface exists

**Impact:** Reduced discoverability

**Evidence:** No search components or routes found

**Resolution Required:**
Implement search functionality (optional feature).

**Priority:** P4 - Feature enhancement

---

### 10. No Video Uploads

**Status:** NOT IMPLEMENTED

**Description:**
- Only image uploads are supported
- No video upload endpoint configured
- UploadThing configuration limited to images

**Impact:** Limited content types

**Evidence:** UploadThing configuration only allows images (src/app/api/uploadthing/core.ts lines 8-12)

**Resolution Required:**
Add video upload endpoint if needed (optional feature).

**Priority:** P4 - Feature enhancement

---

### 11. No User Blocking

**Status:** NOT IMPLEMENTED

**Description:**
- Users cannot block other users
- No mute functionality
- No privacy controls

**Impact:** No protection from harassment

**Evidence:** No blocking features found

**Resolution Required:**
Implement user blocking (optional feature).

**Priority:** P4 - Feature enhancement

---

### 12. No Admin Panel

**Status:** NOT IMPLEMENTED

**Description:**
- No administrative interface
- No user management tools
- No content moderation tools

**Impact:** No admin control over platform

**Evidence:** No admin components or routes found

**Resolution Required:**
Implement admin panel (optional feature).

**Priority:** P4 - Feature enhancement

---

### 13. No Repost/Share Functionality

**Status:** NOT IMPLEMENTED

**Description:**
- Users cannot repost or share posts
- No share to external platforms

**Impact:** Limited content distribution

**Evidence:** No repost/share features found

**Resolution Required:**
Implement repost functionality (optional feature).

**Priority:** P4 - Feature enhancement

---

### 14. No Stories Feature

**Status:** NOT IMPLEMENTED

**Description:**
- No ephemeral content (stories)
- No story creation or viewing

**Impact:** Missing common social media feature

**Evidence:** No story components found

**Resolution Required:**
Implement stories (optional feature).

**Priority:** P4 - Feature enhancement

---

### 15. No Direct Messages

**Status:** NOT IMPLEMENTED

**Description:**
- No private messaging system
- No chat functionality

**Impact:** No private communication

**Evidence:** No messaging components found

**Resolution Required:**
Implement direct messages (optional feature).

**Priority:** P4 - Feature enhancement

---

### 16. No Bookmarks

**Status:** NOT IMPLEMENTED

**Description:**
- Users cannot bookmark or save posts
- No saved posts collection

**Impact:** No content saving

**Evidence:** No bookmark features found

**Resolution Required:**
Implement bookmarks (optional feature).

**Priority:** P4 - Feature enhancement

---

## Configuration Issues

### 17. Repository Name Mismatch

**Status:** CONFIGURATION

**Description:**
- Application is named "Socially" in codebase
- Target GitHub repository is named "x-media"
- This is a naming discrepancy

**Impact:** Potential confusion

**Evidence:**
- package.json name: "socially"
- Application title: "Socially"
- GitHub repo: "x-media"

**Resolution Required:**
Decide whether to rename application to "X-Media" or rename repository to "socially".

**Priority:** P2 - Should be resolved before production

---

### 18. Tasks Page is Demo

**Status:** DEMO CODE

**Description:**
- `/tasks` page exists but is a demo
- Uses in-memory storage, not database
- Not part of core social media features

**Impact:** Potential confusion; demo code in production

**Evidence:** src/app/tasks/page.tsx and src/app/api/tasks/route.ts use in-memory storage

**Resolution Required:**
Remove tasks page or integrate with database.

**Priority:** P3 - Should be resolved before production

---

## Unverified Behavior

### 19. Application Build Status

**Status:** UNVERIFIED

**Description:**
- Application has not been verified to build successfully
- Build may fail due to unknown issues

**Impact:** Deployment may fail

**Evidence:** `npm run build` has not been executed

**Resolution Required:**
Run `npm run build` to verify build succeeds.

**Priority:** P1 - Must be verified before deployment

---

### 20. Database Connection Status

**Status:** UNVERIFIED

**Description:**
- Database connection has not been tested
- DATABASE_URL has not been configured
- Unknown if database is accessible

**Impact:** Application cannot run without database

**Evidence:** No .env file found; database not tested

**Resolution Required:**
Configure DATABASE_URL and test connection.

**Priority:** P1 - Must be verified before application can run

---

### 21. Clerk Integration Status

**Status:** UNVERIFIED

**Description:**
- Clerk integration has not been tested
- Clerk keys have not been configured
- Unknown if authentication works

**Impact:** Authentication may not work

**Evidence:** No .env file found; Clerk not tested

**Resolution Required:**
Configure Clerk keys and test authentication flow.

**Priority:** P1 - Must be verified before application can run

---

### 22. UploadThing Integration Status

**Status:** UNVERIFIED

**Description:**
- UploadThing integration has not been tested
- UploadThing token has not been configured
- Unknown if file uploads work

**Impact:** File uploads may not work

**Evidence:** No .env file found; UploadThing not tested

**Resolution Required:**
Configure UploadThing token and test file uploads.

**Priority:** P1 - Must be verified before application can run

---

## Unresolved Questions

### 23. Self-Likes Allowed

**Status:** QUESTION

**Description:**
- Current implementation allows users to like their own posts
- Unknown if this is intentional or a bug

**Impact:** User experience inconsistency

**Evidence:** toggleLike function does not prevent self-likes

**Resolution Required:**
Decide if self-likes should be prevented and implement if needed.

**Priority:** P3 - User experience decision

---

### 24. Notification Read Status Persistence

**Status:** QUESTION

**Description:**
- Notifications are marked as read when viewed
- Unknown if read status persists across sessions

**Impact:** User experience

**Evidence:** markNotificationsAsRead updates database, should persist

**Resolution Required:**
Test notification read status persistence.

**Priority:** P3 - Testing required

---

### 25. Follow Relationship Limits

**Status:** QUESTION

**Description:**
- Unknown if there are limits on follow relationships
- No maximum followers/following limit configured

**Impact:** Potential performance issues at scale

**Evidence:** No limits found in code or database

**Resolution Required:**
Decide if follow limits are needed and implement if required.

**Priority:** P3 - Performance consideration

---

## Dependencies

### 26. Dependency Versions

**Status:** MONITORING

**Description:**
- Dependencies should be kept updated
- Security vulnerabilities may exist in current versions

**Impact:** Potential security vulnerabilities

**Evidence:** package.json lists specific versions

**Resolution Required:**
Run `npm audit` regularly and update dependencies.

**Priority:** P3 - Ongoing maintenance

---

## Documentation Gaps

### 27. No API Documentation

**Status:** DOCUMENTATION

**Description:**
- No API documentation for external integrations
- No OpenAPI/Swagger specification

**Impact:** Difficult for external developers to integrate

**Evidence:** No API documentation found

**Resolution Required:**
Create API documentation if external integrations are planned.

**Priority:** P4 - Documentation enhancement

---

### 28. No Contribution Guidelines

**Status:** DOCUMENTATION

**Description:**
- No CONTRIBUTING.md file
- No contribution guidelines for external developers

**Impact:** Difficult for external contributors

**Evidence:** No CONTRIBUTING.md found

**Resolution Required:**
Create CONTRIBUTING.md if accepting external contributions.

**Priority:** P4 - Documentation enhancement

---

## Summary

### Critical (Must Fix Before Production)

1. Database not initialized
2. Application build status unverified
3. Database connection unverified
4. Clerk integration unverified
5. UploadThing integration unverified

### High Priority (Should Fix Before Production)

6. Orphaned file storage
7. No rate limiting
8. Repository name mismatch

### Medium Priority (Should Fix for Production Security)

9. No input sanitization
10. No content moderation
11. No automated security scanning
12. Error logging to console
13. No backup strategy documented

### Low Priority (Feature Enhancements)

14. No search functionality
15. No video uploads
16. No user blocking
17. No admin panel
18. No repost/share
19. No stories
20. No direct messages
21. No bookmarks
22. Tasks page is demo code

---

## Next Steps

1. **Immediate (P0):**
   - Run database migrations
   - Configure environment variables
   - Test application locally

2. **Before Production (P1-P2):**
   - Fix orphaned file storage
   - Implement rate limiting
   - Resolve repository name
   - Implement input sanitization

3. **Post-Deployment (P3-P4):**
   - Add automated security scanning
   - Implement structured logging
   - Set up backups
   - Add missing features as needed

---

## Resources

For detailed information on:
- Architecture: See [ARCHITECTURE.md](./ARCHITECTURE.md)
- Security: See [SECURITY.md](./SECURITY.md)
- Database: See [DATABASE_GUIDE.md](./DATABASE_GUIDE.md)
- Deployment: See [DEPLOYMENT_GUIDE.md](./DEPLOYMENT_GUIDE.md)
