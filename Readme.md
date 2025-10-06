# RCA Script - Complete User Guide

## 📖 Table of Contents
1. [Overview](#overview)
2. [Menu Options Explained](#menu-options-explained)
3. [How It Works Behind the Scenes](#how-it-works-behind-the-scenes)
4. [Step-by-Step Usage](#step-by-step-usage)
5. [Advanced Features](#advanced-features)
6. [Troubleshooting](#troubleshooting)

---

## Overview

The **RCA Report Generator** is a comprehensive tool for creating Root Cause Analysis reports, customer health analytics, and extracting insights from ClickUp tickets and Slack conversations.

### What This Tool Does:
✅ Automatically generates professional RCA reports from ClickUp tickets
✅ Analyzes customer health using resource data and Slack sentiment
✅ Extracts technical questions from customer Slack channels
✅ Creates weekly/monthly documentation for all customers
✅ Provides AI-powered insights using GPT-4

---

## Menu Options Explained

When you run `python3 main_script_ai.py`, you'll see this menu:

```
╔══════════════════════════════════════════════════════════════════╗
║                     REPORT GENERATION OPTIONS                     ║
╚══════════════════════════════════════════════════════════════════╝
```

### Option 1: 🔍 Search by Customer & Date Range
**Purpose**: Generate RCA report for a specific customer

**Use when**:
- You need an RCA report for ONE customer
- You want to analyze a specific time period
- Customer requests historical analysis

**How it works**:
1. Select customer from list (Capillary, Vymo, etc.)
2. Choose date range (last 7/30 days, today, or custom)
3. Script fetches ALL ClickUp tickets for that customer and period
4. Generates detailed RCA report with AI analysis

**Behind the scenes**:
- Queries ClickUp API for tickets matching customer + date filter
- Fetches all comments from each ticket
- Queries Slack API for messages in customer channel
- Uses GPT-4 to analyze and generate RCA content
- Creates beautiful HTML report with interactive features

**Output**: `RCA_Report_CustomerName_YYYYMMDD_HHMMSS.html`

---

### Option 2: 📚 Weekly Documentation (Last 7 Days)
**Purpose**: Generate comprehensive report for ALL customers

**Use when**:
- Weekly team sync meetings
- Sending weekly updates to management
- Creating customer success reports

**How it works**:
1. Automatically fetches tickets from ALL customers
2. Filters for last 7 days
3. Groups tickets by customer
4. Generates one comprehensive HTML report

**Behind the scenes**:
- Iterates through every customer in config
- Fetches tickets for each customer (parallel processing)
- Combines all data into single document
- Includes summary statistics and insights

**Output**: `Facets_CS_Documentation_Weekly_YYYYMMDD.html`

---

### Option 3: 📅 Monthly Documentation (Last 30 Days)
**Purpose**: Extended analysis for past month

**Use when**:
- Monthly business reviews
- Quarterly planning
- Trend analysis over longer period

**How it works**: Same as Weekly, but covers 30 days

**Output**: `Facets_CS_Documentation_Monthly_YYYYMMDD.html`

---

### Option 4: 📆 Custom Date Range (All Customers)
**Purpose**: Flexible date range for all customers

**Use when**:
- You need specific date range (e.g., Q3 2025)
- Analyzing specific events or incidents
- Custom reporting requirements

**How it works**:
1. You enter start date: `2025-08-01`
2. You enter end date: `2025-08-31`
3. Script fetches all tickets in that range

**Output**: `RCA_Report_Custom_YYYYMMDD.html`

---

### Option 5: 🎫 Specific Tickets Only
**Purpose**: Analyze specific tickets by URL

**Use when**:
- Customer asks for RCA of particular incident
- Need focused analysis of related tickets
- Quick report for a few tickets

**How it works**:
1. Paste ClickUp ticket URLs (one per line)
2. Press Enter twice when done
3. Script fetches ONLY those tickets
4. Generates focused RCA report

**Example input**:
```
https://app.clickup.com/t/12345678
https://app.clickup.com/t/87654321
```

**Behind the scenes**:
- Extracts ticket IDs from URLs
- Fetches full ticket data including comments
- No date filtering (gets complete ticket history)

**Output**: `RCA_Report_Specific_Tickets_YYYYMMDD.html`

---

### Option 6: 📊 Customer Health Analysis
**Purpose**: Analyze customer resource usage + Slack sentiment

**Use when**:
- Quarterly Business Reviews (QBRs)
- Customer health checks
- Identifying at-risk customers
- Resource optimization planning

**How it works**:
1. Fetches resource data from Control Plane API
2. Analyzes Slack channel sentiment using AI
3. Generates charts and visualizations
4. Creates detailed HTML report

**What it analyzes**:
- **Resources**: CPU, Memory, Storage, Databases
- **Projects**: Resource distribution across projects
- **Slack Sentiment**: Positive/Negative/Neutral analysis
- **AI Insights**: Recommendations and patterns

**Behind the scenes**:
- Queries Control Plane GraphQL API for resource data
- Fetches last 500 Slack messages from customer channel
- Uses GPT-4 for sentiment analysis
- Generates matplotlib charts (PNG images)
- Embeds everything in interactive HTML

**Output**:
- `CustomerName_Analysis_YYYYMMDD.html`
- `CustomerName_resources.png` (chart)
- `CustomerName_projects_breakdown.png` (chart)

---

### Option 7: 📋 Customer Questions Report (Excel)
**Purpose**: Extract all technical questions from Slack channels

**Use when**:
- Identifying common customer pain points
- Building FAQ documentation
- Understanding customer concerns
- Planning product improvements

**How it works**:
1. Select time period (30/60/90/180/365 days or custom)
2. Script scans ALL customer Slack channels
3. Identifies technical questions using AI patterns
4. Filters out Facets team members (by email domain)
5. Generates Excel spreadsheet

**What qualifies as "technical question"**:
✅ Contains technical keywords (error, deployment, pod, api, etc.)
✅ Has question structure (?, how, what, why, can you)
✅ Minimum 4 words (filters out "ok?", "done?")
❌ Simple status questions ("any update?")
❌ Meeting requests ("can we get on call?")

**Behind the scenes**:
- Fetches message history from Slack API
- Applies smart filtering rules
- Checks user email domains to filter team members
- Cleans Slack formatting (mentions, links)
- Groups questions by customer
- Numbers questions per customer (1, 2, 3...)

**Excel format**:
| Customer Name | Questions (numbered) |
|--------------|----------------------|
| Vymo | 1. How to fix deployment error?<br>2. What causes timeout?<br>3. Can we increase memory? |

**Output**: `Customer_Questions_180days_YYYYMMDD.xlsx`

---

## How It Works Behind the Scenes

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                       MAIN SCRIPT                                │
│                   (main_script_ai.py)                            │
└────────────────────────┬────────────────────────────────────────┘
                         │
          ┌──────────────┴───────────────┬──────────────┐
          │                              │              │
┌─────────▼────────┐       ┌─────────────▼──────┐   ┌──▼─────────────┐
│   ClickUp API    │       │   Slack API         │   │  Control Plane │
│  (Tickets/Comments)│      │ (Messages/Sentiment)│   │  API (Resources)│
└──────────────────┘       └─────────────────────┘   └────────────────┘
          │                              │                     │
          └──────────────┬───────────────┴─────────────────────┘
                         │
                ┌────────▼────────┐
                │   AI Processor   │
                │     (GPT-4)      │
                └────────┬─────────┘
                         │
              ┌──────────▼──────────┐
              │  Report Generator    │
              │  (HTML/Excel/Charts) │
              └──────────┬───────────┘
                         │
                 ┌───────▼────────┐
                 │  OUTPUT FILES   │
                 │ (HTML/XLSX/PNG) │
                 └─────────────────┘
```

### Data Flow

1. **User Input** → Menu selection + parameters
2. **API Calls** → Fetch data from ClickUp/Slack/Control Plane
3. **AI Processing** → GPT-4 analyzes conversations and generates RCA
4. **Report Generation** → Create formatted HTML/Excel with embedded data
5. **Output** → Save files locally + optional cloud upload

### Key Components

#### 1. ClickUp Integration (`clickup_extended.py`)
- Fetches tickets with all metadata
- Retrieves all comments and conversation history
- Handles pagination for large datasets
- Rate limiting and error handling

#### 2. Slack Integration (`slack_analyzer.py`)
- Reads channel message history
- Analyzes sentiment using AI
- Filters by user email domains
- Extracts technical questions with pattern matching

#### 3. AI Processor (`ai_processor.py`)
- Uses OpenAI GPT-4 (`gpt-4o-2024-11-20`)
- Structured outputs for consistent formatting
- Prompt engineering for accurate RCA generation
- Validation and quality checks

#### 4. Report Generator (`customer_analysis_reporter.py`)
- Creates interactive HTML with JavaScript
- Generates charts using matplotlib
- Excel generation with openpyxl
- Responsive design with CSS

#### 5. Control Plane Client (`control_plane_client.py`)
- GraphQL API integration
- Resource data fetching
- Project-level breakdowns

---

## Step-by-Step Usage

### First Time Setup

1. **Install dependencies**:
```bash
pip3 install -r requirements.txt
pip3 install -r requirements_customer_analysis.txt
```

2. **Configure API keys** (`config.yaml`):
```yaml
clickup:
  api_token: "your_clickup_token"
openai:
  api_key: "your_openai_key"
slack:
  bot_token: "xoxb-your-slack-token"
control_plane:
  api_url: "https://api.yourorg.controlplane.com"
  api_token: "your_cp_token"
```

3. **Configure customers** (`customer_analysis_config.yaml`):
```yaml
customers:
  - customer_name: "Vymo"
    slack_channel: "C023X4RDS76"
  - customer_name: "Capillary"
    slack_channel: "C02A7G4S53R"
```

### Running Your First Report

**Example 1: Weekly RCA for All Customers**

```bash
python3 main_script_ai.py
```

1. Choose: `2` (Weekly Documentation)
2. Script runs automatically
3. Output: `Facets_CS_Documentation_Weekly_20251006.html`
4. Open in browser to view

**Example 2: Customer Health Analysis**

```bash
python3 main_script_ai.py
```

1. Choose: `6` (Customer Health Analysis)
2. Select customer: `Vymo`
3. Script generates analysis with charts
4. Output: `Vymo_Analysis_20251006_153722.html`

**Example 3: Extract Customer Questions**

```bash
python3 main_script_ai.py
```

1. Choose: `7` (Customer Questions Report)
2. Select time period: `180 days`
3. Script scans all Slack channels
4. Output: `Customer_Questions_180days_20251006.xlsx`
5. Opens automatically in Excel/Numbers

---

## Advanced Features

### AI-Powered RCA Generation

The script uses GPT-4 with custom prompts to:
- Identify root causes from ticket conversations
- Generate debugging steps chronologically
- Suggest resolution steps
- Create professional summaries

**Prompt Engineering**:
- System prompt defines RCA structure
- User prompt includes ticket data + conversations
- Structured outputs ensure consistent formatting
- Validation checks for quality

### Slack Sentiment Analysis

**Sentiment Categories**:
- **Positive**: Appreciation, success messages, resolved issues
- **Negative**: Complaints, errors, failures, delays
- **Neutral**: Status updates, questions, factual statements

**AI Analysis**:
```
Recent Slack conversations show mostly positive sentiment:
- Team quickly resolved production deployment issue
- Customer appreciated fast response time
- However, recurring questions about configuration suggest documentation gaps
```

### Interactive HTML Reports

**Features**:
- 📊 Embedded charts (base64 encoded PNG)
- 🔗 Clickable Slack links
- 🎨 Professional Facets branding
- 📱 Mobile-responsive design
- 💾 Shareable cloud links (optional)

### Excel Reports with Smart Formatting

**Customer Questions Excel**:
- Dynamic row heights (auto-adjusts based on content)
- Alternating row colors for readability
- Sequential numbering per customer
- Cleaned Slack formatting (no user IDs)
- Clickable Slack links

---

## Troubleshooting

### Common Issues

**Issue**: "API connection failed"
```
Solution:
1. Check config.yaml has valid API tokens
2. Test ClickUp token: https://api.clickup.com/api/v2/user
3. Test Slack bot permissions: users:read, channels:history
```

**Issue**: "No questions found" in Excel report
```
Possible causes:
1. Bot not a MEMBER of Slack channel (invite with /invite @Bot-Name)
2. Time period too short (try 180 days)
3. Questions filtered as non-technical (check filtering logic)
```

**Issue**: Customer Health Analysis shows "No data"
```
Solution:
1. Verify Control Plane API token
2. Check customer_name matches Control Plane exactly
3. Ensure resources exist in Control Plane for that customer
```

**Issue**: RCA report empty or incomplete
```
Solutions:
1. Check if tickets have comments (RCA needs conversations)
2. Verify OpenAI API key is valid and has credits
3. Check ticket status (closed tickets work best)
```

### Debug Mode

Enable detailed logging:
```bash
export DEBUG=1
python3 main_script_ai.py
```

### Getting Help

1. Check `QUICKSTART.md` for quick setup
2. Read `CONFIGURATION_GUIDE.md` for detailed config
3. See `CUSTOMER_ANALYSIS_README.md` for health analysis
4. Review `CUSTOMER_QUESTIONS_README.md` for Excel reports

---

## Tips & Best Practices

### For RCA Reports
✅ Wait until tickets are closed for complete analysis
✅ Ensure team members document in ClickUp comments
✅ Link related tickets for comprehensive RCA
✅ Review AI-generated content before sending to customers

### For Customer Health Analysis
✅ Run quarterly for QBRs
✅ Compare month-over-month trends
✅ Share with account managers
✅ Use for capacity planning

### For Questions Reports
✅ Run monthly to identify patterns
✅ Use for FAQ documentation
✅ Share with product team for feature requests
✅ Track question volume as health metric

---
## Support

For issues or feature requests:
- Check existing documentation in this directory
- Review code comments in main_script_ai.py
- Contact your team's DevOps lead

---


