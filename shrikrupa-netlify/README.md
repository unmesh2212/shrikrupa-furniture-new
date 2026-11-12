# Shrikrupa Furniture Works — Netlify Edition

A fresh modern storefront and admin portal built for Netlify Functions + Supabase + Resend. It uses the existing Supabase tables (`admin`, `products`, `categories`, `enquiries`, `business`, `settings`) and performs row-level insert/update/delete operations. It does **not** run the old destructive `writeDB()` sync and does not run any migration automatically.

## Before deployment

1. Back up the existing Supabase project. Do not run `migrate-db.js` from the old project.
2. In Supabase Storage, create a **public** bucket named `product-images` (or set `SUPABASE_STORAGE_BUCKET` to the bucket you choose). The function uses the server-side Supabase secret to upload files; never put that key in browser code.
3. Confirm the existing `admin` row has the same admin email and a valid `password_hash` in the legacy scrypt format. The default admin email is `unmeshbutte@gmail.com`; override with `ADMIN_EMAIL` if the existing account uses another address.
4. Make sure the existing table columns match the old project. Expected columns are described below. This project intentionally does not create or alter tables automatically.

## Netlify setup

1. Push this folder's contents to a **new GitHub repository** (recommended) so the old Render site and old repo remain untouched while testing.
2. In Netlify, Add new site → Import an existing project → select the new repository.
3. Build settings: build command `npm run build`; publish directory `public`; functions directory `netlify/functions` (already in `netlify.toml`).
4. Add these Environment Variables in Netlify → Site configuration → Environment variables:

| Variable | Value |
|---|---|
| `SUPABASE_URL` | Existing Supabase project URL |
| `SUPABASE_SECRET_KEY` | Supabase server-side secret/service-role key (never expose publicly) |
| `SESSION_SECRET` | Long random secret, at least 32 random bytes |
| `RESEND_API_KEY` | New Resend API key with Sending access |
| `RESEND_FROM_EMAIL` | Verified sender, e.g. `Shrikrupa Furniture Works <hello@yourdomain.com>` |
| `ADMIN_EMAIL` | Existing admin email, default `unmeshbutte@gmail.com` |
| `SUPABASE_STORAGE_BUCKET` | `product-images` |

5. Trigger a deploy. Test `https://YOUR-SITE.netlify.app/api/health` and confirm it returns `{"ok":true,"service":"Shrikrupa API"}`.
6. Test public products and categories, submit a test enquiry, then test admin login and OTP reset.

## OTP / Resend domain requirement

Resend's `onboarding@resend.dev` sender is for restricted testing, not production customer/admin delivery. For reliable OTP delivery, add a domain you own in Resend → Domains, publish the DNS records at your domain registrar, wait until Resend marks it verified, and set `RESEND_FROM_EMAIL` to an address on that verified domain. The OTP recipient is the configured `ADMIN_EMAIL`. Do not share API keys or passwords in screenshots or chat.

## Expected existing Supabase columns

- `admin`: `id`, `email`, `password_hash`, `session_version`, `reset_otp_hash`, `reset_otp_expires_at`
- `products`: `id`, `model_no`, `name`, `category`, `colour`, `description`, `image`, `colours`, `featured`, `active`, `images`, `created_at`, `updated_at`
- `categories`: `id`, `name`
- `enquiries`: `id`, `name`, `phone`, `email`, `product`, `model_no`, `city`, `message`, `created_at`, `status`, `notes`
- `settings`: `id`, `maintenance_mode`, `maintenance_message`

If your live schema differs, stop before deploying and adjust the function to the actual schema. No destructive schema changes are included.

## Features

- Modern responsive furniture storefront, category filters, product cards, product-specific enquiries
- Contact/enquiry form stored in existing `enquiries` table
- Hidden-from-navigation admin portal at `/admin.html`
- Admin password login, email OTP reset, 10-minute OTP expiry
- Product create/delete, multiple image upload to Supabase Storage
- Category create/delete (deletion blocked while products use the category)
- Enquiry list/status updates
- Maintenance mode settings
- Server-only Supabase secret and Resend key

## Important limitations / testing status

This project was generated from the old source's known table/column conventions, but it has not been connected to your live Supabase or Resend account and has not been deployed to your Netlify account. Therefore live OTP delivery, admin login, storage permissions, and schema compatibility must be tested before replacing the live site. The old Render website remains separate and can stay online during testing.
