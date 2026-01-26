# Testing the Worker Manually

## 1. Check Worker is Running

First, verify your worker is running:

```bash
# Check health endpoint
curl http://localhost:3001/health
```

Expected response:
```json
{"status":"ok","timestamp":"2024-01-01T00:00:00.000Z"}
```

**Note**: Default port is **3001**, not 3000. Check your `.env` file.

## 2. Check Worker URL in Edge Function

The Edge Function uses `WORKER_URL` from secrets. Verify it's set correctly:

```bash
cd supabase
supabase secrets list
```

Should show:
```
WORKER_URL=http://localhost:3001
```

If it's wrong, update it:
```bash
supabase secrets set WORKER_URL=http://localhost:3001
```

## 3. Manual Test - Call Worker Directly

Use this curl command to test the worker manually:

```bash
curl -X POST http://localhost:3001/analyze \
  -H "Content-Type: application/json" \
  -d '{
    "jobId": "test-job-123",
    "videoPath": "test-job-123/video.mp4",
    "mode": "bag",
    "strategy": "interval-8"
  }'
```

**Important**: The video must exist in Supabase Storage at the `videoPath` first!

## 4. Full Test Flow

### Step 1: Upload a video to storage first

```bash
# Get your service role key from Supabase Dashboard
SERVICE_ROLE_KEY="your_service_role_key"
SUPABASE_URL="http://127.0.0.1:54321"  # or your production URL

# Upload a test video
curl -X POST "${SUPABASE_URL}/storage/v1/object/videos/test-job-123/video.mp4" \
  -H "Authorization: Bearer ${SERVICE_ROLE_KEY}" \
  -H "Content-Type: video/mp4" \
  --data-binary "@path/to/your/video.mp4"
```

### Step 2: Call worker with the video path

```bash
curl -X POST http://localhost:3001/analyze \
  -H "Content-Type: application/json" \
  -d '{
    "jobId": "test-job-123",
    "videoPath": "test-job-123/video.mp4",
    "mode": "bag",
    "strategy": "interval-8"
  }'
```

## 5. Debug Edge Function → Worker Call

The Edge Function logs should show:
- `[jobId] Triggering worker at http://localhost:3001/analyze`
- `[jobId] Worker request sent successfully` (if successful)
- `[jobId] Failed to trigger worker: ...` (if failed)

Check Edge Function logs:
```bash
# Local
supabase functions logs analyze --follow

# Production
supabase functions logs analyze --limit 50
```

## 6. Common Issues

### Worker not receiving calls

1. **Wrong port**: Check worker is on 3001 (or whatever WORKER_URL says)
2. **Worker not running**: Start it with `npm run dev` in `boxing-ai-worker`
3. **Network issue**: Edge Function (in Supabase) can't reach `localhost:3001`
   - **Solution**: Use ngrok or deploy worker publicly
   - Or run Edge Function locally: `supabase functions serve analyze --env-file .env.local`

### "Connection refused" error

The Edge Function (running in Supabase cloud or local Supabase) can't reach `localhost:3001` because:
- `localhost` refers to the Edge Function's environment, not your machine
- **For local testing**: Run Edge Function locally too
- **For production**: Deploy worker to a public URL or use ngrok

### Use ngrok for local testing

```bash
# Install ngrok: https://ngrok.com/download
ngrok http 3001

# Use the ngrok URL in WORKER_URL
supabase secrets set WORKER_URL=https://your-ngrok-url.ngrok.io
```

## 7. Test Script

Create `test-worker.sh`:

```bash
#!/bin/bash

JOB_ID="test-$(date +%s)"
VIDEO_PATH="${JOB_ID}/video.mp4"
WORKER_URL="http://localhost:3001"

echo "Testing worker at ${WORKER_URL}"
echo "Job ID: ${JOB_ID}"

# Test health
echo -e "\n1. Health check:"
curl -s "${WORKER_URL}/health" | jq .

# Test analyze endpoint
echo -e "\n2. Analyze endpoint:"
curl -X POST "${WORKER_URL}/analyze" \
  -H "Content-Type: application/json" \
  -d "{
    \"jobId\": \"${JOB_ID}\",
    \"videoPath\": \"${VIDEO_PATH}\",
    \"mode\": \"bag\",
    \"strategy\": \"interval-8\"
  }" | jq .
```

Run: `chmod +x test-worker.sh && ./test-worker.sh`
