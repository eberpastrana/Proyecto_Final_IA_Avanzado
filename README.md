
# AI Clinical Workflow

This project implements an automated clinical workflow using event-driven architecture, integrating AI-based analysis, human validation (HITL), and monitoring capabilities.

## Overview

The system processes clinical cases received via webhook, analyzes them using AI logic, evaluates confidence, and decides whether to auto-approve or escalate for human review.

## Architecture

- Webhook (event intake)
- n8n (workflow orchestration)
- AI analysis module (simulated)
- Decision engine (confidence-based)
- Human-in-the-Loop validation
- Logging and monitoring

## Workflow

1. Receive clinical case via webhook
2. Normalize input data
3. Perform AI-based analysis
4. Assign priority and confidence score
5. Log classification results
6. Wait for potential human intervention
7. Decide:
   - Low confidence → Human Review
   - High confidence → Auto Approval
8. Return structured response

## Technologies Used

- n8n
- Docker
- Node-based workflows
- REST APIs
- JSON-based processing

## Running the Project

```bash
docker-compose up

REPO

ai-clinical-workflow

JSON

{
  "name": "AI Clinical Workflow - Final Project",
  "nodes": [
    {
      "parameters": {
        "path": "clinical-case",
        "responseMode": "responseNode"
      },
      "name": "Webhook - Intake",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 2,
      "position": [200, 300]
    },
    {
      "parameters": {
        "jsCode": "const input = $input.first().json;\nreturn [{ json: {\n  case_id: input.case_id || 'CASE-' + Date.now(),\n  description: input.description || 'Paciente con síntomas leves',\n  patient: input.patient || 'N/A',\n  created_at: new Date().toISOString()\n}}];"
      },
      "name": "Normalize Input",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [400, 300]
    },
    {
      "parameters": {
        "jsCode": "// Simulación IA + lógica tipo RAG\nconst text = $json.description.toLowerCase();\n\nlet priority = 'media';\nif (text.includes('dolor pecho') || text.includes('alta')) priority = 'alta';\nif (text.includes('leve')) priority = 'baja';\n\nconst timeouts = { alta: 300, media: 900, baja: 1800 };\n\nreturn [{ json: {\n  ...$json,\n  priority,\n  confidence: Math.round((Math.random() * (0.95 - 0.7) + 0.7) * 100) / 100,\n  timeout_sec: timeouts[priority],\n  rag_sources: ['Clinical Guidelines 2024', 'Internal Protocol A'],\n  analyzed_at: new Date().toISOString()\n}}];"
      },
      "name": "AI Analysis (Simulated)",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [600, 300]
    },
    {
      "parameters": {
        "jsCode": "console.log(`[CLASSIFICATION] Case ${$json.case_id} → Priority: ${$json.priority} | Confidence: ${$json.confidence}`);\nreturn [{ json: $json }];"
      },
      "name": "Log - Classification",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [800, 300]
    },
    {
      "parameters": {
        "amount": "={{ $json.timeout_sec }}",
        "unit": "seconds"
      },
      "name": "Wait - Review Window",
      "type": "n8n-nodes-base.wait",
      "typeVersion": 1,
      "position": [1000, 300]
    },
    {
      "parameters": {
        "conditions": {
          "number": [
            {
              "value1": "={{ $json.confidence }}",
              "operation": "smaller",
              "value2": 0.8
            }
          ]
        }
      },
      "name": "Check Confidence (HITL)",
      "type": "n8n-nodes-base.if",
      "typeVersion": 2,
      "position": [1200, 300]
    },
    {
      "parameters": {
        "jsCode": "return [{ json: {\n  ...$json,\n  decision: 'Escalated to Human Review',\n  reviewed: true,\n  reviewer: 'doctor_placeholder',\n  resolved_at: new Date().toISOString()\n}}];"
      },
      "name": "Human Review",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [1400, 200]
    },
    {
      "parameters": {
        "jsCode": "return [{ json: {\n  ...$json,\n  decision: 'Auto Approved',\n  reviewed: false,\n  resolved_at: new Date().toISOString()\n}}];"
      },
      "name": "Auto Approval",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [1400, 400]
    },
    {
      "parameters": {
        "jsCode": "console.log(`[MONITORING] Case ${$json.case_id} → Decision: ${$json.decision}`);\nreturn [{ json: $json }];"
      },
      "name": "Log - Monitoring",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [1600, 300]
    },
    {
      "parameters": {
        "respondWith": "json",
        "responseBody": "={{ {\n  case_id: $json.case_id,\n  priority: $json.priority,\n  confidence: $json.confidence,\n  decision: $json.decision,\n  reviewed: $json.reviewed,\n  timestamp: new Date().toISOString()\n} }}"
      },
      "name": "Response Output",
      "type": "n8n-nodes-base.respondToWebhook",
      "typeVersion": 1,
      "position": [1800, 300]
    }
  ],
  "connections": {
    "Webhook - Intake": {
      "main": [
        [
          {
            "node": "Normalize Input",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Normalize Input": {
      "main": [
        [
          {
            "node": "AI Analysis (Simulated)",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "AI Analysis (Simulated)": {
      "main": [
        [
          {
            "node": "Log - Classification",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Log - Classification": {
      "main": [
        [
          {
            "node": "Wait - Review Window",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Wait - Review Window": {
      "main": [
        [
          {
            "node": "Check Confidence (HITL)",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Check Confidence (HITL)": {
      "main": [
        [
          {
            "node": "Human Review",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "Auto Approval",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Human Review": {
      "main": [
        [
          {
            "node": "Log - Monitoring",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Auto Approval": {
      "main": [
        [
          {
            "node": "Log - Monitoring",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Log - Monitoring": {
      "main": [
        [
          {
            "node": "Response Output",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "active": false,
  "settings": {
    "executionOrder": "v1"
  }
}


