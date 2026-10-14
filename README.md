# 🎯 HireSense AI

**AI-powered resume screening and interview scheduling, built with n8n, Google Gemini, Google Sheets, Gmail and Google Calendar.**

Upload a resume through a simple web form. HireSense AI reads it, scores it against a job description, saves the result to a spreadsheet, and automatically sends the right email (interview invite, HR review, or polite rejection) without any manual screening.

![n8n](https://img.shields.io/badge/Built%20with-n8n-EA4B71?logo=n8n&logoColor=white)
![Gemini](https://img.shields.io/badge/AI-Google%20Gemini-4285F4?logo=google&logoColor=white)
![Google Sheets](https://img.shields.io/badge/Database-Google%20Sheets-34A853?logo=googlesheets&logoColor=white)
![Gmail](https://img.shields.io/badge/Email-Gmail-EA4335?logo=gmail&logoColor=white)
![Status](https://img.shields.io/badge/Status-Active-brightgreen)

---

## 📌 Table of Contents

- [Problem Statement](#-problem-statement)
- [Features](#-features)
- [Workflow Overview](#-workflow-overview)
- [Screenshots](#-screenshots)
- [Tech Stack](#-tech-stack)
- [How It Works](#-how-it-works)
- [Setup Guide](#-setup-guide)
- [Google Sheet Structure](#-google-sheet-structure)
- [Email Templates](#-email-templates)
- [Project Structure](#-project-structure)
- [Roadmap](#-roadmap)
- [Troubleshooting](#-troubleshooting)
- [Author](#-author)

---

## 🧩 Problem Statement

Recruiters and small companies receive dozens or hundreds of resumes for a single role. Reading each one manually is slow, inconsistent and tiring, and good candidates get missed or replied to late.

**HireSense AI** automates the first round of screening:

- Reads resumes and extracts the key details
- Scores every candidate against the job description using the same criteria
- Keeps a ranked record in one place
- Replies to every candidate automatically

---

## ✨ Features

- 📝 **Web upload form** with name, email, role and PDF resume. No Google login needed for candidates.
- 📄 **Automatic PDF text extraction** from uploaded resumes.
- 🎯 **Role-based screening.** Pick a role (or choose "Other" and type your own) and the AI decides which skills that role needs, then checks the resume for them.
- 🤖 **AI scoring (0 to 100)** with Google Gemini, returned as clean structured JSON (required, matched and missing skills, experience, education, strengths, gaps).
- ⚖️ **Skills-based evaluation.** The prompt tells the AI to ignore name, gender and college.
- 📊 **Candidate database** in Google Sheets with every score and summary.
- 🔀 **Smart routing** by recommendation: `Shortlist`, `Maybe` or `Reject`.
- 📅 **Auto interview scheduling** with a Google Calendar event and Google Meet link for shortlisted candidates.
- 📧 **Beautiful HTML emails** for each outcome (interview invite, HR review, rejection).
- 🆓 **Runs on free tiers** (n8n, Gemini API, Google Workspace).

---

## 🔄 Workflow Overview

```
Resume Upload Form
        │
        ▼
Extract Resume Text (PDF)
        │
        ▼
Prepare AI Input (resume + job description)
        │
        ▼
AI Resume Scorer (Gemini + Structured Output Parser)
        │
        ├──────────────► Save to Candidate Sheet (Google Sheets)
        │
        ▼
Route by Recommendation (Switch)
        │
        ├── Shortlist ──► Schedule Interview (Calendar) ──► Send Interview Invite (Gmail)
        ├── Maybe ──────► Notify HR for Review (Gmail)
        └── Reject ─────► Send Rejection Mail (Gmail)
```

![HireSense AI workflow](docs/screenshots/workflow.png)

---

## 📸 Screenshots

### Workflow: Shortlist path
![Shortlist path](docs/screenshots/workflow.png)

### Workflow: Reject path
![Reject path](docs/screenshots/workflow-reject-path.png)

### Candidate database (Google Sheets)
Every screened resume is saved with score, skills, strengths, gaps and recommendation. (Contact details blurred.)

![Candidate sheet](docs/screenshots/candidate-sheet.png)

### Interview invitation email
<img src="docs/screenshots/interview-email.png" width="320" alt="Interview invitation email" />

### Auto-created Google Calendar event with Meet
<img src="docs/screenshots/calendar-event.png" width="320" alt="Calendar event" />

---

## 🛠 Tech Stack

| Layer | Tool |
|---|---|
| Automation / orchestration | [n8n](https://n8n.io) |
| AI model | Google Gemini (Flash) via Google AI Studio |
| Structured output | n8n Structured Output Parser |
| Input | n8n Form Trigger |
| Storage / dashboard | Google Sheets |
| Scheduling | Google Calendar (with Google Meet) |
| Email | Gmail |
| Resume parsing | n8n Extract From File (PDF) |

---

## ⚙️ How It Works

1. **Form submission:** The candidate fills in name, email, role and uploads a PDF resume.
2. **Text extraction:** The PDF is converted to plain text.
3. **AI input preparation:** The resume text and the job description are combined into one prompt.
4. **AI scoring:** Gemini returns structured JSON:

```json
{
  "name": "Candidate Name",
  "email": "candidate@email.com",
  "phone": "9999999999",
  "skills": ["Python", "n8n"],
  "years_experience": 1,
  "education": "B.E. Computer Science",
  "match_score": 78,
  "strengths": "Two lines on strengths",
  "gaps": "Two lines on gaps",
  "recommendation": "Shortlist"
}
```

5. **Save:** The row is added to Google Sheets.
6. **Route:** A Switch node checks `recommendation`.
7. **Act:**
   - **Shortlist** creates a calendar event with a Meet link and emails the invite.
   - **Maybe** emails HR the score, strengths and gaps for a manual decision.
   - **Reject** sends a polite rejection (no score shared with the candidate).

---

## 🚀 Setup Guide

### Prerequisites

- An [n8n](https://n8n.io) instance (Cloud trial or self-hosted)
- A Google account (Sheets, Gmail, Calendar)
- A free Gemini API key from [Google AI Studio](https://aistudio.google.com)

### Steps

1. **Clone this repo**

   ```bash
   git clone https://github.com/gangadevi2931-dotcom/hiresense-ai.git
   cd hiresense-ai
   ```

2. **Import the workflow** into n8n:
   `Workflows → ⋯ → Import from File → workflow/hiresense-ai.json`

3. **Create the Google Sheet** named `Resume Screening` with the columns listed in [Google Sheet Structure](#-google-sheet-structure).

4. **Add credentials in n8n** (use one Google account for all of them):
   - Google Sheets OAuth2
   - Gmail OAuth2
   - Google Calendar OAuth2
   - Google Gemini (PaLM) API key

5. **Update the nodes:**
   - Google Sheets node: select your spreadsheet and sheet
   - Gmail nodes: check the `To` field (use your own email while testing)
   - Calendar node: choose your calendar

6. **Test:** Click **Execute workflow**, open the form's Test URL, upload a sample resume.

7. **Go live:** Click **Publish**, then share the form's **Production URL**.

> ⚠️ While testing, use your own email in the form so real people do not receive invitations by mistake.

---

## 📊 Google Sheet Structure

Create a sheet named **Resume Screening** with this header row (exact spelling):

| name | email | phone | skills | years_experience | education | match_score | strengths | gaps | recommendation | date |
|---|---|---|---|---|---|---|---|---|---|---|

---

## 📧 Email Templates

The workflow sends styled HTML emails with inline CSS (so they render correctly in Gmail):

| Outcome | Style | Contents |
|---|---|---|
| **Shortlist** | Indigo header | Role, date and time, Google Meet "Join Interview" button |
| **Maybe (to HR)** | Amber header | Candidate details, score badge, strengths and gaps cards |
| **Reject** | Slate header | Polite message with encouragement, no score shared |

Templates live in the `email-templates/` folder.

---

## 📁 Project Structure

```
hiresense-ai/
├── README.md
├── workflow/
│   └── hiresense-ai.json        # n8n workflow export
├── email-templates/
│   ├── interview-invite.html
│   ├── hr-review.html
│   └── rejection.html
├── prompts/
│   └── scoring-prompt.md        # AI scoring prompt
└── docs/
    └── screenshots/
        ├── workflow.png
        ├── workflow-reject-path.png
        ├── candidate-sheet.png
        ├── interview-email.png
        └── calendar-event.png
```

---

## 🗺 Roadmap

- [x] Role-based skill matching (AI builds the required skills for the selected role)
- [x] Candidate name, email and role taken from the form instead of AI-extracted values
- [ ] OCR support for image-only (scanned) PDF resumes
- [ ] Optional "Job Description" field so candidates can paste a real JD
- [ ] Mode switch: "Resume Feedback (practice)" vs "Real Hiring"
- [ ] Duplicate resume check (match on email)
- [ ] Free-slot detection with Google Calendar availability
- [ ] HR approval step before sending rejection mails
- [ ] Error-handling workflow with alerts
- [ ] Dashboard in Looker Studio
- [ ] Bias-safe mode (hide name and college before scoring)

---

## 🩺 Troubleshooting

| Problem | Fix |
|---|---|
| `The item has no binary field 'data'` | In **Extract From File**, set *Input Binary Field* to the form's file field label (for example `Resume`). |
| Good resume gets Reject with score 0 | The PDF is probably an image or scan (no selectable text). Export the resume as a text-based PDF from Word or Google Docs. |
| AI returns score 0 / "No resume content" | Make sure the prompt uses the resume text field (for example `{{ $json.resume_text }}`) and *Include Other Input Fields* is ON. |
| Output Parser circle not visible on AI node | Turn ON **Require Specific Output Format**. |
| Gmail node shows "Node was not executed" | Run the full workflow. Branch nodes only run when the Switch sends data to them. |
| Download file: "File not found" | Use the same Google credential in every Google node. |
| 429 / quota error from Gemini | Wait a minute or switch to a lighter Flash model. |

---

## 🔐 Security Notes

- Never commit API keys or OAuth secrets. n8n workflow exports contain only credential names and IDs, but check the JSON before pushing.
- Resumes contain personal data (phone, email). Keep your Google Sheet private and tell users their resume is processed by AI.

---

## 👩‍💻 Author

**Ganga**
CSE (AI & ML) student | AI Automation Engineer in the making | Founder of *Ganga Automation*

- GitHub: [gangadevi2931-dotcom](https://github.com/gangadevi2931-dotcom)
- LinkedIn: [linkedin.com/in/ganga2931](https://linkedin.com/in/ganga2931)
- Instagram: [@automation.unlocked](https://instagram.com/automation.unlocked)

---

⭐ If you found this project useful, please give it a star!
