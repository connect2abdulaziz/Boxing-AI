# Deploying Supabase Edge Function

## Prerequisites

1. **Install Supabase CLI** (if not already installed):
   ```bash
   npm install -g supabase
   # or
   brew install supabase/tap/supabase
   ```

2. **Login to Supabase**:
   ```bash
   supabase login
   ```

3. **Link your project** (if deploying to production):
   ```bash
   cd supabase
   supabase link --project-ref your-project-ref
   ```
   Find your project ref in Supabase Dashboard → Settings → General

## Step 1: Get Your Service Role Key

1. Go to Supabase Dashboard → Settings → API
2. Copy the **Service Role Key** (not the anon key - this has admin access)
3. Keep it secure - never commit it to git

## Step 2: Set Secrets

### For Local Development

Create `supabase/.env.local`:

```bash
cd supabase
cp env.example .env.local
# Edit .env.local with your actual keys
```

```env
SERVICE_ROLE_KEY=your_service_role_key_here
WORKER_URL=http://localhost:3001
STORAGE_BUCKET=videos
```

### For Production Deployment

Set secrets via CLI:

```bash
cd supabase

# Set service role key
supabase secrets set SERVICE_ROLE_KEY=your_service_role_key_here

# Set worker URL (your production worker URL)
supabase secrets set WORKER_URL=https://your-worker-domain.com
# OR for local worker accessible from internet (ngrok, etc.)
supabase secrets set WORKER_URL=https://your-ngrok-url.ngrok.io

# Set storage bucket name
supabase secrets set STORAGE_BUCKET=videos
```

**Important**: Secrets are encrypted and stored securely. They're only accessible to your Edge Functions.

## Step 3: Deploy the Function

### Option A: Deploy to Production (Supabase Cloud)

```bash
cd supabase
supabase functions deploy analyze
```

This will:
- Build and deploy the function
- Make it available at: `https://your-project.supabase.co/functions/v1/analyze`
- Use the secrets you set via `supabase secrets set`

### Option B: Local Development

```bash
cd supabase

# Start Supabase locally (if not already running)
supabase start

# Serve the function locally
supabase functions serve analyze --env-file .env.local
```

The function will be available at: `http://127.0.0.1:54321/functions/v1/analyze`

## Step 4: Verify Deployment

### Check Function Status

```bash
# List deployed functions
supabase functions list

# View function logs
supabase functions logs analyze
```

### Test the Function

**Local:**
```bash
curl -X POST http://127.0.0.1:54321/functions/v1/analyze \
  -H "Authorization: Bearer YOUR_ANON_KEY" \
  -F "video=@test-video.mp4" \
  -F "mode=bag" \
  -F "strategy=interval-8"
```

**Production:**
```bash
curl -X POST https://your-project.supabase.co/functions/v1/analyze \
  -H "Authorization: Bearer YOUR_ANON_KEY" \
  -F "video=@test-video.mp4" \
  -F "mode=bag" \
  -F "strategy=interval-8"
```

## Step 5: Update Frontend

Update your frontend `.env.local`:

**Local:**
```env
NEXT_PUBLIC_SUPABASE_URL=http://127.0.0.1:54321
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_local_anon_key
```

**Production:**
```env
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_production_anon_key
```

## Troubleshooting

### Function not found
- Ensure you're in the `supabase` directory
- Check function name matches: `analyze`

### Secrets not working
- Verify secrets are set: `supabase secrets list`
- Check secret names match exactly (case-sensitive)
- For local: ensure `.env.local` exists and is loaded

### CORS errors
- Edge Function already includes CORS headers
- Check browser console for specific error
- Verify `Authorization` header is included in requests

### Storage upload fails
- Ensure `videos` bucket exists in Supabase Storage
- Verify SERVICE_ROLE_KEY has storage access
- Check bucket permissions in Dashboard

### Worker not receiving requests
- Verify WORKER_URL is correct and accessible
- Check worker is running: `curl http://localhost:3001/health`
- For production: ensure worker URL is publicly accessible

## Updating the Function

After making changes to `supabase/functions/analyze/index.ts`:

```bash
cd supabase
supabase functions deploy analyze
```

The function will be redeployed with your changes.

## Viewing Logs

```bash
# Real-time logs
supabase functions logs analyze --follow

# Recent logs
supabase functions logs analyze --limit 50
```

## Environment Variables Reference

| Variable | Source | Description |
|----------|--------|-------------|
| `SUPABASE_URL` | Auto-provided | Your Supabase project URL |
| `SERVICE_ROLE_KEY` | Secret | Service role key for admin access |
| `WORKER_URL` | Secret | Node.js worker URL |
| `STORAGE_BUCKET` | Secret | Storage bucket name (default: "videos") |

## Security Notes

- ✅ **Service Role Key**: Has admin access - keep it secret
- ✅ **Secrets**: Encrypted in Supabase - safe to use
- ✅ **CORS**: Currently set to `*` - restrict in production
- ⚠️ **Worker URL**: Ensure it's secure (HTTPS in production)
