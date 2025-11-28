# GitHub Actions Logging Guide

This guide explains how to use the enhanced video generation workflow with integrated log forwarding.

## 📋 Overview

The enhanced workflow provides **three levels of logging**:

1. **Real-time logs in GitHub Actions** (Always enabled, free)
2. **Downloadable log artifacts** (Always enabled, free)
3. **Cloud log forwarding** (Optional, free tier available)

---

## 🚀 Quick Start

### Using the Enhanced Workflow

1. **Push the workflow to your repository:**
   ```bash
   cd /home/sathiyarajs/Documents/GitHub/Workflow
   git add .github/workflows/daily-video-generation-with-logs.yml
   git commit -m "Add enhanced workflow with logging"
   git push
   ```

2. **Run the workflow:**
   - Go to your GitHub repository
   - Click **Actions** tab
   - Select **"Video Generator - Daily Dual Content with Log Forwarding"**
   - Click **"Run workflow"**
   - Configure options and click **"Run workflow"**

---

## 📊 Viewing Logs

### Option 1: Real-Time Logs in GitHub Actions (Free)

**Best for:** Monitoring workflow progress in real-time

1. Go to **Actions** → Select your workflow run
2. Click on the **"generate-videos"** job
3. Expand any step to see logs
4. Look for lines prefixed with `[VIDEO-CREATOR]` for container output

**What you'll see:**
- Container startup messages
- n8n initialization
- Workflow activation
- Video generation progress
- Python script output
- Error messages (if any)

### Option 2: Download Log Files (Free)

**Best for:** Detailed analysis, sharing logs, debugging

1. Go to **Actions** → Select your workflow run
2. Scroll to bottom → **Artifacts** section
3. Download `video-generation-logs-<run-id>.zip`
4. Extract and open log files:
   - `container-<run-id>.log` - All container output
   - `main_video.log` - Main video generation logs
   - `netflix_video.log` - Netflix video generation logs

**Retention:** Logs are kept for 7 days (configurable in workflow)

### Option 3: Cloud Log Viewer (Optional, Free Tier)

**Best for:** Advanced querying, alerts, long-term storage

See [Setting Up Cloud Log Forwarding](#setting-up-cloud-log-forwarding) below.

---

## 🔧 Setting Up Cloud Log Forwarding

### BetterStack (Recommended)

**Free Tier:** 1GB/month, 7 days retention

1. **Sign up:**
   - Go to https://betterstack.com/logs
   - Create a free account

2. **Create a source:**
   - Click **"Add source"**
   - Select **"HTTP"** or **"Custom"**
   - Copy the **Source Token**

3. **Add to GitHub Secrets:**
   - Go to your repository → **Settings** → **Secrets and variables** → **Actions**
   - Click **"New repository secret"**
   - Name: `BETTERSTACK_SOURCE_TOKEN`
   - Value: Paste your source token
   - Click **"Add secret"**

4. **Enable in workflow:**
   - Edit [`daily-video-generation-with-logs.yml`](file:///home/sathiyarajs/Documents/GitHub/Workflow/.github/workflows/daily-video-generation-with-logs.yml)
   - Find the Vector configuration section (line ~90)
   - Uncomment the BetterStack sink:
     ```toml
     [sinks.betterstack]
     type = "http"
     inputs = ["parse_logs"]
     uri = "https://in.logs.betterstack.com"
     encoding.codec = "json"
     [sinks.betterstack.auth]
     strategy = "bearer"
     token = "${{ secrets.BETTERSTACK_SOURCE_TOKEN }}"
     ```
   - Commit and push

5. **View logs:**
   - Go to BetterStack dashboard
   - Logs will appear in real-time during workflow execution
   - Use filters: `container:"video-generator-*"`

### Logtail (Alternative)

**Free Tier:** 1GB/month, 3 days retention

Similar setup to BetterStack:
1. Sign up at https://logtail.com
2. Create a source and get token
3. Add `LOGTAIL_SOURCE_TOKEN` to GitHub secrets
4. Uncomment Logtail sink in workflow

---

## 📈 Log Levels Comparison

| Feature | GitHub Actions | Log Artifacts | Cloud (BetterStack) |
|---------|---------------|---------------|---------------------|
| **Cost** | Free | Free | Free (1GB/month) |
| **Real-time** | ✅ Yes | ❌ No | ✅ Yes |
| **Retention** | Until workflow deleted | 7 days | 7 days (free tier) |
| **Search** | Basic (Ctrl+F) | Basic (grep) | Advanced (queries) |
| **Alerts** | ❌ No | ❌ No | ✅ Yes |
| **Dashboards** | ❌ No | ❌ No | ✅ Yes |
| **Download** | ❌ No | ✅ Yes | ✅ Yes |
| **Best For** | Quick checks | Debugging | Production monitoring |

---

## 🔍 Example Log Queries

### GitHub Actions (Search in browser)

Use Ctrl+F (Cmd+F on Mac) to search for:
- `[VIDEO-CREATOR]` - All container logs
- `ERROR` - Error messages
- `✅` - Success messages
- `n8n ready` - n8n startup confirmation
- `Workflow triggered` - Webhook trigger confirmation

### Downloaded Log Files (grep)

```bash
# Extract logs
unzip video-generation-logs-*.zip

# Find errors
grep -i "error" container-*.log

# Find video generation progress
grep "video" main_video.log

# Find specific timestamps
grep "2025-11-28" container-*.log
```

### BetterStack (LogQL)

```logql
# All logs from video creator
container:"video-generator-*"

# Only errors
container:"video-generator-*" AND level:"error"

# Specific video type
video_type:"main"

# Logs from last hour
container:"video-generator-*" AND @timestamp:[now-1h TO now]
```

---

## 🐛 Troubleshooting

### Logs not appearing in GitHub Actions

**Problem:** No `[VIDEO-CREATOR]` logs in output

**Solution:**
1. Check that "Stream Container Logs" step is running
2. Verify container is actually running: look for container ID in logs
3. Check if container crashed: look for error messages in "Start Container" step

### Log artifacts not available

**Problem:** No artifacts in workflow run

**Solution:**
1. Check that workflow completed (even if failed)
2. Artifacts are only created in "Upload Logs as Artifact" step
3. Check retention period (default 7 days)

### Cloud logs not appearing

**Problem:** Logs not showing in BetterStack/Logtail

**Solution:**
1. Verify secret is correctly set: `BETTERSTACK_SOURCE_TOKEN`
2. Check Vector container is running: look for "vector-logger" in workflow
3. Verify sink is uncommented in Vector config
4. Check BetterStack source token is valid
5. Look for Vector errors in GitHub Actions logs

---

## 💡 Tips

1. **Use real-time logs for quick checks** - Fastest way to see if workflow is progressing
2. **Download artifacts for debugging** - Best for detailed analysis
3. **Use cloud logs for production** - Best for monitoring scheduled runs
4. **Set up alerts in BetterStack** - Get notified when errors occur
5. **Keep log artifacts for important runs** - Download before 7-day retention expires

---

## 🔄 Migrating from Old Workflow

If you're using the old workflow ([`daily-video-generation.yml`](file:///home/sathiyarajs/Documents/GitHub/Workflow/.github/workflows/daily-video-generation.yml)):

1. **Keep both workflows** (recommended):
   - Old workflow: Production use
   - New workflow: Testing and debugging

2. **Or replace old workflow:**
   ```bash
   cd /home/sathiyarajs/Documents/GitHub/Workflow/.github/workflows
   mv daily-video-generation.yml daily-video-generation.yml.backup
   mv daily-video-generation-with-logs.yml daily-video-generation.yml
   git add .
   git commit -m "Enable enhanced logging"
   git push
   ```

---

## 📚 Additional Resources

- [GitHub Actions Artifacts Documentation](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts)
- [BetterStack Documentation](https://betterstack.com/docs/logs/)
- [Vector Documentation](https://vector.dev/docs/)
- [Docker Logging Drivers](https://docs.docker.com/config/containers/logging/configure/)
