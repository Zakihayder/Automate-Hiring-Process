# Multi-Role Hiring Automation System
### Trilles AI — n8n Automation Developer Assignment

**Built by:** Zaki Haider  
**Stack:** n8n (Self-Hosted Docker) | Groq AI (Llama 3.3-70B) | Google Sheets | Gmail | Google Calendar | Google Drive  
**Roles Automated:** Software Engineer (SWE) + Business Development Manager (BDM)

---

## Overview

A fully automated end-to-end hiring pipeline that handles everything from application ingestion to candidate communication — completely without manual intervention.

---

## System Architecture

### Two Independent Workflows

| Workflow | Schedule | Responsibility |
|----------|----------|----------------|
| Main Processing Pipeline | Every 15 minutes | Ingest applications, screen resumes with AI, store results |
| Email & Calendar Pipeline | Every 30 minutes | Send emails, create calendar events based on AI classification |

---

## What the System Does

### Main Processing Pipeline
1. Reads new (unprocessed) applications from both SWE and BDM Google Sheets
2. Tags each application with its role name
3. Merges both streams into one unified flow
4. Checks if applications exist — exits gracefully if none
5. Downloads resume PDF from Google Drive (via Service Account)
6. Extracts text from PDF using n8n Extract From File node
7. Sends resume to Groq AI with role-specific evaluation criteria
8. Parses structured JSON response (score, classification, summary, strengths, concerns)
9. Saves full results to Master Candidates Google Sheet
10. Marks original application as "Processed" to prevent reprocessing

### Email & Calendar Pipeline
1. Reads candidates with Email Status = "pending" from Master sheet
2. Routes by AI Classification using Switch node:
   - **Strong** → Interview invitation email + Google Calendar event created
   - **Average** → Hiring manager notification email with full AI assessment
   - **Weak** → Professional rejection email
3. Updates Email Status to "sent" after each email

---

## Google Sheets Structure

### SWE_Applications Sheet
| Column | Field |
|--------|-------|
| A | Timestamp |
| B | Full Name |
| C | Email Address |
| D | Resume (File Upload) |
| E | Years of Experience |
| F | Primary Programming Language |
| G | GitHub Profile URL |
| H | Status |

### BDM_Applications Sheet
| Column | Field |
|--------|-------|
| A | Timestamp |
| B | Full Name |
| C | Email Address |
| D | Resume (File Upload) |
| E | Years of Sales Experience |
| F | Industries Worked In |
| G | LinkedIn Profile URL |
| H | Status |

### Master_Candidates Sheet
| Column | Field |
|--------|-------|
| A | Candidate ID |
| B | Full Name |
| C | Email Address |
| D | Role |
| E | Resume URL |
| F | Resume Text |
| G | AI Score (0-100) |
| H | AI Classification (strong/average/weak) |
| I | AI Summary |
| J | AI Strengths |
| K | AI Concerns |
| L | Interview Recommended |
| M | Email Status |
| N | Calendar Event Created |
| O | Processed At |

---

## AI Evaluation Criteria

The AI evaluates candidates differently based on their role:

**Software Engineer criteria:**
- Technical skills and programming languages
- Projects and GitHub activity
- Problem-solving experience
- System design knowledge
- Years of relevant experience

**Business Development Manager criteria:**
- Sales experience and revenue targets met
- Relationship building and networking
- Industry knowledge
- Communication and presentation skills
- Track record of business growth

**Scoring:**
- 70-100 → Strong → Interview invitation
- 40-69 → Average → Manual review by hiring manager
- 0-39 → Weak → Professional rejection

---

## Setup Requirements

### Prerequisites
- Docker installed on Windows
- Google account (Gmail)
- Groq AI API key (free at console.groq.com)

### Google Cloud Setup
1. Create project: HiringAutomation
2. Enable APIs: Google Sheets, Drive, Gmail, Calendar
3. Create OAuth 2.0 credentials
4. Create Service Account for Drive access
5. Add redirect URI: `http://localhost:5678/rest/oauth2-credential/callback`

### n8n Credentials Required
- Google Sheets OAuth2
- Gmail OAuth2
- Google Drive OAuth2
- Google Calendar OAuth2
- Google Drive Service Account
- Groq AI (OpenAI node with Base URL: https://api.groq.com/openai/v1)

### Running n8n
```bash
docker run -it --rm --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
```
Access at: http://localhost:5678

---

## Error Handling

| Scenario | Handling |
|----------|----------|
| No new applications | Graceful exit via IF node check |
| AI returns invalid JSON | try/catch fallback → classified as "average" |
| Double processing | Status column filter prevents reprocessing |
| Resume download fails | Item stops, retried on next run |
| Email fails | Status stays "pending", retried in 30 min |

---

## Known Limitations

- Gmail limited to 500 emails/day (free account)
- Polling-based trigger (up to 15 min delay) vs real-time webhooks
- English-only AI prompts and email templates
- AI scoring may have inherent biases

---

## Improvements With More Time

- Replace Google Sheets with PostgreSQL or Airtable
- Webhook triggers for real-time processing
- Intelligent calendar scheduling with availability checking
- Candidate self-service status portal
- Multi-stage pipeline (screening → interviews → offers)
- Analytics dashboard in Looker Studio
- Error alerting via Slack

---

##Loom Video URL
  - https://www.loom.com/share/1624174a91034ae7a724755343a03e87

## File Structure

```
Zaki-Haider_HiringAutomation_Main.json    — Main Processing Pipeline workflow
Zaki-Haider_HiringAutomation_Email.json   — Email & Calendar Pipeline workflow
Zaki_Haider_Walkthrough.pdf            — This walkthrough document
README.md                              — This file
```

---

## Contact

**Zaki Haider**  
zakihaider7860@gmail.com  
