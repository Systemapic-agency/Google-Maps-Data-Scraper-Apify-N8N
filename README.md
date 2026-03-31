# Google-Maps-Data-Scraper-Apify-N8N

A powerful **n8n lead generation and business data scraping workflow** that collects business information from **Google Maps** using **Apify**, removes duplicates intelligently using **AI + Google Sheets matching**, and automatically stores fresh leads into **category-specific Google Sheets tabs**.

This workflow is designed for agencies, freelancers, outreach teams, local SEO professionals, appointment setters, and lead generation businesses who want a semi-automated system for collecting clean local business data.

---

## Features

- Accepts business scraping requests through an **n8n form**
- Scrapes business data from **Google Maps**
- Uses **Apify Google Maps Email Extractor**
- Supports multiple business categories
- Uses AI to **remove duplicate businesses**
- Compares scraped leads with existing Google Sheets records
- Stores clean leads into **separate category-specific tabs**
- Captures useful business fields such as:
  - business name
  - address
  - phone number
  - email
  - website
  - social links

---

## Workflow Overview

This workflow follows the process below:

1. User submits a form inside n8n
2. Workflow receives:
   - business category
   - country
   - state
   - city
   - extraction quantity
3. Workflow sends the search request to **Apify**
4. Apify scrapes business records from **Google Maps**
5. Results are returned into n8n
6. An AI agent checks for duplicates against the correct Google Sheet tab
7. New unique businesses are extracted and normalized
8. A **Switch node** routes the leads into the correct Google Sheets tab
9. Clean leads are stored automatically

---

## Main Use Case

This workflow is ideal for:

- local lead generation
- agency prospecting
- Google Maps scraping
- business directory building
- outreach list building
- email list enrichment
- local SEO prospect research

---

## Supported Business Categories

This workflow supports scraping and sorting for the following business types:

- GlowUp Studio
- Restaurant
- Sports
- Bakery
- Photography Studio
- Travel Agency
- Event Management
- Fitness Center/Gym
- Digital Marketing Agency
- Plumber

Each category is routed into its own dedicated **Google Sheets tab**.

---

## Nodes Used

## 1. Form Submission Layer

### 1. On form submission
This workflow starts with an **n8n Form Trigger**.

The form collects the following user inputs:

- **Business name**
- **Country name**
- **State**
- **City**
- **Total Num Of Extraction**

### Form example:
```text
Business name: Photography Studio
Country name: us
State: ny
City: ny
Total Num Of Extraction: 50
```

This makes the workflow flexible and reusable for different scraping requests.

---

## 2. Data Scraping Layer

### 2. Call Apfiy
This node sends the scraping request to **Apify**.

### API used:
```text
https://api.apify.com/v2/acts/poidata~google-maps-email-extractor/runs
```

### It sends:
- search term
- location
- country
- language
- total extraction count

### Request payload includes:
```json
{
  "term": ["Photography Studio"],
  "location": "ny, us",
  "language": "en",
  "country": "us",
  "total": 50
}
```

This starts the Google Maps scraping run.

---

### 3. Get data from Apify
Once Apify completes the scraping run, this node fetches the final dataset.

### API used:
```text
https://api.apify.com/v2/datasets/{{ datasetId }}/items
```

This returns the raw business records scraped from Google Maps.

### Typical returned fields may include:
- business name
- categories
- address
- phone
- email(s)
- website
- social media links

This becomes the lead dataset for processing.

---

## 3. AI Deduplication Layer

### 4. AI Agent
This is the **duplicate removal and lead filtering brain** of the workflow.

The AI agent receives:

- scraped businesses from Apify
- selected category from the form
- access to the correct Google Sheet tab

### Its main job:
- compare newly scraped leads against existing leads
- remove duplicates
- only keep **new unique businesses**

---

### Duplicate Matching Logic

The AI agent follows a structured matching system.

#### Primary Match: Phone Number
It first normalizes and compares phone numbers.

### Phone normalization rules:
- keeps only digits
- removes leading `1` if present
- compares normalized versions

### Example:
```text
+1 (718) 500-0657 → 7185000657
```

If the phone already exists in the Google Sheet, the business is excluded.

---

#### Fallback Match: Name + Address
If no phone number exists, it uses:

- business name
- address

### Matching rules:
- lowercase
- trimmed
- normalized spacing

### Fallback key format:
```text
<name_norm> | <address_norm>
```

If that combination already exists, the lead is excluded.

---

#### Special Rules
- If no phone and no usable name+address exist, the lead is treated as **new**
- Addresses must be preserved exactly
- No unwanted transformation of business data

This makes the lead cleaning layer much more reliable than simple spreadsheet matching.

---

### 5. OpenRouter Chat Model
This node powers the AI duplicate matching process.

### Model used:
```text
openai/gpt-3.5-turbo
```

It supports the AI agent’s decision-making and structured filtering.

---

## 4. Existing Data Lookup Layer

To perform duplicate checks, the AI agent is connected to multiple **Google Sheets Tool nodes**.

Each tool reads existing leads from the correct sheet tab before any new leads are added.

### Read-only lookup tools include:
- Get row(s) Glowup studio
- Get row(s) Restaurant
- Get row(s) Sports
- Get row(s) Bakery
- Get row(s) Photography Studio
- Get row(s) Travel Agency
- Get row(s) Event Management
- Get row(s) Fitness Center/Gym
- Get row(s) Digital Marketing Agency
- Get row(s) Plumber

These are used only for **reading and matching existing records**, not writing.

---

## 5. Data Cleanup Layer

### 6. Code in JavaScript
This node processes the AI output and converts it into clean lead records.

### What it does:
- parses the AI JSON response
- safely extracts valid lead objects
- standardizes missing values
- formats phone numbers
- outputs clean rows for Google Sheets insertion

### Output fields include:
- `name`
- `address`
- `phone`
- `email`
- `website`
- `facebook`
- `instagram`
- `tiktok`
- `youtube`
- `twitter`

### Example cleaned output:
```json
{
  "name": "Vandervoort Studio",
  "address": "309 Vandervoort Ave, Brooklyn, NY 11211",
  "phone": "(718) 500-0657",
  "email": "",
  "website": "",
  "facebook": "",
  "instagram": "",
  "tiktok": "",
  "youtube": "",
  "twitter": ""
}
```

This node acts as the final **lead normalizer** before routing.

---

## 6. Routing Layer

### 7. Switch1
This node routes each clean lead into the correct **Google Sheets tab** based on the selected business category.

### Supported switch paths:
- GlowUp Studio
- Restaurant
- Sports
- Bakery
- Photography Studio
- Travel Agency
- Event Management
- Fitness Center/Gym
- Digital Marketing Agency
- Plumber

This allows the workflow to function like a **multi-niche lead management system**.

---

## 7. Google Sheets Storage Layer

Each category has its own dedicated append node.

The workflow stores leads in a central spreadsheet called:

```text
Google maps data scraper
```

---

### 8. Sheet name glow up studio
Stores new leads for **GlowUp Studio**.

### Stored fields:
- Business name
- Full Address
- Phone number
- Email
- Facebook link
- Youtube link
- Twitter link

---

### 9. Sheet name Restaurant1
Stores new leads for **Restaurant** businesses.

### Stored fields:
- Business name
- Full Address
- Phone number
- Email

---

### 10. Sheet name Sports1
Stores new leads for **Sports** businesses.

### Stored fields:
- Business name
- Full Address
- Phone number
- Email

---

### 11. sheet Name Bakery1
Stores new leads for **Bakery** businesses.

### Stored fields:
- Business name
- Full Address
- Phone number
- Email

---

### 12. sheet Name Photography Studio1
Stores new leads for **Photography Studio** businesses.

### Stored fields:
- Business name
- Full Address
- Phone number
- Email

This tab uses direct business naming instead of category mapping.

---

### 13. sheet Name Travel Agency1
Stores new leads for **Travel Agency** businesses.

### Stored fields:
- Business name
- Full Address
- Phone number
- Email

---

### 14. sheet Name Event Management1
Stores new leads for **Event Management** businesses.

### Stored fields:
- Business name
- Full Address
- Phone number
- Email

---

### 15. sheet Name Fitness Center/Gym1
Stores new leads for **Fitness Center/Gym** businesses.

### Stored fields:
- Business name
- Full Address
- Phone number
- Email

---

### 16. sheet Name Digital Marketing Agency1
Stores new leads for **Digital Marketing Agency** businesses.

### Stored fields:
- Business name
- Full Address
- Phone number
- Email

---

### 17. sheet Name Plumber
Stores new leads for **Plumber** businesses.

### Stored fields:
- Business name
- Full Address
- Phone number
- Email

---

## Example Workflow Flow

A user wants to scrape **Photography Studios in New York**.

### Example form input:
```text
Business name: Photography Studio
Country name: us
State: ny
City: ny
Total Num Of Extraction: 50
```

### Workflow process:
1. Form is submitted
2. Apify scrapes Google Maps businesses
3. Results are returned to n8n
4. AI checks existing Photography Studio records in Google Sheets
5. Duplicate leads are removed
6. Only fresh businesses are kept
7. Clean records are appended into the **Photography Studio** tab

---

## Example Output Fields

Depending on the source business data, the workflow may capture:

- business name
- category
- full address
- phone number
- email
- website
- facebook
- instagram
- tiktok
- youtube
- twitter

---

## Requirements

To run this workflow successfully, you need:

- **n8n**
- **Apify API access**
- **Google Sheets OAuth2 credentials**
- **OpenRouter API credentials**
- a configured Google Sheet with matching category tabs

---

## External Services Used

This workflow integrates with:

- **Apify**
- **Google Maps**
- **OpenRouter**
- **Google Sheets**

---

## Google Sheet Structure Used

The spreadsheet is used as both:

- a lead database
- a duplicate-checking source

### Common columns used:
| Column | Purpose |
|---|---|
| Business name | Business title |
| Full Address | Full business address |
| Phone number | Contact number |
| Email | Business email |
| Website | Optional website |
| Facebook link | Optional social link |
| Instagram link | Optional social link |
| Tiktok link | Optional social link |
| Youtube link | Optional social link |
| Twitter link | Optional social link |

> Note: Some category tabs use fewer columns than others.

---

## Setup Instructions

1. Import the JSON workflow into n8n
2. Connect your **Google Sheets credentials**
3. Connect your **OpenRouter credentials**
4. Add your **Apify API token**
5. Verify your spreadsheet tabs exist for each business category
6. Make sure column names in Google Sheets match your workflow mappings
7. Run a test form submission
8. Confirm:
   - Apify returns business records
   - AI deduplication works
   - new leads are routed correctly
   - rows are appended into the correct sheet

---

## Business Categories Routing Logic

The **Switch node** routes based on the submitted business category.

### Example:
```text
If Business name contains "Restaurant"
→ Send to Restaurant sheet
```

### Example:
```text
If Business name contains "Photography Studio"
→ Send to Photography Studio sheet
```

This makes the workflow easy to expand with more niches later.

---

## Important Notes

- This workflow is built for **local lead scraping**
- It works best for service businesses and local commercial categories
- Duplicate detection depends heavily on:
  - phone number availability
  - consistent business name
  - usable address data
- Some Google Maps records may have incomplete contact information
- The workflow is optimized for **sheet-based lead organization**

---

## Limitations

At the moment, the workflow has a few practical limitations:

- No built-in email verification
- No website scraping after Google Maps extraction
- No lead scoring system
- No CRM sync
- No export to CSV or Airtable
- No retry logic if Apify fails
- No webhook delivery of leads to external systems

---

## Suggested Improvements

You can improve this workflow further by adding:

- email verification layer
- website scraping enrichment
- LinkedIn company extraction
- AI lead scoring
- CRM integration
- Airtable sync
- CSV export
- lead owner assignment
- cold email workflow trigger
- niche-specific outreach message generation

---

## Recommended Workflow Expansion

To turn this into a complete **local lead generation engine**, combine it with:

1. email verification workflow
2. AI cold email generator
3. CRM upload workflow
4. appointment setter pipeline
5. outreach automation system

This can become a full **agency lead scraping and outreach machine**.

---

## Example Real-World Use Cases

This workflow is useful for:

### Agencies
Scrape local businesses for outreach campaigns.

### Freelancers
Find local businesses who need websites, SEO, ads, or branding.

### Appointment Setters
Build prospect lists for local service businesses.

### Local SEO Providers
Generate niche-specific lead databases by city.

### Marketing Teams
Create segmented local prospect lists by business type.

---

## Author

Built as an **n8n Google Maps lead scraping and deduplication workflow** for lead generation, outreach, and local business prospecting systems.

---

## License

You can add your preferred license here, such as:

- MIT
- Apache-2.0
- Proprietary
