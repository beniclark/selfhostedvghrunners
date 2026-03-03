# Azure SRE Agent — Fit for Dynamics, Copilot, and Fabric Workloads

## What Is Azure SRE Agent?

Azure SRE Agent (currently in Preview) is an AI-driven operational automation service that monitors Azure resource groups, diagnoses issues, triages incidents, and can execute remediation with minimal human intervention. You create an agent in the Azure portal, associate it with resource groups, and it goes to work — querying logs, correlating telemetry, running investigations, and notifying your team.

It has deep, specialized support today for **App Service, Container Apps, AKS, and API Management**. For any other Azure service, it can still operate through Azure CLI and REST API calls — and for anything beyond that, it offers an extensibility framework: custom subagents, MCP server integration, Python tools, a knowledge base, scheduled tasks, and a persistent memory system.

## How Does It Fit GTO's Workloads?

### Dynamics 365

**What works today:** SRE Agent can immediately monitor and troubleshoot the Azure infrastructure backing D365 — App Services, Functions, and API Management instances serving as integration endpoints. It can query Application Insights and Log Analytics, connect to your ADO repos (repos and work items — not pipeline logs) for root cause analysis down to specific lines of code, and auto-create work items for identified issues.

**What would need to be built:** D365-specific diagnostics (Dataverse plugin trace logs, solution import errors, plugin queue issues) would require routing that telemetry into Azure Log Analytics and building custom MCP servers or Python tools to expose Dataverse diagnostic APIs to the agent. You'd also upload D365-specific runbooks and architecture docs into the agent's knowledge base so it knows how to interpret what it finds.

### Copilot (M365 Copilot, Copilot Studio, Custom Agents)

**What works today:** SRE Agent can monitor Azure-hosted Copilot backends — App Services, Container Apps, or Functions hosting custom agents or MCP servers, plus any API Management gateways. It can send Teams and Outlook notifications and run scheduled health checks against that infrastructure.

**What would need to be built:** Copilot Studio and M365 Copilot are SaaS platforms, so their internal telemetry (failing actions, connector issues, DLP policy conflicts) isn't natively visible to SRE Agent. You'd need to export that telemetry to Log Analytics and build custom MCP servers wrapping Microsoft Graph and Copilot Studio APIs. This is the highest integration effort of the three workloads.

### Fabric (Pipelines, Lakehouse, Notebooks, Real-Time Intelligence)

**What works today:** SRE Agent has native support for querying Azure Data Explorer (Kusto) clusters, which is a strong foundation. It can also monitor the Azure infrastructure supporting Fabric workloads (storage accounts, networking, identity) and generate scheduled reports via its built-in code interpreter.

**What would need to be built:** Fabric pipeline and notebook execution telemetry lives within the Fabric SaaS platform. You'd need custom MCP servers wrapping Fabric REST APIs to expose pipeline run status, error details, and Spark logs to the agent. Data quality monitoring (freshness, volume, schema drift) could be built using Python tools and scheduled tasks.

## The Common Pattern

For all three workloads, the story is the same:

1. **Azure infrastructure layer** — SRE Agent handles this today, out of the box
2. **Application/SaaS layer** — requires custom integration work (telemetry routing + MCP servers + specialist subagents)

The extensibility framework is what makes this viable. You're not waiting for Microsoft to ship native D365/Copilot/Fabric support — you can build domain-specific operational automation using the subagent builder, MCP servers, and Python tools available today.

## Cost

Billing is based on Azure Agent Units (AAUs) at $0.10/AAU during preview:

- **Baseline** (continuous monitoring): ~$292/month per agent
- **Light use** (few incidents): ~$322/month per agent  
- **Heavy use** (multiple daily incidents): ~$1,222/month per agent

Consider starting with a single agent managing multiple resource groups before scaling to per-workload agents.

## Recommendation

A phased approach makes sense:

1. **Start with the Azure infrastructure layer** — deploy SRE Agent against the resource groups backing all three workloads, connect your repos, upload existing runbooks, and set up scheduled health checks. This delivers value immediately with zero custom development.
2. **Route telemetry** — get D365 plugin traces, custom Copilot agent logs, and Fabric pipeline metrics flowing into Log Analytics so the agent can query them.
3. **Build custom integrations** — develop MCP servers for Dataverse, Copilot Studio, and Fabric APIs; create domain-specialist subagents; author incident response plans for known failure patterns.

The effort is real — especially for Copilot and Fabric where you're bridging SaaS boundaries — but the payoff compounds as the agent's memory system learns from every session and its knowledge base grows with each uploaded playbook.

## Key Links

- [SRE Agent Overview](https://learn.microsoft.com/en-us/azure/sre-agent/overview)
- [Subagent Builder](https://learn.microsoft.com/en-us/azure/sre-agent/subagent-builder-overview)
- [Custom MCP Servers](https://learn.microsoft.com/en-us/azure/sre-agent/custom-mcp-server)
- [Custom Python Tools](https://learn.microsoft.com/en-us/azure/sre-agent/custom-logic-python)
- [Billing](https://learn.microsoft.com/en-us/azure/sre-agent/billing)
- [Incident Management](https://learn.microsoft.com/en-us/azure/sre-agent/incident-management)
