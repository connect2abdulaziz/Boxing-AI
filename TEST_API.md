# API Testing - Curl Commands

## 1. Test Edge Function (Upload Video + Mode)

### PowerShell

```powershell
# Test with a video file
$videoPath = "path\to\your\video.mp4"
$baseUrl = "https://kggmsgwbyqgwqjtttefk.supabase.co"  # or http://127.0.0.1:54321 for local
$anonKey = "your_anon_key"

$formData = @{
    video = Get-Item $videoPath
    mode = "bag"
} | ConvertTo-Json

Invoke-RestMethod -Uri "$baseUrl/functions/v1/analyze" `
    -Method POST `
    -Headers @{
        Authorization = "Bearer $anonKey"
    } `
    -Form @{
        video = Get-Item $videoPath
        mode = "bag"
    }
```

### curl.exe (Windows)

```powershell
curl.exe -X POST "https://kggmsgwbyqgwqjtttefk.supabase.co/functions/v1/analyze" `
  -H "Authorization: Bearer YOUR_ANON_KEY" `
  -F "video=@path\to\video.mp4" `
  -F "mode=bag"
```

### Bash/Linux/Mac

```bash
curl -X POST "https://kggmsgwbyqgwqjtttefk.supabase.co/functions/v1/analyze" \
  -H "Authorization: Bearer YOUR_ANON_KEY" \
  -F "video=@path/to/video.mp4" \
  -F "mode=bag"
```

**Expected Response:**
```json
{
  "jobId": "23329c8a-6c19-4085-9ca6-7c59b8958c47",
  "status": "processing",
  "message": "Video uploaded and processing started"
}
```

## 2. Check Job Status in Database

### PowerShell

```powershell
$baseUrl = "https://kggmsgwbyqgwqjtttefk.supabase.co"
$anonKey = "your_anon_key"
$jobId = "23329c8a-6c19-4085-9ca6-7c59b8958c47"

Invoke-RestMethod -Uri "$baseUrl/rest/v1/analysis_jobs?job_id=eq.$jobId&select=*" `
    -Method GET `
    -Headers @{
        apikey = $anonKey
        Authorization = "Bearer $anonKey"
    }
```

### curl.exe

```powershell
curl.exe -X GET "https://kggmsgwbyqgwqjtttefk.supabase.co/rest/v1/analysis_jobs?job_id=eq.23329c8a-6c19-4085-9ca6-7c59b8958c47&select=*" `
  -H "apikey: YOUR_ANON_KEY" `
  -H "Authorization: Bearer YOUR_ANON_KEY"
```

### Bash

```bash
curl -X GET "https://kggmsgwbyqgwqjtttefk.supabase.co/rest/v1/analysis_jobs?job_id=eq.23329c8a-6c19-4085-9ca6-7c59b8958c47&select=*" \
  -H "apikey: YOUR_ANON_KEY" \
  -H "Authorization: Bearer YOUR_ANON_KEY"
```

**Expected Response:**
```json
[
  {
    "id": "...",
    "job_id": "23329c8a-6c19-4085-9ca6-7c59b8958c47",
    "video_path": "23329c8a-6c19-4085-9ca6-7c59b8958c47/video.mp4",
    "mode": "bag",
    "strategy": "interval-8",
    "status": "completed",
    "result": {
      "analysis_failed": false,
      "technique_score": 85,
      "what_was_good": ["..."],
      "key_fixes": ["..."],
      "drills": ["..."],
      "openings": ["..."],
      "next_steps": "..."
    },
    "frames_count": 10,
    "created_at": "2024-01-01T00:00:00Z",
    "completed_at": "2024-01-01T00:05:00Z"
  }
]
```

## 3. Test Worker Directly (Manual Trigger)

Use this if you want to manually trigger the worker with an existing video in storage:

### PowerShell

```powershell
$body = @{
    jobId = "23329c8a-6c19-4085-9ca6-7c59b8958c47"
    videoPath = "23329c8a-6c19-4085-9ca6-7c59b8958c47/video.mp4"
    mode = "bag"
    strategy = "interval-8"
} | ConvertTo-Json

Invoke-RestMethod -Uri "http://localhost:3001/analyze" `
    -Method POST `
    -ContentType "application/json" `
    -Body $body
```

### curl.exe

```powershell
curl.exe -X POST "http://localhost:3001/analyze" `
  -H "Content-Type: application/json" `
  -d "{\"jobId\":\"23329c8a-6c19-4085-9ca6-7c59b8958c47\",\"videoPath\":\"23329c8a-6c19-4085-9ca6-7c59b8958c47/video.mp4\",\"mode\":\"bag\",\"strategy\":\"interval-8\"}"
```

### Bash

```bash
curl -X POST "http://localhost:3001/analyze" \
  -H "Content-Type: application/json" \
  -d '{"jobId":"23329c8a-6c19-4085-9ca6-7c59b8958c47","videoPath":"23329c8a-6c19-4085-9ca6-7c59b8958c47/video.mp4","mode":"bag","strategy":"interval-8"}'
```

## 4. Complete Test Flow

### Step 1: Upload video and create job

```powershell
# PowerShell
$response = Invoke-RestMethod -Uri "https://kggmsgwbyqgwqjtttefk.supabase.co/functions/v1/analyze" `
    -Method POST `
    -Headers @{ Authorization = "Bearer YOUR_ANON_KEY" } `
    -Form @{
        video = Get-Item "test-video.mp4"
        mode = "bag"
    }

$jobId = $response.jobId
Write-Host "Job ID: $jobId"
```

### Step 2: Poll for results

```powershell
# Poll every 2 seconds until completed
$baseUrl = "https://kggmsgwbyqgwqjtttefk.supabase.co"
$anonKey = "YOUR_ANON_KEY"
$jobId = "23329c8a-6c19-4085-9ca6-7c59b8958c47"

while ($true) {
    $job = Invoke-RestMethod -Uri "$baseUrl/rest/v1/analysis_jobs?job_id=eq.$jobId&select=*" `
        -Headers @{
            apikey = $anonKey
            Authorization = "Bearer $anonKey"
        }
    
    if ($job[0].status -eq "completed") {
        Write-Host "Completed!" -ForegroundColor Green
        $job[0].result | ConvertTo-Json -Depth 10
        break
    } elseif ($job[0].status -eq "failed") {
        Write-Host "Failed: $($job[0].error_message)" -ForegroundColor Red
        break
    }
    
    Write-Host "Status: $($job[0].status) - waiting..." -ForegroundColor Yellow
    Start-Sleep -Seconds 2
}
```

## 5. Quick Test Script

Save as `test-api.ps1`:

```powershell
param(
    [string]$VideoPath = "test-video.mp4",
    [string]$Mode = "bag",
    [string]$BaseUrl = "https://kggmsgwbyqgwqjtttefk.supabase.co",
    [string]$AnonKey = "YOUR_ANON_KEY"
)

Write-Host "1. Uploading video..." -ForegroundColor Cyan
$response = Invoke-RestMethod -Uri "$BaseUrl/functions/v1/analyze" `
    -Method POST `
    -Headers @{ Authorization = "Bearer $AnonKey" } `
    -Form @{
        video = Get-Item $VideoPath
        mode = $Mode
    }

$jobId = $response.jobId
Write-Host "✓ Job created: $jobId" -ForegroundColor Green

Write-Host "`n2. Polling for results..." -ForegroundColor Cyan
while ($true) {
    $job = Invoke-RestMethod -Uri "$BaseUrl/rest/v1/analysis_jobs?job_id=eq.$jobId&select=*" `
        -Headers @{
            apikey = $AnonKey
            Authorization = "Bearer $AnonKey"
        }
    
    $status = $job[0].status
    Write-Host "Status: $status" -ForegroundColor Yellow
    
    if ($status -eq "completed") {
        Write-Host "`n✓ Analysis Complete!" -ForegroundColor Green
        $job[0].result | ConvertTo-Json -Depth 10
        break
    } elseif ($status -eq "failed") {
        Write-Host "`n✗ Failed: $($job[0].error_message)" -ForegroundColor Red
        break
    }
    
    Start-Sleep -Seconds 2
}
```

Run: `.\test-api.ps1 -VideoPath "my-video.mp4" -Mode "bag"`
