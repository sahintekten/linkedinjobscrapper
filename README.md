# linkedinjobscrapper

An [n8n](https://n8n.io) workflow that searches LinkedIn for internship postings matching our filters and saves them to a Google Sheet for further applications.

![Google Sheet populated by the workflow](Screenshot%202025-10-19%20at%2017.26.39.png)

## How it works

The workflow (`Linkedin job scrapper.json`) is a linear pipeline of four nodes:

1. **Manual trigger** (`n8n-nodes-base.manualTrigger`): the workflow is run on demand with "Execute workflow".
2. **Apify: Run task and get dataset** (`@apify/n8n-nodes-apify.apify`): runs a saved Apify actor task ("Linkedin Jobs Scraper Mark 1") that scrapes LinkedIn job postings, then returns the resulting dataset (timeout: 180 s). The search filters live in the Apify task's input.
3. **Edit Fields** (`n8n-nodes-base.set`): maps each scraped posting to a clean row:
   | Column | Source |
   |---|---|
   | Company Name | `companyName`, or derived from the company's LinkedIn URL |
   | Job Title | `title` |
   | URL to Posting | `applyUrl`, falling back to `url` |
   | Job Posted Date | `postedAt` / `datePosted`, formatted as `dd-mm-yy` |
   | Location | `location` |
   | Job Summary / Key Requirements | `descriptionText`, truncated to 4,000 characters |
   | Pay/Stipend | left empty (skipped) |
   | Contact Person, Contact Info | left empty (a note in the workflow says to fill them later with Hunter.io) |
4. **Google Sheets: Append or update row in sheet** (`n8n-nodes-base.googleSheets`): writes the rows to the tracking spreadsheet, auto-mapping fields to the columns above.

## Usage

1. In n8n, import `Linkedin job scrapper.json`.
2. Connect your own **Apify** (OAuth2) and **Google Sheets** (OAuth2) credentials.
3. Select your own Apify actor task (with your search filters) and your target spreadsheet in the two nodes.
4. Click **Execute workflow**.
