# 🏢 Cold Caller Workflow

This document outlines the architecture and execution steps for a semi-automated, agentic workflow designed to streamline the process of reaching out to recruiters on LinkedIn. By leveraging webhooks, web scraping APIs, and LLMs, the system instantly cross-references job descriptions with my CV to generate highly personalised, human-sounding connection messages. 

Crucially, this system adopts a "human-in-the-loop" approach. This route was taken, instead of autonomously interacting with the LinkedIn platform, to avoid risks of account penalties and permanent bans. The workflow drafts and routes the outputted message to Telegram for final manual review and sending.

---

## Workflow Architecture & Step-by-Step Execution

### 1. The Trigger: Job Posting Input via Telegram
* **Action:** The user discovers a relevant job posting (e.g., Data Analyst, BI Analyst) and sends the job URL as a standard chat message to a custom Telegram bot.
* **Mechanism:** Telegram instantly fires a webhook payload containing the text (URL) and the user's unique `chat_id` to the orchestrator platform (Pipedream).

### 2. Data Extraction: Web Scraping via Pipedream
* **Action:** Pipedream receives the payload and initiates a Python code ("Scraper") step to extract the readable job description.
* **Mechanism:** To bypass aggressive applicant tracking system (ATS) and job board anti-bot protections, the URL is appended to a URL-to-Markdown service (e.g., `r.jina.ai`). 
* **Output:** The service returns a clean, structured Markdown string of the job description, stripping away unnecessary navigation bars, footers, and raw HTML noise.

### 3. AI Analysis & Drafting: Gemini API
* **Action:** The cleaned job description and the full, hardcoded CV text are passed to an LLM (Gemini 1.5 Flash).
* **Mechanism:** A highly engineered system prompt instructs the model to act as a matching engine. It scans the job requirements, evaluates the CV, and selects the *single most relevant* overlapping skill or achievement (such as highlighting a Power BI demand model that secured 170 hires when applying for a data role). 
* **Constraints:** Strict negative constraints ensure the drafted message is under 75 words, entirely avoids robotic AI buzzwords (e.g., "synergy," "delve," "dynamic"), and maintains a casual, confident tone appropriate for an informal professional chat.
* **Output:** A drafted, custom-tailored LinkedIn outreach message.

### 4. The Delivery: Output via Telegram
* **Action:** The drafted message is returned directly to the user in their original chat thread.
* **Mechanism:** A final Python step in Pipedream uses the Telegram API to send a POST request, utilizing the original `chat_id` to route the message securely back to the sender.
* **Final Step:** The user receives a push notification, copies the pre-written, highly tailored message, clicks the job link, and manually sends the connection request on LinkedIn.
