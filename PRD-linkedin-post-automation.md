# Product Requirements Document (PRD)
## LinkedIn Post Automation System

---

### Document Info
| Field | Value |
|-------|-------|
| Author | Ray Thanni |
| Created | 17 December 2025 |
| Status | In Development |
| Version | 1.0 |

---

## 1. Overview

### 1.1 Problem Statement
Creating consistent, engaging LinkedIn content requires significant time and effort. The process of writing posts, generating accompanying visuals, and organizing content is manual and fragmented.

### 1.2 Solution
An automated system that generates viral LinkedIn posts based on trending topics, creates AI-generated accompanying images, and stores both in an organized Google Drive folder structure.

### 1.3 Goals
- Reduce time spent on LinkedIn content creation
- Ensure consistent post quality using proven viral frameworks
- Automatically generate relevant visual content
- Maintain organized content archive in Google Drive

---

## 2. User Persona

**Primary User:** Ray Thanni
- Professional sharing insights on AI, technology, and future trends
- Needs to maintain consistent LinkedIn presence
- Values efficiency and automation
- Prefers organized file management

---

## 3. System Components

### 3.1 Claude Code Slash Command (`/virallinkedin`)
**Purpose:** Generate viral LinkedIn posts through guided conversation

**Workflow:**
1. Fetch trending headlines from:
   - International Intrigue
   - ts2.tech (tech/AI news)
2. Present topic options to user
3. Gather qualifying information:
   - Stance (supportive/critical/balanced)
   - Lens (prediction/observation/personal story)
   - Key takeaway
   - Perspective (parent/professional/citizen)
4. Validate idea viability
5. Generate post using training reference examples
6. **AI Evaluation step** - Validate post against Section 7 checklist (see below)
7. Present draft to user for approval
8. Save post locally as markdown
9. Trigger n8n webhook for image generation

### 3.1.1 AI Evaluation (LLM-as-Judge)

Before presenting the draft, evaluate the post against these criteria:

| Criteria | Pass Condition |
|----------|----------------|
| Hook Pattern | Uses proven pattern (POV/confession/contrarian/results/insider) |
| First Line | Punchy, lowercase, scroll-stopping |
| Line Length | 1-2 sentences max per line |
| Voice | Ray's PM perspective, not generic summary |
| Stance | Clear supportive/critical/balanced position |
| Engagement | Ends with a question |
| Hashtags | Includes 3-5 relevant hashtags |
| Takeaway | Has one clear memorable insight |

**Scoring:**
- 8/8 criteria = Ready to present
- 6-7/8 = Auto-revise weak areas, then present
- <6/8 = Full rewrite required

**This step is MANDATORY. Never skip evaluation.**

**Input:** User selections and preferences
**Output:** Markdown file with post content + metadata

### 3.2 n8n Webhook Workflow
**Purpose:** Generate AI images and store content in Google Drive

**Trigger:** POST request to webhook endpoint
- Test URL: `https://YOUR_N8N_DOMAIN/webhook-test/linkedin-image`
- Production URL: `https://YOUR_N8N_DOMAIN/webhook/linkedin-image`

**Payload:**
```json
{
  "post_content": "Full text of the LinkedIn post",
  "filename": "base-filename"
}
```

**Workflow Nodes:**

| Node | Type | Purpose |
|------|------|---------|
| Webhook | Trigger | Receives POST request with post content |
| Gemini - Create Image Prompt | HTTP Request | Generates image description from post |
| Extract Image Prompt | Code | Parses Gemini response |
| Build image request | Code | Constructs image generation payload |
| Gemini - Generate Image | HTTP Request | Creates AI image via Gemini 2.0 Flash |
| Format Response | Code | Extracts base64 image data |
| Generate Filename | Code | Creates standardized filenames |
| Create Folder | Google Drive | Creates subfolder for post |
| Save Text to Drive | Google Drive | Uploads markdown post |
| Convert to File | Convert | Converts base64 to binary |
| Upload Image to Drive | Google Drive | Uploads PNG image |
| Respond to Webhook | Response | Returns confirmation |

---

## 4. File & Folder Structure

### 4.1 Google Drive Structure
```
My Drive/
└── 06 AI/
    └── Linked Posts PM AI/
        ├── {title}-{number}-{ddmmyyyy}/
        │   ├── {title}-{number}-{ddmmyyyy}.md
        │   └── {title}-{number}-{ddmmyyyy}-image.png
        └── ...
```

### 4.2 Naming Convention
- **Folder & Files:** `{5-word-max-title}-{article-number}-{ddmmyyyy}`
- **Example:** `social-media-ban-kids-01-17122025`
- **Text file:** `social-media-ban-kids-01-17122025.md`
- **Image file:** `social-media-ban-kids-01-17122025-image.png`

### 4.3 Local Storage
```
/Users/rayt/cursor/testfolderinside rayt/linkedinPostAutomation/
├── training-reference.md       (style examples)
├── {topic}-{date}.md           (generated posts)
└── PRD-linkedin-post-automation.md
```

---

## 5. Technical Specifications

### 5.1 APIs & Services

| Service | Purpose | Model/Endpoint |
|---------|---------|----------------|
| Google Gemini | Image prompt generation | gemini-2.0-flash |
| Google Gemini | Image generation | gemini-2.0-flash-exp |
| Google Drive API | File storage | OAuth2 |
| n8n | Workflow automation | Self-hosted |

### 5.2 API Configuration

**Gemini - Create Image Prompt:**
- Method: POST
- URL: `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent`
- Auth: Query parameter (API key)

**Gemini - Generate Image:**
- Method: POST
- URL: `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash-exp:generateContent`
- Body includes: `responseModalities: ["text", "image"]`

### 5.3 Authentication
- **Gemini API:** API key via query parameter
- **Google Drive:** OAuth2 credentials stored in n8n

---

## 6. Post Generation Framework

### 6.1 Training Reference Sources
- Lara Acosta (engagement hooks)
- Lenny Rachitsky (insight-driven)
- Steven Bartlett (bold predictions)

### 6.2 Post Structure
1. **Hook:** Provocative opening line (lowercase, punchy)
2. **Context:** 2-3 sentences of setup
3. **Body:** Key insights with line breaks
4. **Takeaway:** Actionable conclusion
5. **CTA:** Engagement prompt (optional)

### 6.3 Content Pillars
- Predicting life in the future
- AI & technology trends
- Work & career evolution
- Parenting in digital age
- Geopolitics & society

---

## 7. Success Metrics

| Metric | Target |
|--------|--------|
| Post generation time | < 5 minutes |
| Image generation success rate | > 95% |
| File organization accuracy | 100% |
| Workflow reliability | > 99% |

---

## 8. Current Status

### 8.1 Completed
- [x] Claude Code slash command (`/virallinkedin`)
- [x] Training reference document
- [x] n8n webhook trigger
- [x] Gemini image prompt generation
- [x] Gemini image generation
- [x] Base64 image extraction
- [x] Filename generation logic
- [x] Google Drive OAuth connection

### 8.2 In Progress
- [ ] Create subfolder in Google Drive
- [ ] Save text file to Google Drive
- [ ] Convert base64 to binary file
- [ ] Upload image to Google Drive
- [ ] End-to-end testing

### 8.3 Future Enhancements
- Automatic LinkedIn posting via API
- Analytics tracking for post performance
- A/B testing different hooks
- Scheduled content calendar
- Multi-platform support (Twitter/X)

---

## 9. Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| Gemini API quota limits | Billing-enabled Google Cloud project |
| Image generation failures | Retry logic + fallback prompts |
| Google Drive API changes | Version-locked API endpoints |
| n8n downtime | Self-hosted with monitoring |

---

## 10. Appendix

### 10.1 Sample Webhook Request
```bash
curl -X POST "https://YOUR_N8N_DOMAIN/webhook/linkedin-image" \
  -H "Content-Type: application/json" \
  -d '{
    "post_content": "your country will ban social media for kids within 2 years.\n\nhere is why i am not mad about it.",
    "filename": "social-media-ban-kids"
  }'
```

### 10.2 Sample Response
```json
{
  "success": true,
  "image_base64": "iVBORw0KGgoAAAA...",
  "image_prompt": "A modern digital illustration...",
  "text_filename": "social-media-ban-kids-01-17122025",
  "image_filename": "social-media-ban-kids-01-17122025-image",
  "folder_name": "social-media-ban-kids-01-17122025"
}
```
