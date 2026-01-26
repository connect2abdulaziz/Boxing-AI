# Database Setup - Analysis Jobs Table

## Run Migration

To create the `analysis_jobs` table, run the migration:

### Local Development

```bash
cd supabase
supabase db reset  # This runs all migrations
# OR
supabase migration up
```

### Production

1. Go to Supabase Dashboard → SQL Editor
2. Run the SQL from `supabase/migrations/20240101000001_create_analysis_jobs.sql`

Or via CLI:
```bash
supabase db push
```

## Verify Table Created

```sql
-- Check table exists
SELECT * FROM analysis_jobs LIMIT 1;

-- Check structure
\d analysis_jobs
```

## Table Schema

```sql
analysis_jobs
├── id (UUID, primary key)
├── job_id (TEXT, unique) - The job identifier
├── video_path (TEXT) - Path in storage bucket
├── mode (TEXT) - 'bag', 'pads', or 'sparring'
├── strategy (TEXT) - 'interval-8', 'interval-9', or 'smart'
├── status (TEXT) - 'pending', 'processing', 'completed', 'failed'
├── result (JSONB) - Analysis result from OpenAI
├── frames_count (INTEGER) - Number of frames extracted
├── error_message (TEXT) - Error if failed
├── created_at (TIMESTAMPTZ)
├── started_at (TIMESTAMPTZ)
└── completed_at (TIMESTAMPTZ)
```

## Row Level Security (RLS)

- **Service role**: Full access (for worker and Edge Function)
- **Public/Anon**: Can read jobs (for frontend polling)

You can restrict this later by adding `user_id` and user-specific policies.

## Query Examples

### Get job by job_id
```sql
SELECT * FROM analysis_jobs WHERE job_id = 'your-job-id';
```

### Get all completed jobs
```sql
SELECT * FROM analysis_jobs WHERE status = 'completed' ORDER BY created_at DESC;
```

### Get failed jobs
```sql
SELECT job_id, error_message, created_at 
FROM analysis_jobs 
WHERE status = 'failed' 
ORDER BY created_at DESC;
```
