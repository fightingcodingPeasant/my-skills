---
name: agnes-ai
description: >
  Generate images and videos using the Agnes AI API (agnes-image-2.0-flash and agnes-video-v2.0).
  Use this skill whenever the user asks to generate, create, draw, or make any image, picture, poster, illustration, product photo, avatar, meme, thumbnail, banner, or visual content — even casually phrased like "画一张", "生成图片", "给我个图", "make an image".
  Also use for image editing (图生图), style transfer, multi-image composition, and any visual content creation.
  Additionally, use when the user asks to generate, create, or make any video, animation, motion clip, short video, reel, TikTok-style video, or animated content — phrased as "生成视频", "做个视频", "animate this", etc.
  API key: sk-4QICt2q57VUTSgCmmynUq9SlTQMlHfQ1D8THTBDJhOV5UUiu
---

# Agnes AI — Image & Video Generation

Call the Agnes AI API to generate images and videos. The API base URL is `https://apihub.agnes-ai.com`.

## Authentication

All requests require:
```
Authorization: Bearer sk-4QICt2q57VUTSgCmmynUq9SlTQMlHfQ1D8THTBDJhOV5UUiu
Content-Type: application/json
```

## Image Generation (agnes-image-2.0-flash)

Endpoint: `POST https://apihub.agnes-ai.com/v1/images/generations`

### Text-to-Image

Send `model`, `prompt`, and `size`. Do NOT send `image`.

```json
{
  "model": "agnes-image-2.0-flash",
  "prompt": "A clean product photo of a glass cube on a white studio background, soft shadows, high detail",
  "size": "1024x768",
  "extra_body": {
    "response_format": "url"
  }
}
```

Response: `data[0].url`

### Image-to-Image (图生图)

Send `model`, `prompt`, `size`, and `image` (array of URLs or Data URIs). The `image` goes inside `extra_body`. Do NOT send `tags`.

```json
{
  "model": "agnes-image-2.0-flash",
  "prompt": "Transform this image into a cinematic cyberpunk style while preserving the main subject and composition",
  "size": "1024x768",
  "extra_body": {
    "image": ["https://example.com/input-image.png"],
    "response_format": "url"
  }
}
```

### Multi-Image Composition

Pass multiple URLs in `extra_body.image`:

```json
{
  "model": "agnes-image-2.0-flash",
  "prompt": "Combine the two characters into an intense fantasy battle scene, dynamic lighting, detailed background, cinematic composition",
  "size": "1024x768",
  "extra_body": {
    "image": [
      "https://example.com/character-1.png",
      "https://example.com/character-2.png"
    ],
    "response_format": "url"
  }
}
```

### Response Formats

- **URL output** (recommended): `extra_body.response_format: "url"` → response has `data[0].url`
- **Base64 output**: `return_base64: true` → response has `data[0].b64_json`

### ⚠️ Important Gotchas

1. **`response_format` goes inside `extra_body`**, NOT at the top level. Putting it at top level causes 400 errors.
2. **Text-to-image does NOT need `image`** parameter.
3. **Image-to-image does NOT need `tags`** parameter.
4. Input image URLs must be publicly accessible HTTPS URLs, or use Data URI format: `data:image/png;base64,...`
5. Set client timeout to 60–360 seconds (image generation takes several seconds).

### Supported Sizes

`1024x768`, `1024x1024`, `768x1024`

### Prompt Best Practices

**Text-to-Image structure:**
`[Main subject] + [Scene/background] + [Style] + [Lighting] + [Composition] + [Quality requirements]`

Example: `A young explorer standing in an ancient temple, cinematic fantasy style, warm dramatic lighting, wide-angle composition, ultra detailed, high quality`

**Image-to-Image structure:**
`[Editing instruction] + [Elements to preserve] + [Target style/scene] + [Lighting] + [Composition] + [Quality requirements]`

Example: `Change the background into a cinematic fantasy temple while preserving the person's face, outfit, and pose, warm dramatic lighting, wide-angle composition, ultra detailed, high quality`

## Video Generation (agnes-video-v2.0)

Video generation is **asynchronous**: create a task first, then poll for results.

### Step 1: Create Video Task

Endpoint: `POST https://apihub.agnes-ai.com/v1/videos`

**Text-to-Video:**
```json
{
  "model": "agnes-video-v2.0",
  "prompt": "A cinematic shot of a cat walking on the beach at sunset, soft ocean waves, warm golden lighting, realistic motion",
  "height": 768,
  "width": 1152,
  "num_frames": 121,
  "frame_rate": 24
}
```

**Image-to-Video:**
```json
{
  "model": "agnes-video-v2.0",
  "prompt": "The woman slowly turns around and looks back at the camera, natural facial expression, cinematic camera movement",
  "image": "https://example.com/image.png",
  "num_frames": 121,
  "frame_rate": 24
}
```

**Multi-Image Video:**
```json
{
  "model": "agnes-video-v2.0",
  "prompt": "Create a smooth transformation scene between the two reference images, cinematic lighting, consistent character identity, natural motion",
  "extra_body": {
    "image": ["https://example.com/image1.png", "https://example.com/image2.png"]
  },
  "num_frames": 121,
  "frame_rate": 24
}
```

**Keyframe Animation:**
```json
{
  "model": "agnes-video-v2.0",
  "prompt": "Generate a smooth cinematic transition between the keyframes, maintaining visual consistency and natural camera movement",
  "extra_body": {
    "image": ["https://example.com/keyframe1.png", "https://example.com/keyframe2.png"],
    "mode": "keyframes"
  },
  "num_frames": 121,
  "frame_rate": 24
}
```

Response includes `video_id` (recommended for polling) and `task_id`.

### Step 2: Poll for Results

**Recommended** — use `video_id`:
```
GET https://apihub.agnes-ai.com/agnesapi?video_id=<VIDEO_ID>
Authorization: Bearer sk-4QICt2q57VUTSgCmmynUq9SlTQMlHfQ1D8THTBDJhOV5UUiu
```

**Legacy** — use `task_id`:
```
GET https://apihub.agnes-ai.com/v1/videos/<TASK_ID>
Authorization: Bearer sk-4QICt2q57VUTSgCmmynUq9SlTQMlHfQ1D8THTBDJhOV5UUiu
```

Poll every 5 seconds. When `status` is `"completed"`, the video URL is in `remixed_from_video_id`.

### Status Values

- `queued` — waiting in queue
- `in_progress` — generating
- `completed` — done, video URL available
- `failed` — error

### Video Parameter Rules

- `num_frames` must be ≤ 441 and satisfy `8n + 1` (valid: 81, 121, 161, 241, 441)
- `frame_rate` range: 1–60
- Duration formula: `seconds = num_frames / frame_rate`

### Recommended Parameters

| Use Case | width | height | num_frames | frame_rate |
|---|---|---|---|---|
| Standard | 1152 | 768 | 121 | 24 |
| Social short | — | — | 81 or 121 | 24 |
| Reproducible | — | — | — | set fixed seed |

### Prompt Best Practices

**Text-to-Video structure:**
`[Subject] + [Action] + [Scene] + [Camera movement] + [Lighting] + [Style]`

Example: `A young astronaut walking across a red desert planet, dust blowing in the wind, slow cinematic tracking shot, dramatic sunset lighting, realistic sci-fi style`

**Image-to-Video structure:**
Describe what should move and what should stay stable.

Example: `Animate the character with subtle breathing motion, hair moving gently in the wind, background lights flickering softly, while keeping the face and outfit consistent`

## Implementation

Use `curl` or `requests` (Python) to call the API. Always handle:
- Timeout (60–360s for images, longer for video)
- Error responses (check `code` and `message` in JSON)
- Video polling loop with 5s intervals
