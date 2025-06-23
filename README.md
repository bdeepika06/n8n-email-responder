# AI Email Auto Responder with n8n

## Project Overview

This project is an **automated AI email responder** built using **n8n (workflow automation)** and **OpenAI GPT-4o**. It reads incoming interview-related emails, determines if it's a weekend or weekday, and replies automatically with a professional, customized response.

---

## Features

* Monitors Gmail inbox using Gmail Trigger
* Filters only valid interview emails
* Detects weekends using system time
* Generates human-like email replies using OpenAI GPT-4o
* Sends auto-generated replies via Gmail
* Fully automated 24/7 email assistant

---

## Workflow Architecture

### 1. Gmail Trigger Node

* Continuously polls Gmail every 1 minute.
* Captures new incoming emails.

### 2. Filter Node

* Filters out irrelevant emails based on:

  * Not Promotions (CATEGORY\_PROMOTIONS)
  * Not Spam (SPAM)
  * Not noreply@ addresses
  * Subject must contain "interview" (case-insensitive)
  * Exclude self-sent emails

### 3. Code Node (Weekend Detection)

* Uses JavaScript to check current day of week.
* Adds `isWeekend` boolean field to the email data.

```javascript
const currentDate = new Date($now.toISO());
return {
  json: {
    ...$json,
    isWeekend: (currentDate.getDay() === 0 || currentDate.getDay() === 6)
  }
};
```

### 4. Edit Fields (Set Node)

* Extracts:

  * Clean `From` email address
  * Sender name (before `<`)
  * Email subject
* Formats data for OpenAI input.

### 5. Merge Node

* Combines output from Filter + Weekend detection into one object.
* Ensures OpenAI gets full context.

### 6. OpenAI Node (GPT-4o)

* Generates response based on dynamic system prompt:

  * If weekend: Sends away message.
  * If weekday: Sends interview scheduling message.
* Outputs email-ready HTML content.

### 7. Gmail Node (Send Reply)

* Sends AI-generated response to the original sender.
* Reuses extracted email and subject to maintain correct conversation thread.

---

## Technologies Used

* **n8n**: Open-source workflow automation tool
* **Gmail API (OAuth2)**
* **OpenAI GPT-4o API**
* **JavaScript (inside Code node)**

---

## How It Works Internally

* Every incoming email flows through multiple decision stages.
* Only valid emails are forwarded to OpenAI after enrichment.
* OpenAI replies professionally using the live context.
* Reply is sent via Gmail automatically.
* Fully serverless - no manual intervention required.

---

## Repository Structure

```
n8n-workflows/
  |-- workflows/
        |-- email-reply-bot.json  # n8n exported workflow file
  |-- README.md                   # This documentation
```

---

## Setup Instructions

1. Export your n8n workflow (already done)
2. Create `n8n-workflows` directory
3. Move your exported file inside `workflows/`
4. Initialize Git repository:

   ```bash
   git init
   git add .
   git commit -m "Initial commit: added email auto responder workflow"
   git remote add origin <your-repo-url>
   git push -u origin main
   ```
5. Done! Your automation is now version-controlled.
