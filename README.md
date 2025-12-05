# Buildium Outstanding Balance Report Workflow (Iris)

## 📋 Overview

This is an automated n8n workflow that generates outstanding balance reports for property management using the Buildium API. The workflow integrates with Telegram for voice/text input, uses AI (Claude & OpenAI) for query classification and interpretation, and automatically generates formatted reports delivered via Telegram and Google Sheets, with task creation in Monday.com.

**Workflow Name:** `Buildium Outstanding Balance Report_latest_workflow`

---

## 🎯 Key Features

### 1. **Multi-Input Support**
- **Telegram Voice Messages**: Record and send voice queries
- **Telegram Text Messages**: Type queries directly
- **Scheduled Automation**: Automatic reports on the 6th of each month at 9 AM

### 2. **AI-Powered Query Processing**
- **OpenAI GPT-4o-mini**: Classifies queries as property-related or general
- **Claude Sonnet 4**: Interprets property management queries and determines required API calls
- Supports general conversational queries with intelligent responses

### 3. **Comprehensive Report Generation**
- Fetches outstanding balances from Buildium API
- Retrieves tenant details, property information, and lease data
- Calculates days delinquent and priority levels (Normal, High, Urgent)
- Tracks late fees since the last 6th of the month
- Groups and sorts data by state

### 4. **Multi-Channel Output**
- **Telegram**: 
  - Text reports with emoji formatting
  - Voice responses using OpenAI TTS
  - Paginated messages for long reports
- **Google Sheets**: 
  - Automatically created spreadsheets with formatted data
  - Shared with view/edit permissions
  - Organized in a specific Google Drive folder
- **Monday.com**: 
  - Automated task creation
  - Assigned to specific team member
  - Includes report summary and spreadsheet link

---

## 🏗️ Architecture

### Workflow Structure

```
┌─────────────────────────────────────────────────────────────┐
│                       INPUT NODES                            │
├─────────────────┬───────────────────────────────────────────┤
│ Telegram Trigger│ Schedule Trigger (6th of month @ 9 AM)    │
└────────┬────────┴───────────────┬───────────────────────────┘
         │                        │
         ▼                        ▼
    ┌────────────────────────────────────┐
    │  Input Processing & Merge          │
    │  (Voice/Text/Scheduled)            │
    └─────────────┬──────────────────────┘
                  │
                  ▼
    ┌────────────────────────────────────┐
    │  OpenAI Query Classifier           │
    │  (Property vs General Query)       │
    └─────────────┬──────────────────────┘
                  │
         ┌────────┴────────┐
         ▼                 ▼
┌─────────────────┐  ┌──────────────────┐
│ General Query   │  │ Property Query   │
│ (GPT Response)  │  │ (Claude + APIs)  │
└─────────────────┘  └────────┬─────────┘
                              │
                              ▼
            ┌──────────────────────────────────┐
            │  Data Fetching & Processing      │
            │  1. Outstanding Balances         │
            │  2. Lease Details                │
            │  3. Property Details             │
            │  4. Transaction History          │
            └────────────┬─────────────────────┘
                         │
                         ▼
            ┌──────────────────────────────────┐
            │  Data Merging & Calculation      │
            │  - Group by tenant               │
            │  - Calculate days delinquent     │
            │  - Calculate late fees           │
            │  - Sort by state                 │
            └────────────┬─────────────────────┘
                         │
                         ▼
            ┌──────────────────────────────────┐
            │  Report Generation               │
            │  - Format for Telegram           │
            │  - Create Google Sheets          │
            │  - Generate Monday.com task      │
            └────────────┬─────────────────────┘
                         │
                         ▼
            ┌──────────────────────────────────┐
            │  Multi-Channel Output            │
            │  ✓ Telegram message/voice        │
            │  ✓ Google Sheets spreadsheet     │
            │  ✓ Monday.com task               │
            └──────────────────────────────────┘
```

---

## 🔧 Configuration Requirements

### 1. **API Credentials**

#### Telegram Bot
- **Required**: Telegram Bot Token
- **Setup**: Create a bot via [@BotFather](https://t.me/botfather)
- **Credentials needed**:
  - `telegramApi` (for trigger and sending messages)
  - Your Telegram Chat ID (for automated reports)

#### Buildium API
- **Required**: Buildium API credentials
- **Credentials needed**:
  - `x-buildium-client-id`
  - `x-buildium-client-secret`
- **Endpoints used**:
  - `/v1/leases/outstandingbalances`
  - `/v1/leases/{LeaseId}`
  - `/v1/rentals/{PropertyId}`

#### OpenAI API
- **Required**: OpenAI API key
- **Usage**:
  - GPT-4o-mini for query classification
  - Text-to-Speech for voice responses
  - Transcription for voice inputs
- **Credentials needed**:
  - `openAiApi` credential ID
  - Bearer token in headers

#### Anthropic API (Claude)
- **Required**: Anthropic API key
- **Usage**: Claude Sonnet 4 for query interpretation
- **Credentials needed**:
  - `x-api-key` in headers

#### Google APIs
- **Required**: Google OAuth 2.0 credentials
- **Scopes needed**:
  - Google Sheets API (create, read, write)
  - Google Drive API (file management, permissions)
- **Credentials needed**:
  - OAuth2 token for API access
  - Google Drive Folder ID (for organizing reports)

#### Monday.com API
- **Required**: Monday.com API token
- **Usage**: Task creation and updates
- **Credentials needed**:
  - Authorization token
  - Board ID
  - Person ID (for task assignment)
  - Column IDs for custom fields

---

### 2. **Environment Variables & IDs**

Replace these placeholders in `workflow.json`:

```json
{
  "YOUR_TELEGRAM_API_ID": "Your Telegram bot credential ID",
  "YOUR_TELEGRAM_CHAT_ID": "Your Telegram chat ID for automated messages",
  "YOUR_OPENAI_API_ID": "Your OpenAI credential ID",
  "YOUR_OPENAI_API_KEY": "Your OpenAI API key",
  "YOUR_ANTHROPIC_API_KEY": "Your Anthropic (Claude) API key",
  "YOUR_BUILDIUM_CLIENT_ID": "Your Buildium client ID",
  "YOUR_BUILDIUM_CLIENT_SECRET": "Your Buildium client secret",
  "YOUR_BUILDIUM_AUTH_ID": "Your Buildium auth credential ID",
  "YOUR_GOOGLE_OAUTH_ID": "Your Google OAuth credential ID",
  "YOUR_GOOGLE_DRIVE_FOLDER_ID": "Google Drive folder ID for reports",
  "YOUR_MONDAY_COM_API_TOKEN": "Your Monday.com API token",
  "YOUR_MONDAY_BOARD_ID": "Monday.com board ID",
  "YOUR_MONDAY_PERSON_ID": "Monday.com person ID for task assignment",
  "YOUR_HEADER_AUTH_ID": "Header authentication credential ID"
}
```

---

## 📦 Installation

### Prerequisites
- n8n instance (self-hosted or cloud)
- All required API accounts and credentials

### Steps

1. **Import the workflow**
   ```bash
   # In n8n, go to Workflows → Import from File
   # Select: workflow.json
   ```

2. **Configure credentials**
   - Go to **Settings → Credentials**
   - Create credentials for each service:
     - Telegram API
     - OpenAI API
     - Anthropic API (Header Auth)
     - Buildium API (HTTP Basic Auth + Header Auth)
     - Google OAuth2 API
     - Monday.com API (Header Auth)

3. **Update workflow nodes**
   - Replace all placeholder values with your actual IDs
   - Update the Schedule Trigger if needed (default: 6th of month @ 9 AM)
   - Update Monday.com column IDs to match your board structure

4. **Test the workflow**
   - Send a test message via Telegram: "Generate outstanding balance report"
   - Verify all integrations are working

5. **Activate the workflow**
   - Toggle the workflow to "Active"

---

## 🚀 Usage

### Via Telegram

#### Text Query
1. Open your Telegram bot chat
2. Send: `"Generate outstanding balance report"`
3. Wait for the AI to process and fetch data
4. Receive formatted report via text message

#### Voice Query
1. Open your Telegram bot chat
2. Record a voice message: "Show me the outstanding balances"
3. The system transcribes and processes your request
4. Receive response as text or voice

### Via Schedule
- Automatically runs on the **6th of each month at 9:00 AM**
- Generates report without manual trigger
- Sends to configured Telegram chat ID
- Creates Google Sheet and Monday.com task

### Sample Queries
- "Show me outstanding balances"
- "Generate balance report"
- "Which tenants owe money?"
- "Outstanding balance report for active leases"

---

## 📊 Report Structure

### Telegram Report Format

```
📊 OUTSTANDING BALANCE REPORT
Generated: [Date & Time]
Last 6th: [Date]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📈 SUMMARY:
• Total Tenants: [Number]
• Tenants with Outstanding Balance: [Number]
• Total Outstanding: $[Amount]
• Priority Breakdown:
  - Urgent: [Count]
  - High: [Count]
  - Normal: [Count]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🏛️ [STATE NAME] ([Count] tenant(s))
──────────────────────────────
1. [Tenant Name]
   📍 [Property Address]
   💰 Balance: $[Amount]
   ⏰ Days Delinquent: [Days]
   🚨 Priority: [URGENT/HIGH/NORMAL]

[... more tenants ...]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━
End of Report ([X] page(s))
```

### Google Sheets Columns

| Column               | Description                          |
|---------------------|--------------------------------------|
| Tenant Name         | Full name of tenant(s)               |
| Property Address    | Property address                      |
| State               | Property state                        |
| Outstanding Balance | Current total balance                 |
| Days Delinquent     | Days since payment due                |
| Priority            | Normal/High/Urgent                    |

### Priority Levels

- **Urgent**: 31+ days delinquent
- **High**: 16-30 days delinquent
- **Normal**: 6-15 days delinquent
- **Current**: 0-5 days

---

## 🔍 Workflow Node Details

### Key Nodes

#### 1. **Telegram Trigger**
- Listens for incoming messages
- Supports both text and voice messages
- Triggers workflow execution

#### 2. **Switch1** (Input Type Router)
- Routes voice messages to transcription
- Routes text messages directly to processing

#### 3. **Transcribes Audio**
- Uses OpenAI Whisper API
- Converts voice to text

#### 4. **Merge Input**
- Normalizes input from all sources
- Handles scheduled vs. manual triggers

#### 5. **OpenAI Query Classifier**
- Uses GPT-4o-mini
- Determines if query is property-related
- Routes to appropriate handler

#### 6. **Claude API - Interpret Query**
- Uses Claude Sonnet 4
- Interprets property management intent
- Determines required API endpoints

#### 7. **HTTP Request Nodes** (Buildium API)
- `Get Query Results`: Fetches outstanding balances
- `Get Lease Details`: Retrieves lease information
- `Get Property Details`: Fetches property data

#### 8. **Merge & Group Nodes**
- Combines data from multiple API calls
- Groups transactions by tenant
- Aggregates late fees

#### 9. **Calculate Days Delinquent**
- Analyzes balance aging buckets
- Assigns priority levels

#### 10. **Format Report**
- Generates Telegram-formatted messages
- Handles pagination for long reports

#### 11. **Google Sheets Nodes**
- `Create Spread Sheet`: Creates new spreadsheet
- `Update Rows of Sheet`: Populates data
- `Move to Folder`: Organizes in Drive
- `Share File`: Sets permissions

#### 12. **Monday.com Nodes**
- `Prepare Monday.com Task Data`: Formats GraphQL mutation
- `HTTP Request`: Creates task
- `Prepare Monday Update with Link`: Adds spreadsheet link
- `Post Update to Monday.com`: Posts update to task

#### 13. **Response Nodes**
- `Send a text message`: Sends text to Telegram
- `Generate audio`: Creates TTS audio
- `Send an audio file`: Sends voice response

---

## 🐛 Troubleshooting

### Common Issues

#### 1. **No response from Telegram bot**
- Check Telegram credentials are active
- Verify bot token is correct
- Ensure workflow is activated

#### 2. **API authentication errors**
- Verify all API credentials are current
- Check credential IDs match in nodes
- Ensure API quotas are not exceeded

#### 3. **Missing tenant data (N/A values)**
- Verify Buildium API permissions
- Check lease has CurrentTenants data
- Ensure property information is complete

#### 4. **Google Sheets not creating**
- Verify OAuth scopes include Sheets & Drive
- Check folder ID is correct
- Ensure account has Drive access

#### 5. **Monday.com task creation fails**
- Verify board ID and column IDs
- Check person ID exists in Monday.com
- Ensure API token has write permissions

#### 6. **Schedule trigger not running**
- Check cron expression: `0 9 6 * *`
- Verify workflow is active
- Check n8n timezone settings

---

## 🔒 Security Considerations

1. **Credential Management**
   - Store all API keys in n8n credential store
   - Never commit credentials to version control
   - Rotate API keys regularly

2. **Google Drive Permissions**
   - Limit spreadsheet access to necessary users
   - Use folder-level permissions
   - Consider using service accounts

3. **Telegram Bot Security**
   - Use bot token securely
   - Restrict bot to authorized chat IDs
   - Monitor bot activity logs

4. **API Rate Limits**
   - Implement retry logic for rate-limited requests
   - Monitor API usage quotas
   - Add delays between API calls if needed

---

## 📈 Performance Optimization

### Current Batch Processing
- Buildium API calls: Batch size of 1 (sequential)
- Can be increased for better performance
- Balance between speed and API limits

### Caching Opportunities
- Property data rarely changes (consider caching)
- Tenant information updates infrequently
- Lease details can be cached short-term

### Pagination
- Outstanding balances: Limit 100 per request
- Implement pagination for larger datasets

---

## 🛠️ Customization

### Modify Report Frequency
Edit the Schedule Trigger cron expression:
```javascript
// Current: 6th of month at 9 AM
"0 9 6 * *"

// Examples:
"0 9 1 * *"  // 1st of month at 9 AM
"0 9 * * 1"  // Every Monday at 9 AM
"0 9 1,15 * *"  // 1st and 15th at 9 AM
```

### Change Priority Thresholds
Edit the `Calculate Days Delinquent` node:
```javascript
if (daysDelinquent >= 31) {
  priority = 'Urgent';
} else if (daysDelinquent >= 16) {
  priority = 'High';
} else if (daysDelinquent >= 6) {
  priority = 'Normal';
}
```

### Customize Report Format
Edit the `Format Report` node to modify:
- Emoji usage
- Text formatting
- Grouping logic
- Summary calculations

### Add Additional Data Fields
1. Modify `Merge All Data` node to include new fields
2. Update `Format For google Sheet Node` to add columns
3. Update report generation to display new data

---

## 📝 Maintenance

### Regular Tasks
- **Weekly**: Monitor API usage and quotas
- **Monthly**: Review and update credentials if needed
- **Quarterly**: Audit generated reports for accuracy
- **Annually**: Review and update AI prompts/models

### Logging
- Enable n8n execution logging
- Monitor failed executions
- Track API response times
- Review error patterns

---

## 🤝 Support & Contribution

### Getting Help
- Check n8n community forums
- Review API documentation:
  - [Buildium API Docs](https://developers.buildium.com/)
  - [OpenAI API Docs](https://platform.openai.com/docs)
  - [Anthropic API Docs](https://docs.anthropic.com/)
  - [Monday.com API Docs](https://developer.monday.com/)

### Workflow Version
- **Current Version**: Latest
- **n8n Compatibility**: n8n v1.0+
- **Last Updated**: December 2025

---

## 📄 License

This workflow is provided as-is for property management automation purposes. Ensure compliance with all API terms of service and data privacy regulations.

---

## 🎨 Workflow Visual Overview

The workflow uses color-coded sticky notes for organization:

- **White**: Input Nodes
- **Green**: Input Processing & Query Processing
- **Yellow**: Data Preparation & API Fetching
- **Purple**: Merging, Grouping & Sorting
- **Blue**: Google Sheets Creation & Processing
- **Pink**: Report Formation & Monday.com Tasks
- **Orange**: Telegram Response Handling

---

## ⚡ Quick Start Checklist

- [ ] Import `workflow.json` into n8n
- [ ] Configure Telegram Bot credentials
- [ ] Set up OpenAI API credentials
- [ ] Set up Anthropic (Claude) API credentials
- [ ] Configure Buildium API credentials
- [ ] Set up Google OAuth2 credentials
- [ ] Configure Monday.com API credentials
- [ ] Replace all placeholder IDs in workflow
- [ ] Update Google Drive Folder ID
- [ ] Update Monday.com Board ID and Person ID
- [ ] Update Telegram Chat ID for automated reports
- [ ] Test with a sample query
- [ ] Activate the workflow
- [ ] Verify scheduled trigger works

---

## 📞 Contact

For questions about this workflow, please refer to the n8n community or your organization's IT support team.

**Happy Automating! 🚀**
