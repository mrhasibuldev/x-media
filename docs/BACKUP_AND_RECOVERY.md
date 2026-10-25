# Backup and Recovery Guide

## Overview

This guide explains backup and recovery procedures for Socially.

## What Needs to Be Backed Up

### 1. Database (PostgreSQL)

**What to Backup:**
- User accounts
- Posts, comments, likes
- Follow relationships
- Notifications

**Importance:** Critical - All user data stored here

**Frequency:** Daily (minimum), hourly (recommended)

---

### 2. Files (UploadThing)

**What to Backup:**
- User-uploaded images

**Importance:** Medium - Can be re-uploaded, but data loss is inconvenient

**Note:** UploadThing may provide built-in backups. Check their documentation.

---

### 3. Source Code (Git)

**What to Backup:**
- Application source code
- Configuration files
- Documentation

**Importance:** Critical - Intellectual property

**Frequency:** Every commit (automatic via Git)

**Note:** Git is not a backup for production data, only for code.

---

### 4. Environment Variables

**What to Backup:**
- Clerk keys
- Database credentials
- UploadThing token

**Importance:** Critical - Required for application to function

**Storage:** Secure password manager or secrets manager

**Note:** Never commit to Git.

---

## Database Backup Procedures

### Manual Backup (pg_dump)

**Using pg_dump:**

```bash
# Backup entire database
pg_dump socially > backup_$(date +%Y%m%d_%H%M%S).sql

# Backup with compression
pg_dump socially | gzip > backup_$(date +%Y%m%d_%H%M%S).sql.gz

# Backup only schema
pg_dump --schema-only socially > schema_backup.sql

# Backup only data
pg_dump --data-only socially > data_backup.sql
```

**Example:**
```bash
pg_dump postgresql://postgres:password@localhost:5432/socially > backup.sql
```

---

### Automated Backup (cron)

**Linux/macOS:**

Add to crontab:
```bash
# Daily backup at 2 AM
0 2 * * * pg_dump postgresql://postgres:password@localhost:5432/socially | gzip > /backups/socially_$(date +\%Y\%m\%d).sql.gz
```

**Windows:**
- Use Task Scheduler
- Schedule pg_dump command

---

### Cloud Database Backups

**Supabase:**

Supabase provides automated backups:
1. Navigate to project dashboard
2. Go to "Database" → "Backups"
3. Configure automated backups
4. Manual backups available on demand

**Neon:**

Neon provides automated backups:
1. Navigate to project dashboard
2. Backups are automatic
3. Point-in-time recovery available

**Railway:**

Railway provides automated backups:
1. Navigate to database service
2. Backups are automatic
3. Manual snapshots available

---

### Prisma Migrations Backup

**Location:** `prisma/migrations/`

**Importance:** Essential for recreating database schema

**Backup:** Already version-controlled in Git

**Note:** Migrations should always be committed to Git.

---

## Database Recovery Procedures

### Restore from pg_dump Backup

**Using psql:**

```bash
# Restore from SQL file
psql socially < backup.sql

# Restore from compressed backup
gunzip < backup.sql.gz | psql socially
```

**Example:**
```bash
psql postgresql://postgres:password@localhost:5432/socially < backup_20240101.sql
```

---

### Restore from Cloud Backup

**Supabase:**
1. Navigate to project dashboard
2. Go to "Database" → "Backups"
3. Select backup to restore
4. Click "Restore"

**Neon:**
1. Navigate to project dashboard
2. Use point-in-time recovery
3. Select timestamp to restore to

**Railway:**
1. Navigate to database service
2. Select snapshot to restore
3. Click "Restore"

---

### Recreate Database from Migrations

**If backup is not available:**

```bash
# Drop existing database (WARNING: deletes all data)
DROP DATABASE socially;

# Recreate database
CREATE DATABASE socially;

# Run migrations
npx prisma migrate deploy
```

**Note:** This recreates schema but loses all data.

---

## File Storage Backup

### UploadThing Backup

**Current Status:**
- UploadThing may provide built-in backups
- Check UploadThing documentation for backup options
- Not verified in this audit

**Alternative:**
- Download all files periodically
- Store in separate storage (e.g., S3, local backup)

**Note:** File URLs in database would need to be updated if restoring to different storage.

---

## Source Code Backup

### Git as Backup

**Repository:**
- Source code is version-controlled in Git
- Every commit is a backup
- Remote repository (GitHub) provides additional backup

**Best Practices:**
- Commit frequently
- Push to remote repository regularly
- Use meaningful commit messages
- Tag releases

**Note:** Git is not a backup for production data, only for code.

---

### Manual Source Code Backup

**Export Repository:**
```bash
# Create zip archive
git archive --format=zip --output=backup.zip HEAD
```

**Copy Project Directory:**
- Copy entire project directory to backup location
- Include node_modules if needed (can be restored with npm install)

---

## Environment Variables Backup

### Secure Storage

**Recommended:**
- Password manager (1Password, Bitwarden, LastPass)
- Secrets manager (AWS Secrets Manager, HashiCorp Vault)
- Encrypted file (GPG encrypted)

**What to Store:**
- Clerk keys
- Database credentials
- UploadThing token
- Any other API keys

**Example (Encrypted File):**
```bash
# Encrypt environment file
gpg --symmetric --cipher-algo AES256 .env

# Decrypt when needed
gpg --decrypt .env.gpg > .env
```

---

## Disaster Recovery Plan

### Scenario 1: Database Corruption

**Symptoms:**
- Application crashes
- Database errors
- Data appears corrupted

**Recovery Steps:**
1. Stop application
2. Identify last good backup
3. Restore database from backup
4. Verify data integrity
5. Restart application
6. Test critical functionality

**Estimated Downtime:** 1-2 hours

---

### Scenario 2: Accidental Data Deletion

**Symptoms:**
- User reports missing data
- Database tables empty
- Critical data lost

**Recovery Steps:**
1. Stop application
2. Identify when deletion occurred
3. Restore from backup before deletion
4. Verify data restored
5. Restart application
6. Communicate with affected users

**Estimated Downtime:** 1-2 hours

---

### Scenario 3: Server Failure

**Symptoms:**
- Server unresponsive
- Cannot access application
- Hardware failure

**Recovery Steps:**
1. Deploy to backup server (if available)
2. Restore database from backup
3. Update DNS if needed
4. Verify application functionality
5. Monitor for issues

**Estimated Downtime:** 2-4 hours

---

### Scenario 4: Security Breach

**Symptoms:**
- Unauthorized access detected
- Data breach suspected
- Security incident reported

**Recovery Steps:**
1. Immediately shut down application
2. Rotate all credentials (API keys, database passwords)
3. Restore from pre-breach backup
4. Investigate breach source
5. Patch vulnerabilities
6. Notify affected users
7. Restart application with new credentials

**Estimated Downtime:** 4-8 hours

---

## Backup Testing

### Regular Testing

**Frequency:** Monthly

**Test Procedure:**
1. Create test database
2. Restore backup to test database
3. Verify data integrity
4. Test application with restored data
5. Document results

**Automated Testing:**
- Script restore process
- Run weekly or monthly
- Alert on failure

---

## Backup Storage Locations

### Local Storage

**Pros:**
- Fast access
- No ongoing cost
- Full control

**Cons:**
- Single point of failure
- Vulnerable to local disasters
- Requires maintenance

**Recommendation:** Use for temporary backups only

---

### Cloud Storage

**Recommended Services:**
- AWS S3
- Google Cloud Storage
- Azure Blob Storage
- Backblaze B2

**Pros:**
- Redundant storage
- Geographic distribution
- Scalable
- Cost-effective

**Cons:**
- Ongoing cost
- Requires configuration

**Recommendation:** Use for primary backup storage

---

### Hybrid Approach

**Strategy:**
- Local backup for quick recovery
- Cloud backup for disaster recovery
- Geographic distribution for high availability

---

## Backup Retention Policy

### Recommended Retention

**Daily Backups:** Keep for 30 days
**Weekly Backups:** Keep for 12 weeks
**Monthly Backups:** Keep for 12 months
**Yearly Backups:** Keep for 7 years

**Total Storage:** Approximately 50-100GB for typical social media application

---

## Monitoring and Alerts

### Backup Monitoring

**What to Monitor:**
- Backup completion status
- Backup file size
- Backup duration
- Backup success/failure rate

**Alerts:**
- Backup failure
- Backup too small (possible corruption)
- Backup taking too long
- Storage running low

---

### Automated Monitoring Tools

**Options:**
- Custom scripts with email alerts
- Monitoring services (Datadog, New Relic)
- Cloud provider monitoring (AWS CloudWatch, GCP Monitoring)

---

## Documentation

### Backup Documentation

**Document:**
- Backup procedures
- Recovery procedures
- Contact information
- Critical systems
- Vendor information

**Storage:**
- Keep with backups
- Store offsite
- Update regularly

---

## Best Practices

### Do

- Automate backups
- Test backups regularly
- Store backups in multiple locations
- Encrypt backups containing sensitive data
- Monitor backup status
- Have a disaster recovery plan
- Document all procedures
- Keep backups for appropriate retention period

### Don't

- Don't rely on a single backup location
- Don't forget to test backups
- Don't store unencrypted backups with sensitive data
- Don't ignore backup failures
- Don't skip backups to save time
- Don't forget to backup environment variables
- Don't assume backups will work without testing

---

## Next Steps

1. Implement automated database backups
2. Test backup and recovery procedures
3. Set up monitoring and alerts
4. Document all procedures
5. Train team on recovery procedures

## Resources

- **PostgreSQL Backup:** https://www.postgresql.org/docs/current/backup.html
- **Supabase Backups:** https://supabase.com/docs/guides/platform/backups
- **Neon Backups:** https://neon.tech/docs/introduction/backups
- **AWS S3:** https://aws.amazon.com/s3/
