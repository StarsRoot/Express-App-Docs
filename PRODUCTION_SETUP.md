# Production Setup Guide for Driver Account Creation

## Overview

This guide explains how to set up driver account creation for production use with Clerk authentication.

## Prerequisites

1. **Clerk Account**: You need a Clerk account at https://clerk.com
2. **Clerk Application**: Create a new application in your Clerk dashboard
3. **API Keys**: Get your Clerk Secret Key from the dashboard

## Step 1: Configure Clerk Secret Key

### Get Your Clerk Secret Key

1. Log in to [Clerk Dashboard](https://dashboard.clerk.com)
2. Select your application
3. Navigate to **API Keys** in the sidebar
4. Copy your **Secret Key** (starts with `sk_test_` for test mode or `sk_live_` for production)

### Set Environment Variable

#### For Backend (`.env` file)

```bash
# Clerk Configuration (REQUIRED for production)
CLERK_SECRET_KEY=sk_live_your_actual_secret_key_here

# Environment
NODE_ENV=production
```

**Important**: 
- Replace `test_secret_key` or any placeholder with your actual Clerk secret key
- Use `sk_live_` keys for production environments
- Keep this key secure and never commit it to version control

#### Verify Configuration

The application will validate the Clerk key on startup:
- ❌ If `CLERK_SECRET_KEY` is missing or is a placeholder → Application will fail in production
- ✅ If `CLERK_SECRET_KEY` is a valid Clerk key → Application will proceed normally

## Step 2: Clerk Application Settings

### Enable Password Authentication

1. In Clerk Dashboard, go to **User & Authentication**
2. Navigate to **Email, Phone, Username**
3. Ensure **Email** is enabled as a sign-in method
4. Under **Email settings**, enable **Password** authentication
5. Configure password requirements:
   - Minimum length: 8 characters
   - Require uppercase, lowercase, and numbers (recommended)

### Configure User Metadata

1. Go to **User & Authentication** → **Metadata**
2. Ensure **Public metadata** is enabled
3. This allows storing the `role: 'driver'` metadata

### Skip Email Verification (Optional)

For driver accounts, you may want to skip email verification:

1. In Clerk Dashboard, go to **User & Authentication**
2. Under **Email** settings, you can configure:
   - Auto-verify emails for programmatically created users
   - Or allow unverified emails to sign in (development only)

**Note**: In production, email verification is recommended for security.

## Step 3: Testing Driver Creation

### Test the Flow

1. **Create a Driver**:
   - Admin/Frontdesk creates driver via UI
   - System creates Clerk user account
   - Credentials are displayed to admin

2. **Driver Login**:
   - Driver goes to `/sign-in`
   - Uses email and password provided by admin
   - Should successfully authenticate

3. **Verify Access**:
   - Driver should see their assigned packages
   - Driver can update package status
   - Driver can access driver portal

### Common Issues

#### Issue: "Unauthorized" Error

**Cause**: Invalid or missing Clerk secret key

**Solution**:
```bash
# Check your .env file
cat apps/backend/.env | grep CLERK_SECRET_KEY

# Should show: CLERK_SECRET_KEY=sk_test_... or sk_live_...
# NOT: CLERK_SECRET_KEY=test_secret_key
```

#### Issue: Password Setting Fails

**Cause**: Clerk API doesn't support direct password setting in some configurations

**Solution**:
- The system will still create the user
- Password will be included in the error message
- Admin can share password with driver manually
- Driver will need to use password reset on first login (if enabled)

#### Issue: Driver Can't See Packages

**Cause**: Clerk ID mismatch

**Solution**:
- Verify driver's `clerkId` in database matches their Clerk user ID
- Check package `assignedTo` field uses driver's Clerk ID
- Verify driver is logged in and has `role: 'driver'` in Clerk metadata

## Step 4: Production Checklist

Before deploying to production:

- [ ] Valid Clerk Secret Key configured (`sk_live_...`)
- [ ] Clerk application configured for password authentication
- [ ] Email verification configured appropriately
- [ ] Driver role metadata enabled in Clerk
- [ ] Test driver creation flow end-to-end
- [ ] Test driver login with created credentials
- [ ] Verify driver can access their portal
- [ ] Verify driver can see assigned packages
- [ ] Set up error monitoring (Sentry, etc.)
- [ ] Configure logging for Clerk API calls
- [ ] Set up alerts for Clerk API failures

## Security Best Practices

1. **Environment Variables**: Never commit `.env` files to git
2. **Secret Keys**: Rotate Clerk secret keys periodically
3. **Password Policy**: Enforce strong password requirements
4. **Access Control**: Use Clerk's role-based access control
5. **Audit Logs**: Enable Clerk audit logs for user creation events
6. **Rate Limiting**: Configure rate limiting on driver creation endpoints

## Monitoring

### Key Metrics to Monitor

- Driver creation success rate
- Clerk API response times
- Failed authentication attempts
- Password reset requests
- Driver login success rate

### Error Handling

The application logs all Clerk API errors:
- Check backend logs for Clerk-related errors
- Monitor Clerk dashboard for API usage and errors
- Set up alerts for repeated failures

## Support

For Clerk-specific issues:
- Clerk Documentation: https://clerk.com/docs
- Clerk Support: https://clerk.com/support

For application issues:
- Check application logs
- Verify environment variables
- Test with Clerk's test API keys first

