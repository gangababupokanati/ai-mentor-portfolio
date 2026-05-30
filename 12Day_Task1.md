# 🎓 Aditya University — AI Boot Camp 2026
# Task 1: AI Tool Automation using n8n

---

## 📌 Problem Statement
Design and implement an AI-powered automation workflow using n8n that generates educational content from a given topic and stores the output in a structured format.

---

## 🔄 Workflow
```
Manual Trigger → Set (Topic Input) → HTTP Request (Groq AI) → Code (Formatter) → Google Sheets
```

---

## 🛠️ Tools Used
- **n8n** — Workflow automation
- **Groq API** — LLaMA 3.3 70B model
- **Google Sheets** — Data storage
- **Docker** — Running n8n locally

---

## ⚙️ Step 1: Docker Setup

### n8n_docker-compose.yml
```yaml
version: "3.8"

services:
  n8n:
    image: n8nio/n8n
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=false
      - GENERIC_TIMEZONE=Asia/Kolkata
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  n8n_data:
```

### Run Command
```bash
docker compose -f n8n_docker-compose.yml up -d
```
Then open: `http://localhost:5678`

---

## ⚙️ Step 2: Node 1 — Manual Trigger
- Type: `Manual Trigger`
- No configuration needed
- This starts the workflow manually

---

## ⚙️ Step 3: Node 2 — Set Node (Topic Input)

| Setting | Value |
|---|---|
| Field Name | `topic` |
| Field Type | `String` |
| Field Value | `Prompt Engineering` |

---

## ⚙️ Step 4: Node 3 — HTTP Request (Groq AI)

### Settings
| Setting | Value |
|---|---|
| Method | `POST` |
| URL | `https://api.groq.com/openai/v1/chat/completions` |
| Body Type | `JSON` |

### Headers
| Name | Value |
|---|---|
| `Authorization` | `Bearer gsk_YOUR_GROQ_API_KEY` |
| `Content-Type` | `application/json` |

### Body (JSON)
```json
{
  "model": "llama-3.3-70b-versatile",
  "temperature": 0.7,
  "messages": [
    {
      "role": "system",
      "content": "You are an educational content generator. Always respond with valid JSON only. No extra text, no markdown, no code blocks."
    },
    {
      "role": "user",
      "content": "Generate educational content for the topic: Prompt Engineering.\n\nReturn ONLY this JSON structure:\n{\n  \"learning_objectives\": [\n    \"objective 1\",\n    \"objective 2\",\n    \"objective 3\"\n  ],\n  \"key_concepts\": [\n    \"concept 1\",\n    \"concept 2\",\n    \"concept 3\",\n    \"concept 4\",\n    \"concept 5\"\n  ],\n  \"mcqs\": [\n    {\n      \"question\": \"Question 1?\",\n      \"options\": [\"A. option\", \"B. option\", \"C. option\", \"D. option\"],\n      \"answer\": \"A\"\n    },\n    {\n      \"question\": \"Question 2?\",\n      \"options\": [\"A. option\", \"B. option\", \"C. option\", \"D. option\"],\n      \"answer\": \"B\"\n    },\n    {\n      \"question\": \"Question 3?\",\n      \"options\": [\"A. option\", \"B. option\", \"C. option\", \"D. option\"],\n      \"answer\": \"C\"\n    },\n    {\n      \"question\": \"Question 4?\",\n      \"options\": [\"A. option\", \"B. option\", \"C. option\", \"D. option\"],\n      \"answer\": \"D\"\n    },\n    {\n      \"question\": \"Question 5?\",\n      \"options\": [\"A. option\", \"B. option\", \"C. option\", \"D. option\"],\n      \"answer\": \"A\"\n    }\n  ],\n  \"summary\": \"A short 2-3 sentence summary of the topic.\"\n}"
    }
  ]
}
```

---

## ⚙️ Step 5: Node 4 — Code Node (Output Formatter)

```javascript
// Get Groq response
const response = $input.first().json;

// Extract text
const text = response.choices[0].message.content;

// Clean and parse JSON
let cleaned = text
  .replace(/```json/g, '')
  .replace(/```/g, '')
  .trim();

// Fix common JSON issues
cleaned = cleaned
  .replace(/[\u0000-\u001F\u007F-\u009F]/g, '')
  .replace(/,\s*}/g, '}')
  .replace(/,\s*]/g, ']');

// Parse JSON safely
let parsed;
try {
  parsed = JSON.parse(cleaned);
} catch(e) {
  return [{
    json: {
      error: e.message,
      raw_text: cleaned.substring(0, 500)
    }
  }];
}

// Return structured output
return [{
  json: {
    topic: "Prompt Engineering",
    learning_objectives: parsed.learning_objectives,
    key_concepts: parsed.key_concepts,
    mcqs: parsed.mcqs,
    summary: parsed.summary,
    generated_at: new Date().toISOString()
  }
}];
```

---

## ⚙️ Step 6: Node 5 — Google Sheets (Data Storage)

### Google Sheet Headers (Row 1)
| A | B | C |
|---|---|---|
| topic | summary | generated_at |

### Node Settings
| Setting | Value |
|---|---|
| Resource | `Sheet Within Document` |
| Operation | `Append Row` |
| Document | Your Google Sheet URL |
| Sheet | `Sheet1` |

### Column Mapping
| Field | Column |
|---|---|
| `={{ $json.topic }}` | A |
| `={{ $json.summary }}` | B |
| `={{ $json.generated_at }}` | C |

---

## 📤 Exported Workflow JSON

```json
{
  "name": "AI Educational Content Generator",
  "nodes": [
    {
      "parameters": {},
      "name": "Manual Trigger",
      "type": "n8n-nodes-base.manualTrigger",
      "typeVersion": 1,
      "position": [100, 300]
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "name": "topic",
              "value": "Prompt Engineering",
              "type": "string"
            }
          ]
        }
      },
      "name": "Set Topic",
      "type": "n8n-nodes-base.set",
      "typeVersion": 3,
      "position": [300, 300]
    },
    {
      "parameters": {
        "method": "POST",
        "url": "https://api.groq.com/openai/v1/chat/completions",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "Authorization",
              "value": "Bearer gsk_YOUR_GROQ_API_KEY"
            },
            {
              "name": "Content-Type",
              "value": "application/json"
            }
          ]
        },
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "{\"model\":\"llama-3.3-70b-versatile\",\"temperature\":0.7,\"messages\":[{\"role\":\"system\",\"content\":\"You are an educational content generator. Always respond with valid JSON only.\"},{\"role\":\"user\",\"content\":\"Generate educational content for Prompt Engineering in JSON with learning_objectives, key_concepts, mcqs, summary.\"}]}"
      },
      "name": "Groq AI Generator",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4,
      "position": [500, 300]
    },
    {
      "parameters": {
        "language": "javaScript",
        "jsCode": "const response = $input.first().json;\nconst text = response.choices[0].message.content;\nlet cleaned = text.replace(/```json/g,'').replace(/```/g,'').trim();\ncleaned = cleaned.replace(/[\\u0000-\\u001F]/g,'').replace(/,\\s*}/g,'}').replace(/,\\s*]/g,']');\nlet parsed;\ntry { parsed = JSON.parse(cleaned); } catch(e) { return [{json:{error:e.message}}]; }\nreturn [{json:{topic:'Prompt Engineering',learning_objectives:parsed.learning_objectives,key_concepts:parsed.key_concepts,mcqs:parsed.mcqs,summary:parsed.summary,generated_at:new Date().toISOString()}}];"
      },
      "name": "Output Formatter",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [700, 300]
    },
    {
      "parameters": {
        "operation": "append",
        "documentId": "YOUR_GOOGLE_SHEET_ID",
        "sheetName": "Sheet1",
        "columns": {
          "mappingMode": "defineBelow",
          "value": {
            "topic": "={{ $json.topic }}",
            "summary": "={{ $json.summary }}",
            "generated_at": "={{ $json.generated_at }}"
          }
        }
      },
      "name": "Google Sheets",
      "type": "n8n-nodes-base.googleSheets",
      "typeVersion": 4,
      "position": [900, 300]
    }
  ],
  "connections": {
    "Manual Trigger": {
      "main": [[{ "node": "Set Topic", "type": "main", "index": 0 }]]
    },
    "Set Topic": {
      "main": [[{ "node": "Groq AI Generator", "type": "main", "index": 0 }]]
    },
    "Groq AI Generator": {
      "main": [[{ "node": "Output Formatter", "type": "main", "index": 0 }]]
    },
    "Output Formatter": {
      "main": [[{ "node": "Google Sheets", "type": "main", "index": 0 }]]
    }
  }
}
```

---

## 📝 Brief Explanation
This n8n workflow automates educational content generation using Groq LLaMA AI. A Manual Trigger starts the workflow, and a Set node defines the input topic as "Prompt Engineering." An HTTP Request node calls the Groq API using the llama-3.3-70b-versatile model to generate learning objectives, key concepts, MCQs, and a summary in JSON format. A Code node parses and structures the response, which is then stored in Google Sheets for easy access and review.

---

## 🔑 API Key
Get your free Groq API key from: https://console.groq.com/keys
