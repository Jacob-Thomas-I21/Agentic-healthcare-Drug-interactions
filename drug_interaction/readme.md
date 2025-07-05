# Drug Interaction Agent Workflow - vFINAL

`drug_interaction_agent_vFINAL` is a production-grade clinical AI workflow for detecting adverse drug reactions (ADR), scoring patient risk, and routing explainable alerts. It orchestrates multiple LLMs (GPT-4o, Claude, etc.), applies AES-256-GCM encryption to PHI, includes retry logic, circuit-breaker resilience, and a Dead Letter Queue (DLQ) for guaranteed error capture.

## 🧱 Architecture Diagram (Mermaid or SVG)

```
graph TD
  Webhook[Secure FHIR Webhook] --> Normalize[Normalize & Deduplicate]
  Normalize --> Encrypt[Encrypt PHI]
  Encrypt --> Secret[Fetch Vault Secrets]
  Secret --> Router[LLM Router]
  Router --> Subflow[Execute Clinical Analysis Sub-Workflow]
  Subflow --> Decision[Score Confidence & Decide]
  Decision --> Explain[Generate Explanation]
  Explain --> Provenance[Generate Provenance Hash]
  Provenance --> DLQGate[DLQ Gate]
  DLQGate --> Audit[Audit + GPT Summary]
  Audit --> Metrics[Export Metrics]
  Metrics --> Dashboard[Push to Realtime Dashboard]
  DLQGate --> DLQ[Send to DLQ]
  DLQ --> WebhookResp[Respond to Webhook]
  Dashboard --> WebhookResp

```

**Explanation of the Diagram:**

1. 1.**Secure FHIR Webhook**: Ingest patient data securely.
2. 2.**Normalize & Deduplicate**: Clean and organize the data.
3. 3.**Encrypt PHI**: Encrypt sensitive patient information using AES-256-GCM.
4. 4.**Fetch Vault Secrets**: Retrieve encryption keys and other secrets from HashiCorp Vault.
5. 5.**LLM Router**: Direct the data to the appropriate LLM for processing.
6. 6.**Execute Clinical Analysis Sub-Workflow**: Perform ADR prediction and dose stratification.
7. 7.**Score Confidence & Decide**: Evaluate the confidence of the predictions and make decisions.
8. 8.**Generate Explanation**: Create a human-readable explanation for the decisions.
9. 9.**Generate Provenance Hash**: Create a hash to ensure data integrity and traceability.
10. 10.**DLQ Gate**: Determine whether to proceed with the workflow or send the data to the DLQ.
11. 11.**Audit + GPT Summary**: Log the transaction and generate a GPT-based summary for audit purposes.
12. 12.**Export Metrics**: Push metrics to Prometheus for monitoring.
13. 13.**Push to Realtime Dashboard**: Update the realtime dashboard with the latest metrics.
14. 14.**Send to DLQ**: If the DLQ Gate condition is met, send the data to the DLQ for further processing.
15. 15.**Respond to Webhook**: Acknowledge the webhook request with a response.

---

## 📦 Setup Instructions

### Environment Variables

Ensure the following environment variables are set in your deployment environment:

- `VAULT_URL`: The URL for your HashiCorp Vault service to fetch secrets.
- `VAULT_TOKEN`: The token for authenticating with the Vault service.
- `WEBHOOK_API_KEY`: API key for authenticating incoming FHIR data webhook requests.
- `VAULT_SLACK_WEBHOOK`: Slack webhook URL for sending alerts and notifications via Vault.
- `CLINICAL_ANALYSIS_SUB_WORKFLOW_ID`: The ID of the sub-workflow responsible for clinical analysis.
- `CLINOPS_WEBHOOK_URL`: Webhook URL for sending manual review requests to ClinOps.
- `AUDIT_LOG_URL`: URL for accessing the audit log.
- `ALERTS_API_URL`: API endpoint for routing alerts.
- `PROMETHEUS_PUSHGATEWAY_URL`: URL for Prometheus Pushgateway to export metrics.
- `MIN_CONFIDENCE_THRESHOLD`: Minimum confidence score threshold (default: 70).
- `MAX_CONFIDENCE_THRESHOLD`: Maximum confidence score threshold (default: 95).
- `ENABLE_EXPLAINABILITY`: Flag to enable/disable explainability (default: true).

### Webhook Secrets

Configure the webhook endpoint as follows:

- **Path**: `/drug-interaction`
- **Authentication**: Header `X-API-KEY` with the value from `WEBHOOK_API_KEY`

### Vault Configuration

All sensitive information is securely managed using HashiCorp Vault. Required secrets:

- `AES_KEY`: The AES-256 encryption key for encrypting PHI.
- `AES_KEY_ID`: The version identifier for the AES key.
- `WEBHOOK_API_KEY`: The shared secret for validating webhook calls.

### Deployment Configuration

- **Docker Image**: `n8nio/n8n:1.45.1`
- **Orchestration**: Kubernetes or Docker Swarm
- **Secret Management**: Use Vault or `.env` injection during container spin-up

---

##

---

## 🔐 Security Design

- **AES-256-GCM Encryption** for all PHI
- **Key Rotation** supported via `AES_KEY_ID`
- **Vault Integration** for secret management
- **DLQ Fallback Path**: Failed items are rerouted for later processing
- **DLQ Gate**: Routes only true failure items, filtering noise

---

## 🛡 Resilience Features

- **Retry Logic**: HTTP nodes configured with retry-on-fail and timeout guards
- **Circuit Breaker**: LLM paths are gated to stop execution after N failures
- **DLQ Routing**: All failed paths write encrypted data with reason into DLQ endpoint
- **Manual Review Trigger**: Mid-confidence items are routed to ClinOps for approval

---

## 📊 Metrics (Prometheus)

- **Latency (ms)**: Avg end-to-end processing
- **Fallback Rate**: Frequency of DLQ or retry usage
- **Confidence Distribution**: Low/Mid/High outcome split
- **Queue Depth**: Estimated load from executions

---

## 📘 Explainability Layer

- **GPT-4o Audit Summary**: Natural language explanations for ADR predictions
- **SHAP-style Explanation**: Feature contribution hints injected for transparency
- **Confidence Score**: Calibrated output between `MIN_CONFIDENCE_THRESHOLD` and `MAX_CONFIDENCE_THRESHOLD`

---

## ⚙️ Importing + Running in n8n

### Importing

1. Download `drug_interaction_agent_vFINAL.json`
2. Log in to your n8n instance
3. Navigate to **Import Workflow**, upload JSON
4. Configure credentials for Vault, Webhook, Slack, Prometheus

### Testing with Sample Input

1. Use `sample_input_fhir.json` from the repo
2. Send POST request to `/drug-interaction` using Postman or cURL
3. View execution graph and node outputs inside n8n UI

### Monitoring Results

- **Audit Log**: Located at `${AUDIT_LOG_URL}`
- **Slack Alerts**: Sent via `VAULT_SLACK_WEBHOOK`
- **ClinOps Review**: Mid-confidence flagged alerts are sent to `${CLINOPS_WEBHOOK_URL}`

---

*This workflow is anonymized and safe for public portfolio use. No real PHI or LLM credentials are exposed.*

