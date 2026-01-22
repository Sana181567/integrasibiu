# Security Configuration Guide

This document explains how to complete the security setup for your INTEGRA application.

## ✅ Recently Fixed

The following security issues have been resolved through database migrations:

1. **RLS Policy Security (Fixed)** - Removed the insecure anonymous INSERT policy on `contact_submissions` table
   - Contact forms now MUST use the Edge Function for secure, validated submissions
   - Direct database insertion by anonymous users is now blocked
   - Migration: `20260121_remove_insecure_anon_insert_policy.sql`

## ⚠️ Manual Configuration Required

The following configurations require manual setup through the Supabase Dashboard:

## 1. Configure Auth DB Connection Strategy

The Auth server needs to be configured to use a percentage-based connection allocation strategy instead of a fixed number.

### Steps:
1. Go to your [Supabase Dashboard](https://app.supabase.com)
2. Select your project
3. Navigate to **Settings** (gear icon in sidebar)
4. Click on **Database** in the left menu
5. Scroll to **Connection pooling** section
6. Find the **Auth** connection pool settings
7. Change from **Fixed** (e.g., 10 connections) to **Percentage-based** allocation
8. Recommended: Set to **10-20%** of total available connections
9. Click **Save** to apply changes

### Why this matters:
- Percentage-based allocation automatically scales with your database instance size
- Fixed connections limit Auth performance even when you upgrade your database
- Better resource utilization during high traffic periods

---

## 2. Enable Leaked Password Protection

Supabase Auth can prevent users from setting passwords that have been compromised in known data breaches by checking against the HaveIBeenPwned.org database.

### Steps:
1. Go to your [Supabase Dashboard](https://app.supabase.com)
2. Select your project
3. Navigate to **Authentication** (shield icon in sidebar)
4. Click on **Settings** tab
5. Scroll to the **Password Protection** section
6. Find the **"Leaked Password Protection"** toggle
7. **Enable** the toggle (turn it ON)
8. Changes are saved automatically

### Why this matters:
- Prevents users from using passwords that have been exposed in data breaches
- Significantly improves account security
- Protects users from credential stuffing attacks
- Industry best practice for password security

### Testing:
After enabling, try creating a test account with a known weak password like `password123`. The system should reject it with a message about the password being compromised.

---

## 3. Set Up Admin User Access

The admin panel at `/admin` requires authenticated users with admin role. Follow these steps to grant admin access:

### Option A: Using Supabase Dashboard (Recommended)

1. Go to your [Supabase Dashboard](https://app.supabase.com)
2. Select your project
3. Navigate to **Authentication** > **Users**
4. Either create a new user or select an existing user
5. Click on the user to open their details
6. Scroll to **User Metadata** section
7. Find **App Metadata** (not User Metadata)
8. Add the following JSON:
   ```json
   {
     "role": "admin"
   }
   ```
9. Click **Save**

### Option B: Using SQL (Advanced)

If you prefer SQL, you can run this query in the SQL Editor:

```sql
-- Replace 'admin@integra.ro' with your admin email
UPDATE auth.users
SET raw_app_meta_data = raw_app_meta_data || '{"role": "admin"}'::jsonb
WHERE email = 'admin@integra.ro';
```

### Creating a New Admin User via SQL:

```sql
-- This requires the service role key or dashboard access
-- Replace with your admin credentials
INSERT INTO auth.users (
  email,
  encrypted_password,
  email_confirmed_at,
  raw_app_meta_data,
  raw_user_meta_data,
  is_super_admin,
  role
)
VALUES (
  'admin@integra.ro',
  crypt('your-secure-password', gen_salt('bf')),
  now(),
  '{"role": "admin"}'::jsonb,
  '{}'::jsonb,
  false,
  'authenticated'
);
```

However, **we recommend creating users through the Supabase Dashboard or auth.signUp() method** instead of direct SQL insertion.

---

## 4. Verify Admin Access

After setting up an admin user:

1. Visit your application's `/admin` page
2. Log in with the admin credentials
3. You should now see the contact form submissions
4. Verify you can delete submissions

---

## 5. Security Best Practices

### Row Level Security (RLS)
- RLS is enabled on the `contact_submissions` table
- Only users with `app_metadata.role = 'admin'` can view and delete submissions
- Anonymous users CANNOT insert directly (security improvement)
- Contact form submissions go through the Edge Function `send-contact-email` which uses service_role for validated insertions

### Admin Credentials
- Use strong passwords (minimum 12 characters, mix of letters, numbers, symbols)
- Store admin credentials securely (use password manager)
- Limit admin accounts to only necessary personnel
- Regularly review and revoke access for former team members

### Environment Variables
- Never commit `.env` file to version control
- Keep `VITE_SUPABASE_ANON_KEY` public (it's safe for client-side use)
- Never expose `SUPABASE_SERVICE_ROLE_KEY` in client-side code
- Rotate keys if they are ever compromised

---

## Troubleshooting

### "Not authorized" when accessing admin panel
- Verify the user has `app_metadata.role = 'admin'` set
- Check you're logged in with the correct account
- Clear browser cache and cookies, then log in again

### Auth connection errors
- Ensure Auth DB connection strategy is set to percentage-based
- Check your database instance has sufficient capacity
- Monitor connection pool usage in Supabase Dashboard

### Contact form not working
- Verify the `send-contact-email` Edge Function is deployed and accessible
- Check that the Edge Function has proper CORS headers configured
- Verify Edge Function has access to SUPABASE_SERVICE_ROLE_KEY environment variable
- Test the Edge Function endpoint directly: `/functions/v1/send-contact-email`
- Check browser console for any CORS or network errors

---

## Support

For additional help:
- Email: integrasibiu@gmail.com
- Phone: 0775582002
- Supabase Documentation: https://supabase.com/docs
