# Boxing AI MVP - Setup Guide

## Architecture

```
Frontend (Next.js) 
  → Uploads video to Supabase Edge Function
  → Edge Function uploads to Supabase Storage
  → Edge Function triggers Node.js Worker
  → Worker: Downloads video → Extracts frames (FFmpeg) → Analyzes (OpenAI) → Saves result
  → Frontend polls for results
```

## Prerequisites

- Node.js 20+
- FFmpeg installed
- Supabase CLI (`supabase start`)
- OpenAI API key

## Setup Steps

### 1. Database Migration

**Create the `analysis_jobs` table:**

```bash
cd supabase
supabase db reset  # Runs all migrations (local)
# OR
supabase migration up
```

**For production:** Run the migration SQL in Supabase Dashboard → SQL Editor:
- File: `supabase/migrations/20240101000001_create_analysis_jobs.sql`

See `DATABASE_SETUP.md` for details.

### 2. Supabase Storage

Create storage buckets:

```bash
# In Supabase Dashboard or via SQL:
# Storage → Create bucket: "videos" (public or private)
# Storage → Create bucket: "results" (public for polling)
```

Or via SQL in Supabase:

```sql
-- Create buckets
INSERT INTO storage.buckets (id, name, public) VALUES ('videos', 'videos', false);
INSERT INTO storage.buckets (id, name, public) VALUES ('results', 'results', true);
```

### 3. Node.js Worker

```bash
cd boxing-ai-worker
npm install
cp .env.example .env
# Edit .env with your keys
npm run dev  # or npm start for production
```

**Environment variables:**
- `SUPABASE_URL` - Your Supabase URL
- `SUPABASE_SERVICE_ROLE_KEY` - Service role key (for storage access)
- `OPENAI_API_KEY` - OpenAI API key
- `PORT` - Worker port (default: 3001)
- `STORAGE_BUCKET` - Storage bucket name (default: "videos")

### 4. Supabase Edge Function

```bash
cd supabase
# Set secrets (note: cannot use SUPABASE_ prefix)
supabase secrets set SERVICE_ROLE_KEY=your_service_role_key
supabase secrets set WORKER_URL=http://localhost:3001  # or your worker URL
supabase secrets set STORAGE_BUCKET=videos

# Deploy function
supabase functions deploy analyze
```

Or for local development:

```bash
supabase functions serve analyze --env-file supabase/.env.local
```

**Edge Function env vars:**
- `SUPABASE_URL` - Auto-set by Supabase (cannot be overridden)
- `SERVICE_ROLE_KEY` - Set via `supabase secrets set` (use this name, not SUPABASE_SERVICE_ROLE_KEY)
- `WORKER_URL` - Your Node.js worker URL
- `STORAGE_BUCKET` - Storage bucket name

### 5. Frontend

```bash
cd boxing-ai-frontend
npm install
cp .env.example .env.local
# Edit .env.local:
# NEXT_PUBLIC_SUPABASE_URL=http://127.0.0.1:54321
# NEXT_PUBLIC_SUPABASE_ANON_KEY=your_anon_key

npm run dev
```

### 6. Docker (Optional)

Build and run the worker:

```bash
cd boxing-ai-worker
docker build -t boxing-ai-worker .
docker run -p 3001:3001 --env-file .env boxing-ai-worker
```

## Testing

1. Start Supabase: `supabase start`
2. Start Worker: `cd boxing-ai-worker && npm run dev`
3. Start Frontend: `cd boxing-ai-frontend && npm run dev`
4. Upload a video in the frontend
5. Check worker logs for processing
6. Frontend will poll and display results

## Production Deployment

1. **Worker**: Deploy to a server with FFmpeg (or use Docker)
2. **Edge Function**: Deploy via `supabase functions deploy analyze`
3. **Frontend**: Deploy to Vercel/Netlify with env vars
4. **Storage**: Ensure buckets exist in production Supabase project

## Troubleshooting

- **Worker not receiving requests**: Check `WORKER_URL` in Edge Function secrets
- **Storage upload fails**: Verify bucket exists and service role key has access
- **FFmpeg errors**: Ensure FFmpeg is installed and in PATH
- **Polling not working**: Check `results` bucket is public and result JSON exists
