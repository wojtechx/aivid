
# 🎬 AIvid Video Workflow with YouTube & Twitter Integration

This document outlines the updated automation workflow for turning AIwik article summaries into videos, uploading them to YouTube and Twitter, and managing file storage using Cloudflare R2.

---

## 🔄 Workflow Steps

### 1. Trigger
- **Source**: Supabase
- **Condition**: `video_status = 'pending'`
- **Purpose**: Detect articles ready to be converted into videos.

---

### 2. Fetch Summary
- Retrieve `summary_text` and `title` from Supabase
- Identify related `slug` or `summary_file` for file naming

---

### 3. Text-to-Speech (TTS)
- Convert `summary_text` into audio using:
  - ✅ **ElevenLabs**

- **Output**: `.mp3` or `.wav` file

---

### 4. Image Generation
- Use **ChatGPT (DALL·E)** to generate **1 visual slide**
- Prompts based on title and summary context

- **Output**: `.png` or `.jpg` image

---

### 5. Video Creation
- Use **Hedra API** to combine:
  - Generated audio
  - 4 AI-generated image

- **Output**: Final `.mp4` video

---

### 6. Temporary Storage (Cloudflare R2)
- Upload video to **Cloudflare R2** (S3-compatible, cheap, fast)
- Generate temporary or signed URL
- **Purpose**: Staging before external publishing

---

### 7. YouTube Upload (Optional)
- Upload `.mp4` to YouTube using:
  - ✅ YouTube Data API
  - ✅ n8n YouTube Node (OAuth2 required)

- **Metadata**:
  - Title: from article
  - Description: from summary
  - Visibility: public/unlisted

---

### 8. Twitter Post (Optional)
- Use Twitter API to:
  - Upload video
  - Tweet a caption with `title`, `summary`, and `YouTube link`

- **Authentication**: Twitter v2 API with OAuth2 or Bearer Token

---

### 9. Supabase Update
- Set:
  - `video_status = 'done'`
  - `video_url = R2 public URL`
  - `youtube_url = link to uploaded video`
  - `twitter_post_url = tweet link (if applicable)`
- Optional: delete video from R2 after publishing

---

## 🛠️ Additional Supabase Columns (SQL)

```sql
ALTER TABLE articles
ADD COLUMN video_status TEXT DEFAULT 'pending',
ADD COLUMN video_url TEXT,
ADD COLUMN youtube_url TEXT,
ADD COLUMN twitter_post_url TEXT,
ADD COLUMN image_urls TEXT[],
ADD COLUMN audio_url TEXT,
ADD COLUMN duration_sec INTEGER;
```

---

## ✅ Notes

- **Supabase Storage** is permanent by default but not cost-effective for large/streamed video
- **Cloudflare R2** is cheap, S3-compatible, and ideal for temporary staging
- R2 is recommended due to:
  - Free 10 GB storage tier
  - No egress charges
  - Public or signed URLs
- After uploading to YouTube/Twitter, video files can be deleted from R2 to save space
- All tools can be connected in n8n using HTTP Request or Code nodes

---
