# Agentic Chat Capability

This document outlines the design and implementation of the agentic chat capability for the Trip Planning Automation System.

## Overview

The agentic chat system is designed to provide automated, context-aware responses to customer inquiries across multiple channels (WhatsApp and email). It leverages natural language processing and a knowledge base of travel-related information to resolve customer queries without human intervention, while maintaining the ability to escalate to human operators when necessary.

## Architecture

The agentic chat capability consists of the following components:

1. **Message Receiver**: Captures incoming messages from various channels
2. **Context Builder**: Retrieves customer history and conversation context
3. **Intent Classifier**: Determines the customer's intent and query type
4. **Knowledge Base**: Stores structured travel information and FAQs
5. **Response Generator**: Creates personalized, contextually appropriate responses
6. **Human Handoff System**: Escalates complex queries to human operators
7. **Timeline Recorder**: Logs all interactions in the customer timeline

## Implementation in n8n

### 1. Webhook Setup

Create a webhook endpoint in n8n to receive incoming messages from:
- WhatsApp Business API
- Email reply parsing (via SendGrid or similar)

```json
{
  "name": "Incoming Message Webhook",
  "type": "n8n-nodes-base.webhook",
  "parameters": {
    "path": "/chat",
    "responseMode": "responseNode",
    "options": {}
  }
}
```

### 2. Context Building Workflow

This workflow retrieves the customer's history and current context:

```javascript
// Sample code for the Function node to build context
const customerId = $input.item.json.customerId;
const message = $input.item.json.message;

// Get customer details
const customerDetails = await fetchCustomerDetails(customerId);

// Get recent interactions (last 5)
const recentInteractions = await fetchRecentInteractions(customerId, 5);

// Build context object
return {
  customerId,
  message,
  customerName: customerDetails.firstName,
  interestStatus: customerDetails.interestStatus,
  stage: customerDetails.currentStage,
  recentInteractions,
  timestamp: new Date().toISOString()
};
```

### 3. Intent Classification

Use an AI service (OpenAI, Azure, etc.) to classify the customer's intent:

```json
{
  "name": "Classify Intent",
  "type": "n8n-nodes-base.openAi",
  "parameters": {
    "authentication": "predefinedCredentialType",
    "resource": "chat",
    "prompt": "=Classify the following customer message into one of these categories: PRICING_INQUIRY, ITINERARY_QUESTION, BOOKING_REQUEST, CANCELLATION, DOCUMENT_REQUEST, GENERAL_INFORMATION, OTHER.\n\nCustomer message: {{$json.message}}\n\nProvide your answer in JSON format with 'intent' and 'confidence' fields.",
    "model": "gpt-3.5-turbo",
    "options": {
      "temperature": 0.2
    }
  }
}
```

### 4. Knowledge Base Query

Based on the detected intent, query the appropriate knowledge base:

```javascript
// Sample code for querying knowledge base
const intent = $input.item.json.intent;
const message = $input.item.json.message;

let knowledgeBase;
switch (intent) {
  case 'PRICING_INQUIRY':
    knowledgeBase = 'pricing';
    break;
  case 'ITINERARY_QUESTION':
    knowledgeBase = 'itineraries';
    break;
  case 'BOOKING_REQUEST':
    knowledgeBase = 'booking';
    break;
  // ... other intents
  default:
    knowledgeBase = 'general';
}

// Query the knowledge base (this could be a vector DB, Google Sheets, etc.)
const relevantInfo = await queryKnowledgeBase(knowledgeBase, message);

return {
  ...$input.item.json,
  relevantInfo
};
```

### 5. Response Generation

Generate a tailored response using LLM:

```json
{
  "name": "Generate Response",
  "type": "n8n-nodes-base.openAi",
  "parameters": {
    "authentication": "predefinedCredentialType",
    "resource": "chat",
    "prompt": "=You are a helpful travel assistant. Use the following information to answer the customer's question. Be friendly, concise, and helpful.\n\nCustomer name: {{$json.customerName}}\nCustomer's question: {{$json.message}}\nRelevant information: {{$json.relevantInfo}}\n\nIf you absolutely cannot answer based on the information provided, apologize and offer to connect them with a human agent. Your response should be in a conversational tone.",
    "model": "gpt-4",
    "options": {
      "temperature": 0.7
    }
  }
}
```

### 6. Human Handoff Logic

Implement logic to determine when to escalate to a human:

```javascript
// Sample code for human handoff decision
const confidence = $input.item.json.confidence;
const intent = $input.item.json.intent;
const sensitiveKeywords = ['urgent', 'angry', 'refund', 'cancel', 'unhappy'];

// Check if message contains sensitive keywords
const containsSensitiveKeywords = sensitiveKeywords.some(keyword => 
  $input.item.json.message.toLowerCase().includes(keyword)
);

// Determine if human handoff is needed
const needsHumanHandoff = 
  confidence < 0.7 || 
  containsSensitiveKeywords || 
  intent === 'CANCELLATION' || 
  intent === 'OTHER';

return {
  ...$input.item.json,
  needsHumanHandoff
};
```

### 7. Response Delivery

Send the response to the appropriate channel:

```json
{
  "name": "Route Response",
  "type": "n8n-nodes-base.switch",
  "parameters": {
    "dataType": "string",
    "value1": "={{ $json.channel }}",
    "rules": {
      "rules": [
        {
          "value2": "whatsapp",
          "outputIndex": 0
        },
        {
          "value2": "email",
          "outputIndex": 1
        }
      ]
    }
  }
}
```

## Timeline Integration

All chat interactions are recorded in the customer timeline:

```json
{
  "name": "Update Timeline",
  "type": "n8n-nodes-base.httpRequest",
  "parameters": {
    "url": "https://sheets.googleapis.com/v4/spreadsheets/{{ $env.GOOGLE_SHEET_ID }}/values/CustomerTimeline:append",
    "method": "POST",
    "authentication": "oAuth2",
    "sendBody": true,
    "bodyParameters": {
      "values": [
        [
          "={{ $json.customerId }}",
          "={{ $now }}",
          "=Chat Interaction: {{ $json.intent }}",
          "={{ $json.channel }}",
          "=Q: {{ $json.message.substring(0, 100) }}{{ $json.message.length > 100 ? '...' : '' }} A: {{ $json.response.substring(0, 100) }}{{ $json.response.length > 100 ? '...' : '' }}",
          "=AI Agent{{ $json.needsHumanHandoff ? ' (Escalated)' : '' }}"
        ]
      ]
    },
    "queryParametersUi": {
      "parameter": [
        {
          "name": "valueInputOption",
          "value": "USER_ENTERED"
        }
      ]
    }
  }
}
```

## Knowledge Base Setup

The knowledge base for the agentic chat should be structured as follows:

### 1. Basic Travel Information

- Destination details
- Visa requirements
- Weather information
- Local attractions
- Travel advisories
- Dietary preferences (vegetarian/non-vegetarian options)
- Food specialties and restrictions at destinations

### 2. Package-Specific Information

- Available travel packages
- Itinerary details
- Pricing information
- Included services
- Optional add-ons

### 3. Booking and Administrative Information

- Payment methods
- Cancellation policies
- Booking modification procedures
- Insurance options
- COVID-19 policies

### 4. Frequently Asked Questions

- General travel FAQs
- Package-specific FAQs
- Pre-departure FAQs
- Post-booking FAQs

## Implementation Phases

### Phase 1: Basic Q&A

- Implement simple intent classification
- Build basic knowledge base with FAQ responses
- Set up channel-specific response formatting
- Implement human handoff for uncertain queries

### Phase 2: Context-Aware Responses

- Incorporate customer history into responses
- Add personalization based on customer preferences
- Implement sentiment analysis for response tone adjustment
- Enable multi-turn conversations with context retention

### Phase 3: Proactive Assistance

- Add proactive recommendations based on customer profile
- Implement predictive responses to common follow-up questions
- Create automated follow-up sequences based on conversation outcomes
- Develop adaptive conversation flows for complex inquiries

## Monitoring and Improvement

To continuously improve the agentic chat system:

1. **Performance Metrics**: Track resolution rate, handoff rate, and customer satisfaction
2. **Conversation Logs**: Regularly review chat logs to identify improvement areas
3. **Knowledge Base Updates**: Continuously update the knowledge base with new information
4. **Model Retraining**: Periodically retrain the NLP models with new conversation data

## Human Operator Interface

The system includes an interface for human operators to:

1. View customer information and conversation history
2. Take over conversations from the AI when needed
3. Add notes to customer records
4. Update the knowledge base with new information
5. Flag conversations for review and model improvement

## Security and Privacy Considerations

The agentic chat system implements the following security measures:

1. **Data Encryption**: All messages are encrypted in transit and at rest
2. **PII Handling**: Personal identifiable information is redacted from logs
3. **Access Control**: Strict access controls for human operators
4. **Consent Management**: Clear customer consent for automated processing
5. **Retention Policies**: Data retention policies compliant with regulations 