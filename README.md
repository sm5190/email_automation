# AI-Powered Cold Outreach Email Automation in n8n

This project is an n8n workflow that automates personalized cold outreach email drafting at scale.

It accepts two uploaded PDF files through an n8n form:

1. A contact list PDF containing recipient details
2. A resume PDF containing candidate background and experience

The workflow extracts and parses both files, combines the data, uses an AI agent to generate personalized outreach emails, creates Gmail drafts with the resume attached, logs the activity to Google Sheets, and waits between batches to control sending volume. 

## Features

- Upload contact list and resume through an n8n form
- Extract structured data from PDFs
- Parse recruiter/contact information from a PDF table
- Parse resume details including education, experience, projects, and skills
- Generate tailored cold outreach emails with an AI agent
- Enforce structured JSON output for subject and body
- Create Gmail drafts automatically
- Attach the uploaded resume PDF to each draft
- Log outreach status to Google Sheets
- Process contacts in batches with a delay between runs 

## Workflow Overview

### 1. Form submission
The workflow starts with an `On form submission` trigger where the user uploads:
- the email/contact PDF
- the resume PDF :contentReference[oaicite:3]{index=3}

### 2. PDF extraction
Two `Extract from File` nodes read text from both PDFs:
- one for the contact file
- one for the resume file :contentReference[oaicite:4]{index=4}

### 3. Contact parsing
A JavaScript code node parses the extracted contact PDF into structured rows with fields such as:
- `name`
- `email`
- `company`
- `is_recruiter`
- `done`

This makes the recipient list usable for downstream automation. :contentReference[oaicite:5]{index=5}

### 4. Resume parsing
Another JavaScript code node parses the resume PDF into structured JSON, including:
- basic info
- education
- experience
- projects
- skills

This gives the AI agent rich context for generating personalized outreach. :contentReference[oaicite:6]{index=6}

### 5. Merge and prepare items
The workflow merges the parsed contact list with the parsed resume and prepares one item per contact, while also carrying forward the uploaded resume as a binary attachment. 

### 6. Batch processing
The `Loop Over Items` node processes recipients in batches. In the uploaded workflow, the batch size is currently set to `10`. :contentReference[oaicite:8]{index=8}

### 7. AI-generated outreach email
An AI Agent uses the recipient details plus parsed resume content to generate:
- a professional subject line
- a concise personalized body

The prompt is designed to position the candidate strongly for:
- backend engineering
- distributed systems
- cloud infrastructure
- production AI / LLM systems
- agentic AI roles :contentReference[oaicite:9]{index=9}

### 8. Structured output parsing
A structured output parser ensures the AI returns valid JSON in the format:

```json
{
  "subject": "string",
  "body": "string"
}
```

This keeps the Gmail draft step stable and predictable.

## 9. Gmail draft creation

For each contact, the workflow creates a Gmail draft:

- to the contact's email address
- with the AI-generated subject
- with the AI-generated body
- with the uploaded resume attached

## 10. Google Sheets logging

After the draft is created, the workflow appends a row to a Google Sheet to track outreach status, including:

- name
- email
- company
- mailed = Yes
- response_received = No

## 11. Wait and continue

A `Wait` node pauses execution for 1 hour before continuing the next batch. The workflow loops back into the batch processor after the wait.

## Node Summary

- **On form submission**: collects contact PDF and resume PDF
- **Extract from File**: extracts text from contact PDF
- **Code in JavaScript**: parses contact table into structured records
- **Extract from File1**: extracts text from resume PDF
- **Code in JavaScript1**: parses resume into structured JSON
- **Merge**: merges workflow branches
- **Code in JavaScript2**: combines contact and resume data, prepares attachment
- **Loop Over Items**: processes recipients in batches
- **AI Agent**: generates tailored cold outreach emails
- **Structured Output Parser**: validates JSON response
- **Code in JavaScript3**: maps binary resume for Gmail attachment
- **Create a draft**: creates Gmail draft with attachment
- **Append row in sheet**: logs outreach status to Google Sheets
- **Wait**: delays the next batch by 1 hour

## Requirements

Before using this workflow, configure the following in n8n:

- OpenAI credentials
- Gmail OAuth2 credentials
- Google Sheets OAuth2 credentials

You also need:

- a contact PDF with recipient information
- a resume PDF
- a Google Sheet for outreach logging

## Expected Input Format

### Contact PDF

The contact file should contain rows with data that can be parsed into fields like:

- Name
- Email
- Company
- Recruiter flag
- Done flag

The parsing logic expects each row to contain an email address and surrounding text that can be split into the relevant fields.

### Resume PDF

The resume parser works best when the resume contains recognizable section headers such as:

- Education
- Experience
- Projects
- Technical Skills / Skills

It then extracts structured data from those sections for prompt personalization.

## Setup

1. Import `Email_automation.json` into n8n
2. Connect your OpenAI account
3. Connect your Gmail account
4. Connect your Google Sheets account
5. Update the Google Sheet destination if needed
6. Test the form trigger by uploading:
   - a contact PDF
   - your resume PDF
7. Run the workflow and verify draft creation and logging

## Use Cases

This workflow is useful for:

- job application outreach
- recruiter outreach
- founder outreach
- targeted cold emailing for software engineering roles
- semi-automated personal branding and job search operations

## Notes

- The current workflow creates drafts instead of sending emails directly, which adds a human review step before sending.
- The current batch size in the uploaded workflow is 10.
- The current wait interval is 1 hour.
- Resume personalization is prompt-driven, so output quality depends on resume clarity and contact data quality.

## Future Improvements

Possible enhancements:

- skip already-contacted emails automatically
- detect duplicate contacts before drafting
- send emails directly after approval
- add retry and error handling paths
- enrich contact company information from external sources
- support CSV or Google Sheets contact input instead of PDF
- add approval workflow before draft creation
- log Gmail draft IDs and timestamps for auditing

## License

This project is for personal productivity and outreach automation. Add your preferred license before publishing publicly.
