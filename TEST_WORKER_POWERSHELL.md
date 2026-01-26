# Testing Worker - PowerShell Commands

## 1. Test Health Endpoint

```powershell
Invoke-WebRequest -Uri "http://localhost:3001/health" -Method GET | Select-Object -ExpandProperty Content
```

Or simpler:
```powershell
curl.exe http://localhost:3001/health
```

## 2. Test Analyze Endpoint

### Option A: Using Invoke-WebRequest (PowerShell native)

```powershell
$body = @{
    jobId = "test-manual-123"
    videoPath = "7da05fe9-fe37-4f64-a0e0-0fe9c57455a7/video.mp4"
    mode = "bag"
    strategy = "interval-8"
} | ConvertTo-Json

Invoke-WebRequest -Uri "http://localhost:3001/analyze" `
    -Method POST `
    -ContentType "application/json" `
    -Body $body
```

### Option B: Using curl.exe (Windows curl)

```powershell
curl.exe -X POST http://localhost:3001/analyze `
    -H "Content-Type: application/json" `
    -d "{\"jobId\":\"test-manual-123\",\"videoPath\":\"7da05fe9-fe37-4f64-a0e0-0fe9c57455a7/video.mp4\",\"mode\":\"bag\",\"strategy\":\"interval-8\"}"
```

### Option C: One-liner (easiest)

```powershell
$json = '{"jobId":"test-manual-123","videoPath":"7da05fe9-fe37-4f64-a0e0-0fe9c57455a7/video.mp4","mode":"bag","strategy":"interval-8"}'; Invoke-RestMethod -Uri "http://localhost:3001/analyze" -Method POST -ContentType "application/json" -Body $json
```

## 3. Full Test Script

Save as `test-worker.ps1`:

```powershell
# Test Worker Script
$WORKER_URL = "http://localhost:3001"
$JOB_ID = "test-$(Get-Date -Format 'yyyyMMddHHmmss')"
$VIDEO_PATH = "7da05fe9-fe37-4f64-a0e0-0fe9c57455a7/video.mp4"

Write-Host "Testing worker at $WORKER_URL" -ForegroundColor Cyan
Write-Host "Job ID: $JOB_ID" -ForegroundColor Yellow

# 1. Health check
Write-Host "`n1. Health check:" -ForegroundColor Green
try {
    $health = Invoke-RestMethod -Uri "$WORKER_URL/health" -Method GET
    Write-Host "✓ Worker is running" -ForegroundColor Green
    $health | ConvertTo-Json
} catch {
    Write-Host "✗ Worker is not running: $_" -ForegroundColor Red
    exit 1
}

# 2. Test analyze endpoint
Write-Host "`n2. Analyze endpoint:" -ForegroundColor Green
$payload = @{
    jobId = $JOB_ID
    videoPath = $VIDEO_PATH
    mode = "bag"
    strategy = "interval-8"
} | ConvertTo-Json

try {
    $response = Invoke-RestMethod -Uri "$WORKER_URL/analyze" `
        -Method POST `
        -ContentType "application/json" `
        -Body $payload
    Write-Host "✓ Request sent successfully" -ForegroundColor Green
    $response | ConvertTo-Json -Depth 10
} catch {
    Write-Host "✗ Error: $_" -ForegroundColor Red
    if ($_.Exception.Response) {
        $reader = New-Object System.IO.StreamReader($_.Exception.Response.GetResponseStream())
        $responseBody = $reader.ReadToEnd()
        Write-Host "Response: $responseBody" -ForegroundColor Yellow
    }
}
```

Run: `.\test-worker.ps1`

## 4. Quick Commands Reference

### Health Check
```powershell
Invoke-RestMethod http://localhost:3001/health
```

### Analyze (with your actual job ID)
```powershell
$body = '{"jobId":"7da05fe9-fe37-4f64-a0e0-0fe9c57455a7","videoPath":"7da05fe9-fe37-4f64-a0e0-0fe9c57455a7/video.mp4","mode":"bag","strategy":"interval-8"}'
Invoke-RestMethod -Uri "http://localhost:3001/analyze" -Method POST -ContentType "application/json" -Body $body
```

## 5. Check Worker Logs

Make sure your worker is running and check the console output:

```bash
cd boxing-ai-worker
npm run dev
```

You should see:
- `🚀 Boxing AI Worker running on port 3001`
- Request logs when you call the endpoint

## 6. Common Issues

### "Connection refused"
- Worker not running: Start it with `npm run dev` in `boxing-ai-worker`
- Wrong port: Check `PORT` in `.env` (default is 3001)

### "Cannot GET /analyze"
- Wrong method: Use POST, not GET
- Wrong endpoint: Should be `/analyze`, not `/health`

### "Missing required fields"
- Check JSON format is correct
- All fields required: `jobId`, `videoPath`, `mode`, `strategy`
