# 🎬 Google Drive → YouTube Auto-Uploader

Automatically downloads a video from a **public Google Drive link** and uploads it to your **YouTube channel** — runs entirely inside **GitHub Actions**.

Default upload status is **`private`**.  
Default title is the **original filename from Google Drive**.

---

## 📁 Project Structure

```
.
├── .github/
│   └── workflows/
│       └── upload.yml          ← GitHub Actions workflow
├── scripts/
│   ├── upload.py               ← Main download + upload script
│   └── generate_token.py       ← One-time local token setup
├── requirements.txt
└── README.md
```

---

## ⚙️ One-Time Setup

### 1. Enable YouTube Data API

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a project → **Enable YouTube Data API v3**
3. **OAuth Consent Screen** → External → add your Google account as a **Test User**
4. **Credentials** → Create → **OAuth 2.0 Client ID** → Desktop app
5. Download the JSON → rename it `client_secret.json`

---

### 2. Generate Your YouTube Token (run locally once)

```bash
pip install -r requirements.txt
python scripts/generate_token.py --secrets client_secret.json
```

- A browser window opens → **sign in with your YouTube channel account**
- The script prints a **base64-encoded token string**

---

### 3. Add GitHub Secrets

Go to your repo → **Settings → Secrets → Actions → New repository secret**

| Secret name | Value |
|---|---|
| `YOUTUBE_TOKEN_JSON` | The base64 string from step 2 (**required**) |
| `GOOGLE_API_KEY` | A Google API key *(optional, helps fetch Drive file metadata)* |

---

## 🚀 Usage

### Option A — Manual (GitHub UI)

1. Go to **Actions → Upload Video to YouTube → Run workflow**
2. Fill in the form:

| Field | Description | Default |
|---|---|---|
| `drive_url` | Public Google Drive share link | *(required)* |
| `title` | Override video title | Drive filename |
| `description` | Video description | *(empty)* |
| `tags` | Comma-separated tags | *(empty)* |
| `category_id` | [YouTube category ID](https://developers.google.com/youtube/v3/docs/videoCategories/list) | `22` (People & Blogs) |
| `privacy` | `private` / `unlisted` / `public` | `private` |

---

### Option B — API Call (repository_dispatch)

Trigger the workflow via the GitHub REST API:

```bash
curl -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer YOUR_GITHUB_PAT" \
  https://api.github.com/repos/{owner}/{repo}/dispatches \
  -d '{
    "event_type": "upload_video",
    "client_payload": {
      "drive_url": "https://drive.google.com/file/d/FILE_ID/view?usp=sharing",
      "title": "My Awesome Video",
      "description": "Uploaded automatically via GitHub Actions",
      "tags": "tutorial,automation",
      "privacy": "private"
    }
  }'
```

> **`YOUR_GITHUB_PAT`** needs `repo` scope.  
> Generate one at: Settings → Developer settings → Personal access tokens → Fine-grained tokens

---

### Option C — Another Workflow (workflow_call)

Call from another workflow in the same repo:

```yaml
jobs:
  upload:
    uses: ./.github/workflows/upload.yml
    secrets: inherit
    with:
      drive_url: "https://drive.google.com/file/d/FILE_ID/view"
      privacy: "private"
```

---

## 📤 Outputs

After a successful run, these outputs are available in downstream steps:

| Output | Example |
|---|---|
| `video_id` | `dQw4w9WgXcQ` |
| `video_title` | `My Awesome Video` |
| `video_url` | `https://www.youtube.com/watch?v=dQw4w9WgXcQ` |

The **Actions job summary** also shows a table with these values.

---

## 🔒 Notes on Google Drive Links

The Drive video must be **publicly shared** ("Anyone with the link can view"):

```
https://drive.google.com/file/d/FILE_ID/view?usp=sharing   ✅
https://drive.google.com/open?id=FILE_ID                    ✅
https://drive.google.com/uc?id=FILE_ID                      ✅
```

Private or restricted files will fail at the download step.

---

## 🔄 Token Refresh

The OAuth token auto-refreshes using the `refresh_token`. As long as your app stays in **Testing** mode in Google Cloud Console, tokens remain valid for 7 days.

To make tokens permanent:
- Submit your OAuth app for **Google verification** (for production use), **or**
- Re-run `generate_token.py` every 7 days and update the secret

---

## 📋 YouTube Category IDs (common)

| ID | Category |
|---|---|
| 1 | Film & Animation |
| 2 | Autos & Vehicles |
| 10 | Music |
| 17 | Sports |
| 20 | Gaming |
| 22 | People & Blogs *(default)* |
| 24 | Entertainment |
| 27 | Education |
| 28 | Science & Technology |
