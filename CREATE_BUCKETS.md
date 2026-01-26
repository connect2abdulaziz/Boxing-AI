# Creating Storage Buckets - Quick Fix

The error "Bucket not found" means you need to create the storage buckets first.

## Option 1: Using Supabase Dashboard (Easiest)

1. Go to your Supabase Dashboard
2. Navigate to **Storage** in the left sidebar
3. Click **"New bucket"**
4. Create bucket **"videos"**:
   - Name: `videos`
   - Public: **No** (private)
   - File size limit: 100MB
   - Allowed MIME types: `video/mp4`, `video/mpeg`, `video/quicktime`, `video/x-msvideo`, `video/webm`
5. Click **"Create bucket"**
6. Create bucket **"results"**:
   - Name: `results`
   - Public: **Yes** (public - for frontend polling)
   - File size limit: 1MB
   - Allowed MIME types: `application/json`
7. Click **"Create bucket"**

## Option 2: Using SQL (Recommended for Local)

### For Local Supabase:

```bash
cd supabase
supabase db reset  # This will run all migrations including the bucket creation
```

Or run the SQL directly:

```bash
supabase db execute --file supabase/migrations/20240101000000_create_storage_buckets.sql
```

### For Production:

1. Go to Supabase Dashboard → **SQL Editor**
2. Run this SQL:

```sql
-- Create 'videos' bucket (private)
INSERT INTO storage.buckets (id, name, public, file_size_limit, allowed_mime_types)
VALUES (
  'videos',
  'videos',
  false,
  104857600, -- 100MB
  ARRAY['video/mp4', 'video/mpeg', 'video/quicktime', 'video/x-msvideo', 'video/webm']
)
ON CONFLICT (id) DO NOTHING;

-- Create 'results' bucket (public)
INSERT INTO storage.buckets (id, name, public, file_size_limit, allowed_mime_types)
VALUES (
  'results',
  'results',
  true,
  1048576, -- 1MB
  ARRAY['application/json']
)
ON CONFLICT (id) DO NOTHING;
```

## Option 3: Using Supabase CLI

```bash
cd supabase

# Apply the migration
supabase migration up

# Or if you want to reset everything (local only)
supabase db reset
```

## Verify Buckets Exist

### Via Dashboard:
- Go to Storage → You should see both `videos` and `results` buckets

### Via SQL:
```sql
SELECT id, name, public FROM storage.buckets;
```

You should see:
```
id      | name    | public
--------|---------|--------
videos  | videos  | false
results | results | true
```

## After Creating Buckets

1. **Test the upload again** - the error should be gone
2. **Check bucket permissions** - ensure service role can upload to `videos`
3. **Verify results bucket is public** - frontend needs to read from it

## Troubleshooting

### Still getting "Bucket not found"?
- Verify bucket name matches exactly: `videos` (case-sensitive)
- Check you're using the correct Supabase project
- For local: ensure `supabase start` is running
- Check bucket exists: `SELECT * FROM storage.buckets WHERE id = 'videos';`

### Permission errors?
- Ensure SERVICE_ROLE_KEY is set correctly
- Check storage policies allow service role access
- For local: service role should have full access by default
