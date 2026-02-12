# LinkedIn Post Automation

An AI-powered system that generates viral LinkedIn posts and accompanying images from trending news topics.

## What it does

1. Takes a trending topic or news article as input
2. Guides you through framing your perspective (stance, lens, key takeaway)
3. Generates a LinkedIn post using proven viral frameworks
4. Validates the post quality using AI evaluation (LLM-as-judge)
5. Creates an AI-generated image via Google Gemini
6. Saves everything to Google Drive in an organized folder structure

## Screenshots

### n8n Workflow
![n8n Workflow](images/n8n-workflow.png)

### Generated Image Example
![Generated Image Example](images/generated-image-example.png)

### Google Drive Output Structure
![Google Drive Structure](images/google-drive-structure.png)

## Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  LLM            │────▶│   n8n Webhook   │────▶│  Google Drive   │
│  (Post Gen)     │     │  (Image Gen)    │     │  (Storage)      │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                               │
                               ▼
                        ┌─────────────────┐
                        │  Google Gemini  │
                        │  (Image API)    │
                        └─────────────────┘
```

## Components

### 1. Post Generation (Any LLM)

Generates posts using a training reference of successful LinkedIn creators:
- **L Acosta** - Hook patterns (POV, confession, contrarian)
- **S Bartlett** - Bold predictions, emotional resonance

### 2. AI Evaluation

Every post is validated against 8 criteria before saving:

| Criteria | Pass Condition |
|----------|----------------|
| Hook Pattern | Uses proven pattern (POV/confession/contrarian/results/insider) |
| First Line | Punchy, lowercase, scroll-stopping |
| Line Length | 1-2 sentences max per line |
| Voice | Your perspective, not generic summary |
| Stance | Clear supportive/critical/balanced position |
| Engagement | Ends with a question |
| Hashtags | Includes 3-5 relevant hashtags |
| Takeaway | Has one clear memorable insight |

**Scoring:** 8/8 = ready, 6-7 = auto-revise, <6 = full rewrite

### 3. Image Generation (n8n + Gemini)

An n8n workflow that:
1. Receives the post content via webhook
2. Uses Gemini to generate an image prompt from the post
3. Generates an image using Gemini 2.0 Flash
4. Uploads both post and image to Google Drive

## Example Posts

See the [examples/](examples/) folder for generated posts:

| Post | Topic | Pillar |
|------|-------|--------|
| [OpenAI Kills Safety Team](examples/openai-kills-safety-team-2025-02-12.md) | OpenAI disbanding alignment team - PM implications | Product Management |
| [Safety First Leadership](examples/safety-first-leadership-warning-signs-2025-02-12.md) | Warning signs when companies sacrifice values for speed | Leadership |
| [Prioritization Frameworks](examples/prioritization-frameworks-theater-2025-02-12.md) | Why most prioritization is theater | Product Management |

## Setup

### Prerequisites

- [n8n](https://n8n.io/) (self-hosted or cloud)
- Google Cloud project with Gemini API enabled
- Google Drive account

### Step 1: Get a Gemini API Key

1. Go to [Google AI Studio](https://aistudio.google.com/apikey)
2. Click **Create API Key**
3. Copy and save your API key

### Step 2: Import the n8n Workflow

1. Open your n8n instance
2. Click **Add workflow** → **Import from file**
3. Select `workflow.json` from this repo
4. The workflow will appear with placeholder credentials

### Step 3: Configure Gemini Credentials in n8n

1. In n8n, go to **Settings** → **Credentials** → **Add Credential**
2. Search for **HTTP Query Auth**
3. Configure:
   - **Name:** `Gemini API Key`
   - **Parameter Name:** `key`
   - **Parameter Value:** `your-gemini-api-key-here`
4. Save the credential
5. Open the workflow and update these nodes to use your credential:
   - `Gemini - Create Image Prompt`
   - `Gemini - Generate Image`

### Step 4: Configure Google Drive Credentials

1. In n8n, go to **Settings** → **Credentials** → **Add Credential**
2. Search for **Google Drive OAuth2 API**
3. Follow the OAuth flow to connect your Google account
4. Save the credential
5. Update these nodes to use your credential:
   - `Create folder`
   - `Upload file`
   - `save text to drive`

### Step 5: Set Your Google Drive Folder

1. In Google Drive, create a folder where posts will be saved (e.g., `LinkedIn Posts`)
2. Copy the folder URL (e.g., `https://drive.google.com/drive/folders/abc123...`)
3. In n8n, open the `Create folder` node
4. Update the **Parent Folder** field with your folder URL

### Step 6: Activate the Workflow

1. Toggle the workflow to **Active**
2. Copy your webhook URL (shown in the Webhook node)
3. Test with:
   ```bash
   curl -X POST "https://your-n8n-domain/webhook/linkedin-image" \
     -H "Content-Type: application/json" \
     -d '{"post_content": "Test post content", "filename": "test-post"}'
   ```

## Workflow Nodes Explained

| Node | Purpose |
|------|---------|
| **Webhook** | Receives POST request with post content |
| **Gemini - Create Image Prompt** | Generates an image description from the post |
| **Extract Image Prompt** | Parses Gemini's response |
| **Gemini - Generate Image** | Creates the actual image |
| **Format Response** | Extracts base64 image data |
| **Generate Filename** | Creates consistent filename pattern |
| **Create folder** | Makes a new folder in Google Drive |
| **Convert md to txt File** | Packages post as uploadable file |
| **Code - converts binary to image** | Converts base64 to PNG |
| **Merge1** | Synchronizes parallel file processing |
| **save text to drive** | Uploads the post file |
| **Upload file** | Uploads the image file |
| **Respond to Webhook** | Returns response to caller |

## Usage

### Generate an Image for Your Post

1. Write your LinkedIn post
2. Call the webhook with curl (see below)
3. The workflow generates an image and saves both to Google Drive

### Webhook API

**Endpoint:** `POST https://YOUR_N8N_DOMAIN/webhook/linkedin-image`

**Payload:**
```json
{
  "post_content": "your full post text here",
  "filename": "descriptive-filename"
}
```

**Response:**
```json
{
  "success": true,
  "image_base64": "...",
  "image_prompt": "A modern digital illustration...",
  "text_filename": "topic-name-01-27012026.md",
  "image_filename": "topic-name-01-27012026-image.png",
  "folder_name": "topic-name-01-27012026"
}
```

## File Structure

```
linkedin-post-automation/
├── README.md                 # This file
├── workflow.json             # n8n workflow (import this)
├── training-reference.md     # Viral post examples & patterns
├── examples/                 # Example generated posts
│   ├── openai-kills-safety-team-2025-02-12.md
│   ├── safety-first-leadership-warning-signs-2025-02-12.md
│   └── prioritization-frameworks-theater-2025-02-12.md
└── images/                   # Screenshots for README
```

### Google Drive Output

```
Your Drive Folder/
└── {topic}-{number}-{ddmmyyyy}/
    ├── {topic}-{number}-{ddmmyyyy}.md
    └── {topic}-{number}-{ddmmyyyy}-image.png
```

## Post Structure

Every generated post follows this framework:

1. **Hook** - Provocative opening line (lowercase, punchy)
2. **Context** - 2-3 sentences of setup
3. **Body** - Key insights with line breaks
4. **Takeaway** - Actionable conclusion
5. **CTA** - Engagement question
6. **Hashtags** - 3-5 relevant tags

## Tech Stack

| Component | Technology |
|-----------|------------|
| Post Generation | Any LLM |
| Image Prompt Generation | Google Gemini 2.0 Flash |
| Image Generation | Google Gemini 2.0 Flash (experimental) |
| Workflow Automation | n8n (self-hosted or cloud) |
| File Storage | Google Drive API |

## Troubleshooting

### "Model not found" error
Make sure you're using `gemini-2.0-flash-exp-image-generation` for image generation. Regular `gemini-2.0-flash` doesn't support image output.

### Images not uploading
Check that your Google Drive OAuth credential has write permissions and the folder URL is correct.

### Webhook timeout
Image generation can take 30-60 seconds. Increase your webhook timeout if needed.

## Roadmap

- [ ] Automatic LinkedIn posting via API
- [ ] Analytics tracking for post performance
- [ ] A/B testing different hooks
- [ ] Scheduled content calendar
- [ ] Multi-platform support (Twitter/X)

## License

MIT

## Author

Ray Thanni - [@raythanni](https://linkedin.com/in/raythanni)
