# Google Sheets Setup Guide

This document provides detailed instructions for setting up the Google Sheets required for the Trip Planning Automation System.

## Overview

The system uses Google Sheets as the primary data storage solution for:
1. Customer contact information
2. Customer interaction timeline
3. Campaign tracking and analytics

## Sheet 1: CustomerContacts

This is the main database of all customer contacts and their current status.

### Structure

| Column Name | Data Type | Description | Example |
|-------------|-----------|-------------|---------|
| Customer ID | Text | Unique identifier for each customer | CUS-12345 |
| First Name | Text | Customer's first name | John |
| Last Name | Text | Customer's last name | Smith |
| Email | Text | Customer's email address | john.smith@example.com |
| Phone Number | Text | Customer's phone number with country code | +1234567890 |
| Contact Channel | Text | Primary channel of communication | WhatsApp/Email/Call |
| Interest Status | Text | Customer's response to initial contact | Yes/No/Maybe |
| Notes | Text | Additional information about the customer | Interested in family packages |
| Last Contact Date | DateTime | When the customer was last contacted | 2023-07-01T15:30:00Z |
| Current Stage | Text | Current stage in the customer journey | Initial Contact/Documents Sent/Follow-up Required |

### Setup Instructions

1. Create a new Google Sheet
2. Rename the first tab to "CustomerContacts"
3. Add the column headers as described above
4. Apply data validation for Interest Status (Yes/No/Maybe)
5. Format the Last Contact Date column as Date & Time
6. Freeze the first row for easier viewing

## Sheet 2: CustomerTimeline

This sheet tracks all interactions with customers in chronological order.

### Structure

| Column Name | Data Type | Description | Example |
|-------------|-----------|-------------|---------|
| Customer ID | Text | Unique identifier for each customer | CUS-12345 |
| Timestamp | DateTime | When the interaction occurred | 2023-07-01T15:30:00Z |
| Action/Event | Text | Type of interaction | Email Sent/WhatsApp Message/Document Delivery |
| Channel | Text | Channel used for interaction | WhatsApp/Email/Call |
| Details | Text | Description of the interaction | Customer replied "Yes" to initial contact |
| Triggered By | Text | Who/what initiated the action | System/Operator Name |

### Setup Instructions

1. Create a new tab in the same Google Sheet
2. Name the tab "CustomerTimeline"
3. Add the column headers as described above
4. Format the Timestamp column as Date & Time
5. Sort the default view by Timestamp (newest first)
6. Freeze the first row for easier viewing

## Sheet 3: CampaignAnalytics (Optional)

This sheet tracks overall campaign performance.

### Structure

| Column Name | Data Type | Description | Example |
|-------------|-----------|-------------|---------|
| Date | Date | Date of the analytics record | 2023-07-01 |
| Campaign Name | Text | Name of the campaign | Summer Travel 2023 |
| Total Sent | Number | Number of messages sent | 5000 |
| Email Sent | Number | Number of emails sent | 3000 |
| WhatsApp Sent | Number | Number of WhatsApp messages sent | 2000 |
| Responses | Number | Total number of responses received | 1500 |
| Yes Responses | Number | Number of "Yes" responses | 750 |
| No Responses | Number | Number of "No" responses | 500 |
| Maybe Responses | Number | Number of "Maybe" responses | 250 |
| Documents Delivered | Number | Number of document sets delivered | 750 |
| Conversion Rate | Percentage | Percentage of positive responses | 15% |

### Setup Instructions

1. Create a new tab in the same Google Sheet
2. Name the tab "CampaignAnalytics"
3. Add the column headers as described above
4. Format the Date column as Date
5. Format the Conversion Rate column as Percentage
6. Add conditional formatting to highlight high/low conversion rates

## Access Control Setup

To securely share the Google Sheet with your n8n service:

1. Create a Google Cloud Service Account
2. Grant this service account Edit permissions on your Google Sheet
3. Generate and download a JSON key for this service account
4. Copy the relevant details into your `.env` file
5. Never share the service account key in public repositories

## Data Protection Considerations

- Use cell protection to prevent accidental edits to critical columns
- Set up daily backups of the sheet (Google Sheets → File → Version history)
- Consider adding data validation to prevent incorrect data entry
- For GDPR compliance, add a sheet for tracking consent and data deletion requests

## Query Formulas

Add these useful formulas to your sheet for quick insights:

### Count of customers by Interest Status
```
=COUNTIF(CustomerContacts!G:G,"Yes")
=COUNTIF(CustomerContacts!G:G,"No")
=COUNTIF(CustomerContacts!G:G,"Maybe")
```

### Count of interactions by channel
```
=COUNTIF(CustomerTimeline!D:D,"WhatsApp")
=COUNTIF(CustomerTimeline!D:D,"Email")
=COUNTIF(CustomerTimeline!D:D,"Call")
```

### Latest interaction for a specific customer
```
=MAXIFS(CustomerTimeline!B:B,CustomerTimeline!A:A,"CUS-12345")
```

## Automation Tips

- Use Google Sheets' conditional formatting to highlight important status changes
- Create views with filters for each operator to focus on their assigned customers
- Set up email notifications for important changes using Google Sheets' built-in notification rules
- Consider using Google Apps Script for additional automation beyond what n8n provides 