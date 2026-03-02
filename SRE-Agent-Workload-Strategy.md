# How Azure SRE Agent Can Support Dynamics, Copilot, and Fabric Workloads

## Executive Summary

Azure SRE Agent is an AI-driven operational automation service that monitors Azure resources, diagnoses issues, triages incidents, and executes remediation — with minimal human intervention. Today, it provides specialized, out-of-the-box support for Azure App Service, Container Apps, AKS, and API Management. It can also manage any Azure service through Azure CLI and REST APIs.

What makes SRE Agent strategically valuable beyond those built-in scenarios is its **extensibility framework**: a subagent builder, custom MCP (Model Context Protocol) server integration, Python tool authoring, a persistent knowledge base, and configurable incident response plans. These primitives allow teams to build custom operational automation for workloads that go beyond its native capabilities — including Dynamics, Copilot, and Fabric.

This document outlines how GTO could leverage SRE Agent's extensibility to create value across each of these three workload types, with a clear distinction between what works today and what would need to be built.

---

## How SRE Agent Works

Before diving into workload-specific scenarios, it's important to understand the operating model:

- **Resource group scoping**: You create an SRE Agent instance in the Azure portal and associate it with one or more Azure resource groups. The agent monitors and manages the resources within those groups.
- **Built-in specialized support**: App Service, Container Apps, AKS, and API Management have deep, preconfigured diagnostic capabilities.
- **General Azure management**: The agent can manage any Azure service through Azure CLI and REST API calls via custom runbooks and subagents.
- **Incident management**: Integrates with Azure Monitor alerts (default), PagerDuty, and ServiceNow.
- **Source control integration**: Connects to Azure DevOps repos/work items and GitHub repos/issues for root cause analysis down to individual lines of code and automated work ticket generation.
- **Run modes**: Operates in **Review mode** (generates a plan, waits for consent) by default. **Autonomous mode** is available within incident response plans, where the agent has implicit consent to act within defined boundaries.
- **Communication connectors**: Outlook email notifications, Microsoft Teams channel messages, and custom MCP server integration.

### Extensibility Primitives

| Capability | Description |
|---|---|
| **Subagent Builder** | Create specialized subagents with tailored instructions, tool access, handoff rules, and domain-specific knowledge bases |
| **Custom MCP Servers** | Connect the agent to external systems and APIs via remotely hosted MCP endpoints over HTTPS |
| **Python Tools** | Author custom Python logic that runs in a secure sandbox — for data transformation, API calls, log parsing, and more |
| **Knowledge Base** | Upload runbooks, architecture docs, and troubleshooting guides (.md/.txt) that subagents reference during investigations |
| **Memory System** | Persistent user memories (`#remember`), session insights (auto-learned from past investigations), and documentation connectors (Azure DevOps repo sync) |
| **Code Interpreter** | Execute Python code and shell commands in an isolated sandbox for data analysis, visualizations, and PDF report generation |
| **Scheduled Tasks** | Automate recurring operational activities — health checks, compliance scans, performance reviews — using cron schedules or natural language |

---

## 1. Dynamics 365 (D365 + Power Platform + Custom LOB Integrations)

### Why SRE Agent Is Relevant

Dynamics applications span multiple components — Power Platform customizations, Dataverse plugins, API layers, Azure Functions, and integration pipelines. Failures are frequently cross-system, making diagnosis slow and dependency-heavy. SRE Agent's ability to correlate telemetry, execute structured investigations, and automate remediation playbooks makes it a strong candidate for operational support.

### What Works Today (Built-In)

SRE Agent can immediately manage the **Azure infrastructure layer** that supports Dynamics workloads:

- Monitor and troubleshoot Azure App Services, Functions, and API Management instances that serve as D365 integration endpoints
- Query Azure Monitor logs and Application Insights telemetry from Azure-hosted middleware
- Perform deep investigation with hypothesis-driven root cause analysis on Azure resources
- Connect to Azure DevOps repos or GitHub repos used for integration code — enabling RCA down to specific lines of code
- Create automated work items in Azure DevOps or GitHub when issues are identified
- Generate scheduled health check reports on the Azure infrastructure supporting D365

### What Would Need to Be Built

To extend SRE Agent into D365-specific operational scenarios, GTO would need to invest in custom development:

| Scenario | Required Build |
|---|---|
| **Automated triage for D365 production issues** (Dataverse plugin trace logs, API exceptions, identity failures) | Route Dataverse trace logs and plugin telemetry to Azure Log Analytics. Build a custom MCP server exposing Dataverse diagnostic APIs. Create a D365-specialist subagent with troubleshooting instructions. Upload plugin architecture docs to the knowledge base. |
| **Deployment failure analysis** (D365 solution import errors, schema mismatches) | Build a custom MCP server or Python tool that can query Azure DevOps pipeline logs and D365 solution import results. Note: the native ADO integration is for repos/work items, not pipeline log analysis. |
| **Automated playbooks for known D365 incidents** (plugin queue backlogs, real-time sync failures) | Upload incident playbooks as knowledge base documents. Create subagent with domain-specific instructions and handoff rules. Configure incident response plans to trigger the subagent on relevant Azure Monitor alerts. |
| **Performance diagnostics for forms and plugins** | Requires Dataverse plugin execution telemetry routed to Log Analytics. Build a Python tool to parse and summarize plugin execution traces. |

### Estimated Effort

Medium-to-high. The primary work involves ensuring D365 operational telemetry flows into Azure Monitor / Log Analytics and building MCP servers that expose Dataverse and Power Platform APIs to the agent.

---

## 2. Copilot (M365 Copilot, Copilot Studio, MCP-Based Custom Agents)

### Why SRE Agent Is Relevant

Copilot-based workloads involve agents, connectors, MCP servers, security policies, and API orchestration. When issues occur, troubleshooting spans authentication, DLP, throttling, model context, or connector failures. These environments generate high volumes of semi-structured logs that benefit from automated correlation.

### What Works Today (Built-In)

SRE Agent can manage the Azure-hosted components of Copilot ecosystems:

- Monitor Azure App Services, Container Apps, or Functions hosting custom Copilot backends or MCP servers
- Troubleshoot API Management instances serving as Copilot gateway layers
- Query Application Insights and Log Analytics for custom agent telemetry (if instrumented)
- Send Teams notifications and Outlook emails when issues are detected
- Run scheduled health checks against Azure-hosted MCP server infrastructure

### What Would Need to Be Built

| Scenario | Required Build |
|---|---|
| **Troubleshooting MCP server failures** (connection errors, auth failures, token issues) | Build a custom MCP server that exposes MCP server health/status APIs. Create a subagent with MCP troubleshooting instructions. Upload MCP architecture documentation to the knowledge base. Note: SRE Agent connects *to* MCP servers as a client — it monitors Azure infrastructure hosting them, but doesn't natively introspect MCP protocol-level errors. |
| **Diagnosing Copilot Studio agent errors** (failing actions, connector issues, malformed responses) | Copilot Studio is a SaaS platform with its own telemetry. Would require exporting Copilot Studio analytics to Log Analytics via custom integration, then building a specialist subagent. |
| **Access, governance, and DLP policy troubleshooting** | No native integration with M365/Power Platform governance APIs. Would require a custom MCP server wrapping the Microsoft Graph Security and Compliance APIs. Upload DLP policy documentation and governance playbooks to the knowledge base. |
| **Monitoring agent quality and run health** | Route custom agent telemetry to Application Insights. Build a scheduled task that generates aggregate quality reports. Create Python tools for pattern detection (timeouts, error rates, token usage). |

### Estimated Effort

High. Copilot Studio and M365 Copilot are SaaS platforms without direct Azure resource group representation. Significant integration work is needed to surface their telemetry and APIs to SRE Agent.

---

## 3. Fabric (Data Engineering, Real-Time Analytics, Pipelines, Notebooks)

### Why SRE Agent Is Relevant

Fabric workloads are multi-service (Pipelines, Lakehouse, Warehouse, Notebooks, Real-Time Intelligence) and produce dispersed operational logs across OneLake, Data Factory, Synapse runtimes, and Spark engines. SRE Agent excels in environments where logs are complex and multi-step systems require correlation.

### What Works Today (Built-In)

SRE Agent can manage underlying Azure infrastructure and any Kusto/ADX clusters:

- Manage and query Azure Data Explorer (Kusto) clusters — a native integration of the SRE Agent
- Monitor Azure resources supporting Fabric workloads (storage accounts, networking, identity)
- Query Azure Monitor and Log Analytics for any telemetry that's been routed there
- Generate scheduled reports and visualizations via code interpreter
- Automated incident response for Azure-level alerts affecting Fabric infrastructure

### What Would Need to Be Built

| Scenario | Required Build |
|---|---|
| **Pipeline failure diagnosis** (failing transformations, missing datasets, auth issues) | Fabric pipeline telemetry is within the Fabric SaaS platform. Would require a custom MCP server wrapping Fabric REST APIs to expose pipeline run status, error details, and dependency chains to the agent. Create a data engineering subagent with pipeline troubleshooting instructions. |
| **Spark / Notebook error root-cause analysis** | Build a Python tool to parse Spark stack traces into human-readable explanations. Route Fabric notebook execution logs to Log Analytics (if possible). Upload Spark troubleshooting guides and memory/partitioning best practices to the knowledge base. |
| **Real-Time Intelligence (KQL / Eventstream) issue triage** | If RTI data is in Azure Data Explorer, the agent can query it natively. For Fabric-native Eventhouse, a custom MCP server may be needed. Create a subagent focused on ingestion, schema drift, and throughput analysis. |
| **Data quality degradation detection** | Build Python tools for anomaly detection on data freshness, volume, and schema metrics. Create scheduled tasks that run data quality checks and alert via Teams/Outlook. |
| **Dynamics-to-Fabric integration failures** (Synapse Link, Power Platform exports) | Would require correlating logs from multiple systems. Build an integration-specialist subagent with access to both D365 and Fabric MCP servers. Upload Synapse Link architecture docs to the knowledge base. |

### Estimated Effort

Medium-to-high. The Kusto/ADX integration is a strong foundation. The primary gap is surfacing Fabric-native telemetry (pipeline runs, notebook executions, Eventstream status) into a form the agent can access — either via Log Analytics routing or custom MCP servers wrapping Fabric APIs.

---

## Summary: Value vs. Effort Matrix

| Workload | Out-of-Box Value | Custom Build Required | Strategic Value |
|---|---|---|---|
| **Dynamics 365** | Azure infrastructure monitoring for D365 backends (App Service, Functions, APIM) | MCP servers for Dataverse APIs, telemetry routing to Log Analytics, D365-specialist subagents | High — cross-stack triage and automated playbooks for known incident patterns |
| **Copilot** | Azure infrastructure monitoring for custom agent backends | MCP servers for Copilot Studio / M365 APIs, telemetry export, governance integration | High — but highest integration effort due to SaaS platform boundaries |
| **Fabric** | Kusto/ADX cluster querying, Azure infrastructure monitoring | MCP servers for Fabric REST APIs, Spark log parsing tools, pipeline telemetry routing | High — strong Kusto foundation; good fit for data quality monitoring use cases |

---

## Implementation Approach

We recommend a phased approach:

### Phase 1: Foundation (Weeks 1–4)
- Deploy SRE Agent instances and associate them with Azure resource groups supporting D365, Copilot, and Fabric workloads
- Configure incident management (Azure Monitor alerts, PagerDuty, or ServiceNow)
- Connect source code repos (ADO/GitHub) for integration code RCA
- Upload existing runbooks, architecture diagrams, and troubleshooting guides to the knowledge base
- Set up scheduled health check tasks for Azure infrastructure

### Phase 2: Telemetry Routing (Weeks 4–8)
- Route D365/Dataverse plugin trace logs to Azure Log Analytics
- Instrument custom Copilot backends with Application Insights
- Configure Fabric workspace telemetry export where possible
- Validate that operational signals from all three workloads are queryable by the agent

### Phase 3: Custom Integration (Weeks 8–16)
- Build custom MCP servers for D365, Copilot Studio, and Fabric APIs
- Create domain-specialist subagents for each workload area
- Author Python tools for log parsing, anomaly detection, and data transformation
- Configure incident response plans with appropriate triggers and autonomy levels
- Build and test automated playbooks for high-frequency incident patterns

### Phase 4: Optimization (Ongoing)
- Review session insights to identify knowledge gaps and recurring patterns
- Refine subagent instructions based on real-world incident data
- Expand automated remediation as confidence grows
- Tune alert thresholds and incident filters to reduce noise

---

## Cost Considerations

Azure SRE Agent billing is based on Azure Agent Units (AAUs):

| Flow Type | Rate |
|---|---|
| **Always-on flow** (continuous monitoring) | 4 AAUs per agent hour |
| **Active flow** (task execution) | 0.25 AAUs per agent task, per second |

At current preview pricing ($0.10/AAU), the baseline cost is approximately **$292/month per agent** for continuous monitoring alone, plus variable active flow costs based on task volume.

For reference, a minimal-workload scenario (few incidents, occasional interactive use) runs approximately **$322/month per agent**, while a high-operational-load scenario (two incidents/day, 10 min each) runs approximately **$1,222/month per agent**.

If GTO deploys separate agents per workload domain, plan for cumulative costs accordingly. Consider starting with a single agent managing multiple resource groups to validate the approach before scaling.

---

## Key Takeaways

1. **SRE Agent is not an "Azure DevOps agent"** — it's an Azure-native operational automation platform that monitors resource groups and uses AI to diagnose, triage, and remediate issues.

2. **Out-of-the-box specialization is limited** to App Service, Container Apps, AKS, and API Management — but any Azure service is manageable through CLI/REST.

3. **The real strategic value for GTO lies in the extensibility framework** — subagent builder, custom MCP servers, Python tools, knowledge base, and memory system create a programmable operational automation chassis.

4. **D365, Copilot, and Fabric are SaaS/PaaS platforms** — they don't natively expose their internals as Azure resource group resources. Custom integration work is required to surface their telemetry and APIs to the agent.

5. **The effort is real but the payoff compounds** — once the integration layer is built, the agent's memory system learns from every session, and its knowledge base grows with each uploaded playbook, making it progressively more effective over time.

---

## Reference Documentation

| Topic | Link |
|---|---|
| SRE Agent Overview | https://learn.microsoft.com/en-us/azure/sre-agent/overview |
| Create and Use an Agent | https://learn.microsoft.com/en-us/azure/sre-agent/usage |
| Subagent Builder | https://learn.microsoft.com/en-us/azure/sre-agent/subagent-builder-overview |
| Subagent Builder Scenarios | https://learn.microsoft.com/en-us/azure/sre-agent/subagent-builder-scenarios |
| Custom MCP Server Integration | https://learn.microsoft.com/en-us/azure/sre-agent/custom-mcp-server |
| Custom Python Tools | https://learn.microsoft.com/en-us/azure/sre-agent/custom-logic-python |
| Memory System | https://learn.microsoft.com/en-us/azure/sre-agent/memory-system |
| Incident Management | https://learn.microsoft.com/en-us/azure/sre-agent/incident-management |
| Incident Response Plans | https://learn.microsoft.com/en-us/azure/sre-agent/incident-response-plan |
| Agent Run Modes | https://learn.microsoft.com/en-us/azure/sre-agent/agent-run-modes |
| Scheduled Tasks | https://learn.microsoft.com/en-us/azure/sre-agent/scheduled-tasks |
| Code Repository Connection | https://learn.microsoft.com/en-us/azure/sre-agent/code-repository-connect |
| Deep Investigation | https://learn.microsoft.com/en-us/azure/sre-agent/deep-investigation |
| Code Interpreter | https://learn.microsoft.com/en-us/azure/sre-agent/code-interpreter |
| Connectors | https://learn.microsoft.com/en-us/azure/sre-agent/connectors |
| Roles and Permissions | https://learn.microsoft.com/en-us/azure/sre-agent/roles-permissions-overview |
| Billing | https://learn.microsoft.com/en-us/azure/sre-agent/billing |
| Data Privacy | https://learn.microsoft.com/en-us/azure/sre-agent/data-privacy |
| Starter Prompts | https://learn.microsoft.com/en-us/azure/sre-agent/prompts |
| FAQ | https://learn.microsoft.com/en-us/azure/sre-agent/faq |
