# Installing FFmpeg on Windows

## Quick Install (Recommended)

### Option 1: Using Chocolatey (Easiest)

1. **Install Chocolatey** (if not installed):
   ```powershell
   Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
   ```

2. **Install FFmpeg**:
   ```powershell
   choco install ffmpeg
   ```

3. **Restart your terminal** and verify:
   ```powershell
   ffmpeg -version
   ffprobe -version
   ```

### Option 2: Manual Installation

1. **Download FFmpeg**:
   - Go to: https://www.gyan.dev/ffmpeg/builds/
   - Download: `ffmpeg-release-essentials.zip` (or latest version)

2. **Extract**:
   - Extract to: `C:\ffmpeg` (or any location you prefer)

3. **Add to PATH**:
   - Open **System Properties** → **Environment Variables**
   - Under **System Variables**, find `Path` → **Edit**
   - Click **New** → Add: `C:\ffmpeg\bin`
   - Click **OK** on all dialogs

4. **Restart terminal** and verify:
   ```powershell
   ffmpeg -version
   ffprobe -version
   ```

### Option 3: Using Scoop

```powershell
# Install Scoop (if not installed)
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
irm get.scoop.sh | iex

# Install FFmpeg
scoop install ffmpeg
```

## Verify Installation

```powershell
# Check FFmpeg
ffmpeg -version

# Check ffprobe
ffprobe -version

# Both should show version information
```

## If FFmpeg is Installed but Not Found

If FFmpeg is installed but the worker still can't find it, you can specify the path in `.env`:

```env
# In boxing-ai-worker/.env
FFMPEG_PATH=C:\ffmpeg\bin\ffmpeg.exe
FFPROBE_PATH=C:\ffmpeg\bin\ffprobe.exe
```

Or if installed via Chocolatey/Scoop:
```env
FFMPEG_PATH=C:\ProgramData\chocolatey\bin\ffmpeg.exe
FFPROBE_PATH=C:\ProgramData\chocolatey\bin\ffprobe.exe
```

## Troubleshooting

### "Cannot find ffprobe" error

1. **Check if FFmpeg is in PATH**:
   ```powershell
   where.exe ffmpeg
   where.exe ffprobe
   ```
   Should show paths like `C:\ffmpeg\bin\ffmpeg.exe`

2. **If not found**, add to PATH (see Option 2 above)

3. **Or set explicit paths** in `.env`:
   ```env
   FFMPEG_PATH=C:\path\to\ffmpeg.exe
   FFPROBE_PATH=C:\path\to\ffprobe.exe
   ```

4. **Restart the worker** after setting paths:
   ```powershell
   cd boxing-ai-worker
   npm run dev
   ```

### Test FFmpeg Works

```powershell
# Test video info
ffprobe -v error -show_format -show_streams input_video.mp4

# Test frame extraction
ffmpeg -i input_video.mp4 -ss 00:00:01 -vframes 1 output_frame.png
```

## Alternative: Use FFmpeg Binary in Project

If you can't install system-wide, you can:

1. Download FFmpeg binaries
2. Place in `boxing-ai-worker/ffmpeg/` folder
3. Set in `.env`:
   ```env
   FFMPEG_PATH=./ffmpeg/bin/ffmpeg.exe
   FFPROBE_PATH=./ffmpeg/bin/ffprobe.exe
   ```
