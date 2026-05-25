
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
