# WorkWise: AI Instagram News Automation

**AI-powered Instagram post automation for [@_workwise__](https://www.instagram.com/_workwise__/)**

WorkWise is an end-to-end [n8n](https://n8n.io) automation that finds AI and tech news, writes the Instagram post, designs the graphic, and publishes it twice a day (9 AM and 2 PM, Asia/Karachi). A human approves every post by email before it goes live.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Tech Stack](#tech-stack)
- [System Architecture](#system-architecture)
- [Key Features](#key-features)
- [Engineering Challenges and Solutions](#engineering-challenges-and-solutions)
- [Business Value](#business-value)
- [Snapshots](#snapshots)

---

## Project Overview

**Problem.** Running a news page means checking sources every day, writing captions, designing a new graphic each time, and avoiding repeated topics. Doing this by hand is slow and hard to keep consistent.

**Solution.** The workflow pulls stories from five RSS feeds, filters out old and already-published stories, and lets Google Gemini pick the single most useful one for students, freelancers, small business owners and the general public. It then generates the caption, hashtags and image text, renders a 1080×1350 graphic from one of nine layouts, and sends it for approval. On approval it posts through the Instagram Graph API and logs the post to Google Sheets.

## Tech Stack

The project combines workflow automation, a large language model, two image services and the Instagram API.

| Layer          | Tool                                                                              | Role in the project                                      |
|----------------|-----------------------------------------------------------------------------------|----------------------------------------------------------|
| Orchestration  | n8n                                                                               | Scheduling, branching, loops, error workflow             |
| AI             | Google Gemini (Flash-Lite) via n8n LangChain chain                                | Story selection, caption and post copy                   |
| Output control | Structured Output Parser (JSON schema)                                            | Forces valid, predictable AI output                      |
| News sources   | RSS: TechCrunch AI, The Verge, MIT Technology Review, OpenAI Blog, Google AI Blog | Story ingestion                                          |
| Design         | JavaScript (n8n Code node), HTML/CSS                                              | Nine post layouts, rendered by the HTML/CSS to Image API |
| Images         | Unsplash API, AI image generation, article cover images                           | Photo for each post                                      |
| Approval       | Gmail (send-and-wait custom form)                                                 | Human review with three actions                          |
| Publishing     | Instagram Graph API                                                               | Create media container, then publish                     |
| Logging        | Google Sheets                                                                     | Published-post history and duplicate check               |

## System Architecture

![Workflow architecture: 2 decisions, 2 loops](images/architecture.png)

The pipeline runs left to right, then back along the second row. **Redo design** re-renders the same story, **another topic** returns to Gemini with the rejected story excluded, and an unsuitable story stops the run.

## Key Features

1. **Multi-source news pipeline.** Five RSS feeds are merged, limited to the last 24 hours, de-duplicated by link, checked against the Google Sheets history, sorted newest first and trimmed to the 15 best candidates.

2. **AI editorial engine.** A detailed prompt makes Gemini act as editor and fact-checker. It selects one story and writes the headline (max 10 words), 3–7 points, caption, call to action, hashtags, category, image keywords, takeaway and an optional before/after comparison. A `suitable` flag stops the run when no story meets the standard, so weak content is never forced out.

3. **Fact-safety rules.** The prompt forbids invented facts, numbers or quotes, requires distinguishing reported facts from implications, and avoids income or growth promises.

4. **Topic-level duplicate protection.** Besides filtering by link, the AI receives the last 30 published titles and must avoid the same event even when another publication covers it.

5. **Dynamic design engine.** Nine layouts (gradient, card, spotlight, list, magazine, takeaway, breaking news, before/after, bold text) combine with four image modes (Unsplash photo, AI-generated image, article cover image, text-only). Accent colours are random, font sizes adapt to headline length, and image sources fall back automatically if one fails.

6. **Human-in-the-loop approval.** An email shows the headline, caption and image, and offers three actions: **Publish**, **Choose another topic**, **Reject and redo design**.

7. **Feedback loops.** "Redo design" re-renders the same story with a layout not yet used, up to five times. "Choose another topic" removes the rejected story, adds its title to the AI's avoid-list and re-runs selection.

8. **Publishing and logging.** The Instagram API call runs in two steps (create container, wait, publish), and each post is logged with link, title, time slot and Instagram post ID.

## Engineering Challenges and Solutions

| Challenge | Solution |
|-----------|----------|
| Different sites cover the same story, so link-based filtering misses repeats | Two layers: exact link match against the Sheets log, plus an AI check against the last 30 published titles |
| AI could invent facts or return unusable text | Strict prompt rules, a JSON schema parser, a `suitable` stop condition and a human approval step |
| Posts all looked the same | Weighted random choice of layout, image mode and accent colour, with no layout repeated during redo attempts |
| External feeds and image services can fail | Feed nodes continue on error, image sources fall back in order (chosen source, Unsplash, AI image), and a workflow-level error handler is attached |
| Rejecting a topic must not bring the same story back | A code node rebuilds the story list each loop, excluding every rejected link, while the AI prompt also receives the rejected titles |
| Loops need memory of earlier attempts | Run-index based lookups of previous runs track used layouts and rejected stories inside one execution |
| Instagram needs the image to be processed before publishing | Two-step publish with a wait between creating the media container and publishing it |

## Business Value

- **Saves time:** Replaces the daily routine of reading news, picking a story, writing a caption, designing a graphic and posting. The workflow handles this work, and the owner only approves.
- **Consistent posting:** Publishes automatically at 9 AM and 2 PM every day without depending on someone remembering or being available.
- **Lower content cost:** Needs no social media manager, copywriter or designer, and runs on affordable tools (n8n, Gemini, Unsplash, Google Sheets).
- **Brand control:** The human approval step lets the owner check every post before it goes public. This protects the brand from AI mistakes and wrong or off-tone content.
- **Quality and accuracy:** The AI is instructed not to invent facts and to skip weak stories, so the page doesn't publish filler just to hit a schedule.
- **Audience growth potential:** Regular, relevant posts support follower growth and engagement. The caption ends with a discussion question, and the account can later be used for brand building or lead generation.
- **Scalable and reusable:** The same system can be adapted to other niches, clients or platforms by changing the RSS feeds, prompt and design templates. This makes it sellable as an automation service.

## Snapshots

### n8n workflow

![n8n workflow](images/n8n-workflow.png)

### Email for approval

<img src="images/approval-email.png" alt="Approval email" width="320">

### Image attached with approval

<img src="images/approval-image.png" alt="Post image attached to the approval email" width="360">

### Instagram posts

![Instagram posts](images/instagram-posts.png)

### Instagram page

[instagram.com/_workwise__](https://www.instagram.com/_workwise__/)
