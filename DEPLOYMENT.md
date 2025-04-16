# Deployment Guide: Trip Planning Automation System

This guide outlines the steps to deploy the Trip Planning Automation System using n8n and the associated integrations. Please follow these steps in sequence to ensure a successful deployment.

## Prerequisites

1. Server with at least 4GB RAM, 2 CPU cores
2. Node.js 14+ installed
3. WhatsApp Business API access
4. Google Workspace account with API access
5. SendGrid or other email service account
6. Document storage solution (Google Drive, S3, etc.)

## Step 1: n8n Installation

### Self-hosted (Recommended for production)

```bash
# Install n8n globally
npm install n8n -g

# Create a dedicated directory
mkdir -p /opt/n8n-data
cd /opt/n8n-data

# Copy the .env file
cp /path/to/project/.env.example .env

# Edit the .env file with your actual credentials
nano .env

# Start n8n as a service
n8n start --tunnel
```

### Docker Installation (Alternative)

```bash
# Pull the n8n Docker image
docker pull n8nio/n8n

# Create a volume for persistent data
docker volume create n8n_data

# Start n8n container
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  -v $(pwd)/.env:/home/node/.n8n/.env \
  n8nio/n8n
```

## Step 2: Google Sheets Setup

1. Create a new Google Sheet with the following tabs:
   - `CustomerContacts` (Main customer database)
   - `CustomerTimeline` (Timeline of customer interactions)

2. Set up the column headers for `CustomerContacts`:
   - Customer ID
   - First Name
   - Last Name
   - Email
   - Phone Number
   - Contact Channel
   - Interest Status
   - Notes
   - Last Contact Date
   - Current Stage

3. Set up the column headers for `CustomerTimeline`:
   - Customer ID
   - Timestamp
   - Action/Event
   - Channel
   - Details
   - Triggered By

4. Share the Google Sheet with your service account email (from the `.env` file)

## Step 3: Document Preparation

1. Prepare the 5 required documents:
   - Travel Brochure (PDF)
   - Sample Itinerary (PDF)
   - Pricing Details (PDF)
   - FAQ (PDF)
   - Terms & Conditions (PDF)

2. Upload these documents to your chosen storage solution

3. Update the `DOCUMENT_PATH` in your `.env` file to point to the directory containing these documents

## Step 4: WhatsApp Business API Setup

1. Register for WhatsApp Business API access
2. Set up a verified business profile
3. Create message templates for initial outreach and follow-ups
4. Configure webhooks to send incoming messages to your n8n webhook endpoint
5. Generate an API token and update it in your `.env` file

## Step 5: Email Service Configuration

1. Set up an account with SendGrid or your preferred email provider
2. Configure domain authentication and sender verification
3. Create email templates for initial outreach and document delivery
4. Set up webhooks for email tracking (opens, clicks)
5. Generate an API key and update it in your `.env` file

## Step 6: n8n Workflow Import and Configuration

1. Access your n8n instance at `http://your-server:5678`
2. Import the workflow JSON files from the `workflows` directory:
   - `initial_email_campaign.json`
   - `initial_whatsapp_campaign.json`
   - `response_handling.json`
   - `document_delivery.json`

3. For each workflow:
   - Check and update all credentials
   - Verify all environment variables are properly set
   - Activate the workflow

4. Test each workflow with a small sample (10-20 contacts) before full deployment

## Step 7: Webhook Configuration

1. Identify the webhook URL from your n8n `response_handling` workflow
2. Configure this URL in your WhatsApp Business API settings
3. Configure this URL in your email service webhook settings
4. Test the webhooks by sending test messages

## Step 8: Monitoring Setup

1. Set up Slack notifications for system alerts
2. Configure monitoring for API rate limits
3. Set up daily health check workflows
4. Establish a backup routine for the Google Sheets data

## Step 9: Scaling Considerations

For handling 50,000+ contacts:

1. Implement batch processing (already configured in workflows)
2. Set up proper rate limiting to comply with API limits
3. Consider dedicated server resources for n8n
4. Monitor system performance during initial rollout
5. Have fallback mechanisms for all critical operations

## Step 10: Testing and Validation

Before full deployment:

1. Perform end-to-end testing with 100-500 test contacts
2. Validate all data is correctly recorded in Google Sheets
3. Confirm all webhooks are functioning properly
4. Verify document delivery works across email and WhatsApp
5. Test error handling by simulating failures

## Troubleshooting

- **Workflow errors**: Check the n8n execution logs
- **API rate limits**: Adjust batch sizes and intervals in the workflows
- **Missing data**: Verify Google Sheets permissions and service account setup
- **Webhook failures**: Check network connectivity and firewall settings

## Maintenance

Regular maintenance tasks:

1. Monitor n8n logs daily
2. Review Google Sheets data for consistency
3. Update document content as needed
4. Adjust rate limits based on observed performance
5. Update API keys and credentials periodically for security

## Support Resources

- n8n Documentation: [https://docs.n8n.io/](https://docs.n8n.io/)
- WhatsApp Business API Documentation: [https://developers.facebook.com/docs/whatsapp/](https://developers.facebook.com/docs/whatsapp/)
- Google Sheets API Documentation: [https://developers.google.com/sheets/api](https://developers.google.com/sheets/api)
- SendGrid Documentation: [https://sendgrid.com/docs/](https://sendgrid.com/docs/) 