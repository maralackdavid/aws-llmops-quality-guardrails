# Automated LLMOps Telemetry & Quality Guardrails (AWS SageMaker AI + CloudWatch)

[![AWS SageMaker](https://img.shields.io/badge/AWS-SageMaker_AI-FF9900?logo=amazonaws)](https://aws.amazon.com/sagemaker/)
[![CloudWatch](https://img.shields.io/badge/AWS-CloudWatch_XRay-FF9900?logo=amazonaws)](https://aws.amazon.com/cloudwatch/)
[![CodePipeline](https://img.shields.io/badge/AWS-CodePipeline-FF9900?logo=amazonaws)](https://aws.amazon.com/codepipeline/)
[![Framework](https://img.shields.io/badge/Methodology-PMI--CPMAI-blue)](https://www.pmi.org/)

An enterprise-grade LLMOps telemetry and automated quality evaluation platform built on **AWS SageMaker Model Evaluation**, **AWS X-Ray**, and **Amazon CloudWatch**. Enforces automated CI/CD quality regression gating to block deployments whenever citation accuracy or hallucination rates violate operational SLAs.

---

## 1. CPMAI Phase I: Matching AI to Business Needs

Following the **PMI Certified Professional in Managing AI (CPMAI) Phase I (Business Understanding)** framework, this project establishes the business justification, observability metrics, and risk governance required to run production Generative AI applications reliably without silent quality degradation or cost overruns.

### 1.1 Business Objective & ROI Feasibility
* **Target Audience**: Enterprise AI Platform Engineering, DevOps/SRE Teams, and Product Owners.
* **Problem Statement**: Unmonitored production LLM applications suffer from silent quality degradation, prompt regression, hallucination spikes, and unexpected token cost surges. Manual post-deployment audits are reactive, exposing the enterprise to customer churn, brand damage, and SLA penalty fees.
* **Projected Financial ROI**: Preventing silent production regressions and automated blocking of bad prompt deployments avoids an estimated **\$950K in annual SLA violation penalties, incident response overhead, and runaway token costs**.

### 1.2 Cognitive vs. Non-Cognitive Justification
* **Why AI Evaluation is Required (Probabilistic Need)**: Assessing LLM output quality (faithfulness, semantic relevancy, hallucination detection, citation accuracy) requires probabilistic model-based evaluation (SageMaker Model Evaluation / LLM-as-a-judge) that simple regex or static assertion tests cannot accomplish.
* **Non-Cognitive Integration (Deterministic Gating)**: Metric aggregation, P95 latency calculation, CloudWatch alarm triggers, and CI/CD build aborts in AWS CodePipeline are **100% deterministic**, ensuring rigid operational reliability around the probabilistic evaluation scores.

### 1.3 AI Pattern Mapping
* **Primary Pattern**: **Patterns & Anomalies** (detecting model drift, citation degradation, hallucination spikes, and token spend anomalies).
* **Secondary Pattern**: **Predictive Analytics & Decision Support** (automating go/no-go deployment decisions in CI/CD build pipelines based on benchmark score trends).

### 1.4 DIKUW Pyramid Alignment
* **Data (Base Facts)**: Raw prompt/completion token logs, execution timestamps, and ground-truth Q&A pairs stored in S3.
* **Information (Structured Metrics)**: Calculated P50/P95/P99 latency percentiles, cost-per-query subsegments, and citation coverage ratios emitted via AWS CloudWatch & X-Ray.
* **Knowledge (Evaluation Insights)**: Faithfulness, answer relevancy, and context recall scores computed automatically by SageMaker Model Evaluation across prompt iterations.
* **Understanding & Governance (Automated Regression Gating)**: Enforcing automated CI/CD build failures when quality drops below 90% citation accuracy or exceeds 5% hallucination rate, protecting brand integrity and customer trust.

### 1.5 CPMAI Go/No-Go Assessment (3x3 Feasibility Matrix)

| Feasibility Pillar | Assessment Criteria | Status | Strategic Justification |
| :--- | :--- | :---: | :--- |
| **Business Feasibility** | Problem Definition | 🟢 **GO** | Clear operational risk of silent LLM regressions and token budget overruns. |
| | Sponsor Commitment | 🟢 **GO** | SRE and AI leadership require automated quality gates prior to production push. |
| | Sufficient ROI | 🟢 **GO** | Significant cost avoidance (\$950K/yr) by preventing SLA breaches and outages. |
| **Data Feasibility** | Data Availability | 🟢 **GO** | 100-item golden test dataset established and curated in Amazon S3. |
| | Access & Security | 🟢 **GO** | X-Ray and CloudWatch encrypted at rest with KMS; strict IAM execution roles. |
| | Data Quality | 🟢 **GO** | Golden dataset validated against expert human ground-truth answers. |
| **Execution Feasibility** | Technology & Skills | 🟢 **GO** | Native integration between SageMaker, CloudWatch, X-Ray, and CodePipeline. |
| | Implementation Timeline | 🟢 **GO** | Pipeline telemetry and automated gates deployed within standard 2-week sprint. |
| | Operational Context | 🟢 **GO** | Runs natively inside existing AWS CI/CD pipelines without developer workflow friction. |

*Overall Assessment*: **ALL GREEN (GO)** — Project approved for technical implementation.

---

## 2. Target System Architecture

```mermaid
graph TD
    subgraph ExecutionLayer ["1. GenAI Application & RAG Execution Tier"]
        APIGW["AWS API Gateway"]
        LambdaApp["AWS Lambda / Fargate\n(GenAI Application Handler)"]
        BedrockFM["Amazon Bedrock\n(Claude 3.5 Sonnet)"]
    end

    subgraph ObservabilityTier ["2. Real-Time Telemetry & Distributed Tracing"]
        XRay["AWS X-Ray\n(Trace Graphs & Subsegment Spans)"]
        CloudWatchLogs["Amazon CloudWatch Logs\n(Raw Token Spend & Citation Logs)"]
        MetricsCollector["CloudWatch Custom Metrics\n(P50/P95 Latency & Cost per Request)"]
    end

    subgraph EvaluationHarness ["3. Continuous Evaluation & Quality Gating"]
        GoldenDataset[("100-Item Golden Test Dataset\n(Amazon S3)")]
        SageMakerEval["AWS SageMaker Model Evaluation / Ragas\n(Faithfulness, Relevancy & Citation Checks)"]
        CodePipeline["AWS CodePipeline / GitHub Actions\n(CI/CD Automated Deployment Gate)"]
    end

    subgraph AnomalyAndAlerting ["4. Operational Alerting & Fallback"]
        AlarmTrigger{"Quality Gate Check\n(Citation > 90% & Hallucination < 5%)"}
        SNSAlert["AWS SNS Alert\n(PagerDuty / Slack Notification)"]
        RollbackState["Automated Rollback / Fallback Model"]
    end

    %% Application Execution Flow
    APIGW -->|1. Submit Request| LambdaApp
    LambdaApp -->|2. Invoke Model| BedrockFM
    
    %% Telemetry Emission
    LambdaApp -.->|3. Emit Execution Trace| XRay
    LambdaApp -.->|4. Log Token Spend & Similarity Scores| CloudWatchLogs
    CloudWatchLogs -->|5. Aggregate Metrics| MetricsCollector

    %% CI/CD Quality Gating Flow
    CodePipeline -->|6. Trigger Pre-Deployment Eval| SageMakerEval
    GoldenDataset -->|7. Load Ground-Truth Test Pairs| SageMakerEval
    SageMakerEval -->|8. Evaluate Faithfulness & Hallucinations| AlarmTrigger

    %% Decision Gate
    AlarmTrigger -->|PASS: Metrics Above Threshold| CodePipeline
    AlarmTrigger -->|FAIL: Quality Regression Detected| SNSAlert
    SNSAlert -->|Block Deployment & Trigger| RollbackState