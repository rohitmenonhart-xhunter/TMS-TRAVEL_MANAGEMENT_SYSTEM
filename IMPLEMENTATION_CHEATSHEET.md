# Implementation Cheatsheet

This document provides quick reference code snippets, credential setup instructions, and common operations for implementing the Trip Planning Automation System with n8n.

## Credentials Setup

### Google Sheets

```
CREDENTIAL TYPE: OAuth2
CLIENT ID: [Your Google Cloud Client ID]
CLIENT SECRET: [Your Google Cloud Client Secret]
AUTHORIZATION URL: https://accounts.google.com/o/oauth2/auth
TOKEN URL: https://oauth2.googleapis.com/token
SCOPE: https://www.googleapis.com/auth/spreadsheets https://www.googleapis.com/auth/drive
```

### WhatsApp Business API

```
CREDENTIAL TYPE: Generic Credential Type
API TOKEN: [Your WhatsApp Business API Token]
BASE URL: https://graph.facebook.com/v17.0
```

### Email Service (SendGrid)

```
CREDENTIAL TYPE: SendGrid API
API KEY: [Your SendGrid API Key]
```

### OpenAI API

```
CREDENTIAL TYPE: OpenAI API
API KEY: [Your OpenAI API Key]
```

## Common n8n Function Code Snippets

### 1. Parsing Customer Data from Google Sheets

```javascript
// Function to find and process a customer from Sheet data
const findCustomer = (sheetData, customerId) => {
  // Extract headers from first row
  const headers = sheetData[0];
  
  // Find column indexes
  const idIndex = headers.indexOf('Customer ID');
  const nameIndex = headers.indexOf('First Name');
  const emailIndex = headers.indexOf('Email');
  const phoneIndex = headers.indexOf('Phone Number');
  
  // Find the customer row
  const customerRow = sheetData.slice(1).find(row => row[idIndex] === customerId);
  
  if (!customerRow) {
    return null;
  }
  
  return {
    id: customerRow[idIndex],
    name: customerRow[nameIndex],
    email: customerRow[emailIndex],
    phone: customerRow[phoneIndex],
    // Add other fields as needed
  };
};

// Example usage
const customerId = $input.item.json.customerId;
const sheetData = $node["Google Sheets"].json.values;
const customer = findCustomer(sheetData, customerId);

return { customer };
```

### 2. Classifying Customer Responses

```javascript
// Function to classify a customer response
const classifyResponse = (text) => {
  const normalizedText = text.toLowerCase().trim();
  
  // Yes patterns
  if (/\b(yes|yeah|yep|sure|definitely|correct|ok|okay|absolutely|confirm)\b/.test(normalizedText)) {
    return 'yes';
  }
  
  // No patterns
  if (/\b(no|nope|not|don't|cannot|never|stop|unsubscribe)\b/.test(normalizedText)) {
    return 'no';
  }
  
  // Maybe patterns
  if (/\b(maybe|perhaps|possibly|consider|thinking|later|sometime|might|more info|details)\b/.test(normalizedText)) {
    return 'maybe';
  }
  
  // Default to unknown if no clear match
  return 'unknown';
};

// Example usage
const messageText = $input.item.json.message;
const classification = classifyResponse(messageText);

return {
  originalMessage: messageText,
  classification: classification,
  timestamp: new Date().toISOString()
};
```

### 3. Formatting Document Data for Email Delivery

```javascript
// Prepare documents for email attachment
const prepareDocumentsForEmail = (basePath, documentList) => {
  return documentList.map(doc => ({
    name: doc.filename,
    path: `${basePath}/${doc.filename}`,
    contentType: 'application/pdf'
  }));
};

// Example usage
const documents = [
  { filename: 'brochure.pdf', title: 'Travel Brochure' },
  { filename: 'pricing.pdf', title: 'Pricing Details' },
  { filename: 'itinerary.pdf', title: 'Sample Itinerary' },
  { filename: 'faq.pdf', title: 'Frequently Asked Questions' },
  { filename: 'terms.pdf', title: 'Terms and Conditions' }
];

const basePath = '/data/documents';
const emailAttachments = prepareDocumentsForEmail(basePath, documents);

return { emailAttachments };
```

### 4. Generating Tracking IDs for Messages

```javascript
// Generate a unique tracking ID for a message
const generateTrackingId = (prefix, customerId) => {
  const timestamp = Date.now();
  const random = Math.floor(Math.random() * 1000).toString().padStart(3, '0');
  return `${prefix}-${customerId}-${timestamp}-${random}`;
};

// Example usage
const customerId = $input.item.json.customerId;
const emailTrackingId = generateTrackingId('EMAIL', customerId);
const whatsappTrackingId = generateTrackingId('WA', customerId);

return {
  customerId,
  emailTrackingId,
  whatsappTrackingId,
  timestamp: new Date().toISOString()
};
```

### 5. Formatting Messages with Customer Data

```javascript
// Format a message template with customer data
const formatMessage = (template, customerData) => {
  let formatted = template;
  
  // Replace all placeholders with actual values
  Object.keys(customerData).forEach(key => {
    const placeholder = new RegExp(`\\{\\{${key}\\}\\}`, 'g');
    formatted = formatted.replace(placeholder, customerData[key] || '');
  });
  
  return formatted;
};

// Example usage
const template = "Hello {{firstName}},\n\nThank you for your interest in our {{packageName}} package. Are you interested in learning more?\n\nBest regards,\nThe Travel Team";

const customerData = {
  firstName: $input.item.json.firstName,
  packageName: 'European Adventure'
};

const formattedMessage = formatMessage(template, customerData);

return { formattedMessage };
```

## Common HTTP Requests

### 1. Append Data to Google Sheet

```
METHOD: POST
URL: https://sheets.googleapis.com/v4/spreadsheets/{{SHEET_ID}}/values/{{SHEET_NAME}}:append
QUERY PARAMETERS:
  valueInputOption: USER_ENTERED
BODY:
{
  "values": [
    ["{{customerId}}", "{{firstName}}", "{{lastName}}", "{{email}}", "{{phone}}", "{{channel}}", "{{status}}", "{{notes}}", "{{timestamp}}", "{{stage}}"]
  ]
}
```

### 2. Send WhatsApp Message

```
METHOD: POST
URL: https://graph.facebook.com/v17.0/{{PHONE_NUMBER_ID}}/messages
HEADERS:
  Authorization: Bearer {{API_TOKEN}}
  Content-Type: application/json
BODY:
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "{{recipientPhone}}",
  "type": "text",
  "text": {
    "body": "{{messageBody}}"
  }
}
```

### 3. Send WhatsApp Document

```
METHOD: POST
URL: https://graph.facebook.com/v17.0/{{PHONE_NUMBER_ID}}/messages
HEADERS:
  Authorization: Bearer {{API_TOKEN}}
  Content-Type: application/json
BODY:
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "{{recipientPhone}}",
  "type": "document",
  "document": {
    "link": "{{documentUrl}}",
    "caption": "{{documentCaption}}"
  }
}
```

## Workflow Testing Tips

### Testing Initial Outreach

1. Create a small test sheet with 5-10 test contacts (use your own contact details)
2. Run the initial outreach workflow with a small batch size (1-2)
3. Verify emails/WhatsApp messages are received properly
4. Check that tracking links work
5. Verify data is correctly updated in Google Sheets

### Testing Response Processing

1. Prepare test responses covering all scenarios:
   - Clear "Yes" responses
   - Clear "No" responses
   - Various "Maybe" responses
   - Ambiguous/unclear responses
2. Send these responses to the webhook endpoints
3. Verify classification is working correctly
4. Check that the right follow-up workflows are triggered
5. Verify timeline updates are recorded correctly

### Testing Document Delivery

1. Ensure all test documents are properly uploaded to storage
2. Test document delivery via email
   - Check attachments are included
   - Verify personalization is working
3. Test document delivery via WhatsApp
   - Verify sequential sending works
   - Check documents are receivable on mobile
4. Verify delivery status is recorded in Google Sheets

## Debugging Common Issues

### Webhook Issues

- Verify webhook URL is correct and accessible
- Check webhook security settings
- Ensure proper error handling in the webhook node
- Test webhook using a tool like Postman

### Google Sheets Issues

- Verify Google API credentials are correctly set up
- Check sheet name and range in API requests
- Ensure proper permissions are set on the Google Sheet
- Verify sheet structure matches expected format

### Rate Limiting Issues

- Monitor API usage for WhatsApp Business API
- Implement proper batch sizes and delays
- Set up retry mechanisms for failed requests
- Add circuit breakers to prevent API bans

### Data Processing Issues

- Add comprehensive logging at each step
- Use n8n's debug feature to examine data at each node
- Implement error catching in function nodes
- Set up error notification workflow for critical failures

## Optimization Tips

1. Use batch processing for all bulk operations
2. Implement proper caching for frequently accessed data
3. Optimize Google Sheets reads by using specific ranges
4. Use webhook responses to immediately acknowledge receipt
5. Implement proper indexing in Google Sheets for lookups
6. Use environment variables for all configurable settings
7. Set up comprehensive error handling at every step
8. Implement idempotent operations to prevent duplication 