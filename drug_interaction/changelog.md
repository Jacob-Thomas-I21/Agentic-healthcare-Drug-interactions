CHANGELOG
v1.0

    Initial Release: Implemented basic ADR (Adverse Drug Reaction) detection using a hardcoded workflow.
    No Encryption: PHI (Protected Health Information) was not encrypted.
    No DLQ: No Dead Letter Queue (DLQ) for handling failed messages.
    Basic Metrics: Rudimentary logging without advanced metrics or dashboards.

vPORTFOLIO

    LLM Router Integration: Introduced an LLM Router to manage and distribute tasks across multiple language models (GPT-4o, Claude, DeepSeek, MiniMax).
    DLQ Fallback: Implemented a Dead Letter Queue (DLQ) to handle and recover from failed message processing.
    Provenance Tracking: Added provenance hashing to ensure data integrity and traceability of decisions.
    Metrics Export: Integrated Prometheus for exporting key metrics such as latency, fallback rates, and confidence scores.
    Sticky Notes & Sub-Workflows: Added n8n sticky notes for documentation and annotations. Stubbed out sub-workflows for modularity.

vFINAL

    AES-256-GCM Encryption: Implemented AES-256-GCM encryption with key rotation for securing PHI. Keys are managed securely via HashiCorp Vault.
    Explainability Layer: Enhanced the workflow with GPT-based explanations for decisions, providing transparency and auditability.
    Confidence Calibration: Introduced confidence scoring and calibration to ensure decision accuracy and reliability.
    Slack Alerts: Integrated Slack notifications for real-time alerts and updates to the clinical operations team.
    CI Runner & Health Checks: Added a Continuous Integration (CI) runner for automated testing and a daily health check cron job to monitor system health.
    Real Prometheus Pushes: Configured real-time metric exports to Prometheus, enabling live monitoring and alerting.
    Circuit Breaker & Subflow Execution: Implemented a circuit breaker to prevent cascading failures and enhanced the sub-workflow execution logic for better modularity and resilience.
