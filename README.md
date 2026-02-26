# GitHub-Hosted Runners vs Self-Hosted Runners

A comprehensive comparison of GitHub-hosted and self-hosted runners for GitHub Actions, including recommendations and common customer questions.

---

## Table of Contents

- [Overview](#overview)
- [Key Differences](#key-differences)
- [Detailed Comparison](#detailed-comparison)
  - [Infrastructure & Maintenance](#infrastructure--maintenance)
  - [Hardware & Performance](#hardware--performance)
  - [Networking & Security](#networking--security)
  - [Scaling & Operations](#scaling--operations)
  - [Cost](#cost)
- [When to Recommend Each](#when-to-recommend-each)
  - [GitHub-Hosted (Standard)](#github-hosted-standard)
  - [GitHub-Hosted (Larger Runners)](#github-hosted-larger-runners)
  - [Self-Hosted (Manual)](#self-hosted-manual)
  - [Self-Hosted (ARC / Kubernetes)](#self-hosted-arc--kubernetes)
  - [Hybrid Approach](#hybrid-approach)
- [Expected Customer Questions](#expected-customer-questions)
  - [Cost & Licensing](#cost--licensing)
  - [Security & Compliance](#security--compliance)
  - [Performance & Reliability](#performance--reliability)
  - [Migration & Operations](#migration--operations)
  - [Architecture & Scaling](#architecture--scaling)
- [Reference Documentation](#reference-documentation)

---

## Overview

GitHub Actions supports two categories of runners:

- **GitHub-Hosted Runners** — Virtual machines managed by GitHub. Available in standard and larger configurations. Each job receives a fresh, ephemeral VM.
- **Self-Hosted Runners** — Machines you manage yourself (physical, VM, or container). Can be provisioned manually or orchestrated at scale using Actions Runner Controller (ARC) on Kubernetes.

---

## Key Differences

| Category | GitHub-Hosted (Standard) | GitHub-Hosted (Larger) | Self-Hosted (Manual) | Self-Hosted (ARC/K8s) |
|---|---|---|---|---|
| **Managed By** | GitHub | GitHub | You | You (K8s cluster + controller) |
| **Specs** | 4-core, 16 GB RAM, 14 GB SSD | Up to 64-core, 256 GB RAM, GPU | You choose | You choose (pod resources) |
| **OS** | Ubuntu, Windows, macOS | Ubuntu, Windows, macOS | Any OS with runner agent | Container-based (Linux primarily) |
| **Ephemeral** | ✅ Always | ✅ Always | ⚠️ Optional (you configure) | ✅ Default |
| **Private Networking** | ❌ Not available | ✅ Azure VNET integration | ✅ Full network access | ✅ Full network access |
| **Static IPs** | ❌ Dynamic | ✅ Via VNET | ✅ You control | ✅ You control |
| **Autoscaling** | ✅ Managed by GitHub | ✅ Managed by GitHub | ❌ Manual | ✅ Kubernetes-native (Runner Scale Sets) |
| **Runner Groups** | ❌ Not applicable | ✅ Supported | ✅ Supported | ✅ Supported |
| **Maintenance** | None | None | Full responsibility | K8s cluster + runner images |
| **Plan Required** | Free+ | Team / Enterprise Cloud | Free+ | Free+ |
| **Per-Minute Cost** | ✅ Included minutes, then pay-per-minute | ✅ Pay-per-minute | ❌ No GitHub charge (you pay infra) | ❌ No GitHub charge (you pay infra) |

---

## Detailed Comparison

### Infrastructure & Maintenance

#### GitHub-Hosted Runners

- GitHub manages all infrastructure — provisioning, OS updates, security patching, and tool updates.
- Pre-installed tool cache includes a wide set of SDKs, runtimes, Docker, and common CLI tools.
- Every job runs on a **fresh VM**. No state persists between jobs — this eliminates cross-job contamination but means no persistent caching on the runner itself.
- Larger runners are available on **GitHub Team and Enterprise Cloud** plans and offer machines up to 64-core with up to 256 GB RAM, GPU support (Linux), and macOS M1/M2 chips.

#### Self-Hosted Runners

- You are responsible for provisioning, patching, monitoring, and securing the machines.
- You install and maintain the exact toolchain your workflows need.
- Persistent environment means caches (Docker layers, package caches) survive between jobs — beneficial for large builds.
- When using **ARC on Kubernetes**, the controller manages pod lifecycle. GitHub supports the ARC controller and runner images, but your Kubernetes cluster, networking, and storage are your responsibility.

### Hardware & Performance

| Runner Type | CPU | RAM | Storage | GPU |
|---|---|---|---|---|
| GitHub-Hosted Standard (Linux/Windows) | 4-core | 16 GB | 14 GB SSD | ❌ |
| GitHub-Hosted Standard (macOS) | 3-core (M1) or 4-core (Intel) | 14 GB | 14 GB SSD | ❌ |
| GitHub-Hosted Larger (Linux/Windows) | Up to 64-core | Up to 256 GB | Configurable | ✅ Linux only |
| GitHub-Hosted Larger (macOS) | M1/M2 higher core counts | Configurable | Configurable | ❌ |
| Self-Hosted | **You choose** | **You choose** | **You choose** | **You choose** |

### Networking & Security

#### GitHub-Hosted

- **Standard runners** have public internet access only. IP ranges are dynamic and published by GitHub but change regularly — difficult for firewall allow-listing.
- **Larger runners** support **Azure Private Networking** via VNET integration, enabling access to private resources without exposing them to the public internet. This also provides **static IP ranges**.
- Additional private networking patterns include OIDC authentication with cloud providers, API gateways, and WireGuard tunnels.

#### Self-Hosted

- Full access to internal networks, private databases, on-prem resources, and internal package feeds.
- Static/known IPs — straightforward for firewall allow-listing.
- **Security consideration:** Persistent self-hosted runners carry risk of secrets or artifacts leaking between jobs if not properly hardened. Ephemeral runners (default in ARC) mitigate this risk.

#### Runner Groups

- **Runner groups** allow you to control which organizations and repositories have access to specific runners.
- Available for both **self-hosted** and **larger GitHub-hosted** runners.
- Enterprise-level groups can be shared across organizations.
- Organization-level groups can be scoped to specific repositories.

### Scaling & Operations

| Approach | Scaling | Concurrency Control | Effort |
|---|---|---|---|
| GitHub-Hosted (Standard) | Automatic — managed by GitHub | Determined by plan limits | None |
| GitHub-Hosted (Larger) | Automatic — managed by GitHub | Configurable max concurrency | Minimal |
| Self-Hosted (Manual) | Manual — you add/remove machines | You manage capacity | High |
| Self-Hosted (ARC) | Automatic — Kubernetes-native via **Runner Scale Sets** | Configurable min/max replicas | Medium (requires K8s expertise) |

#### Actions Runner Controller (ARC)

- ARC is a **Kubernetes operator** that orchestrates and scales self-hosted runners.
- Uses **Runner Scale Sets** as the scaling mechanism — this is the **newest and recommended** mode, replacing the older listener-based approach.
- Runners are **ephemeral by default** — each job gets a clean pod.
- Requires **Kubernetes 1.27+** and **Helm 3** for deployment.
- GitHub supports the controller and runner images. Your Kubernetes cluster, networking, and storage are your responsibility.

### Cost

| Runner Type | Pricing Model | Notes |
|---|---|---|
| GitHub-Hosted Standard | Included free minutes per plan; pay-per-minute overage | Free minutes vary by plan (Free, Pro, Team, Enterprise) |
| GitHub-Hosted Larger | Pay-per-minute (no free minutes) | Higher per-minute rate based on machine size |
| Self-Hosted | No GitHub per-minute charge | You pay for underlying infrastructure (VMs, K8s cluster, networking) |

> **Note:** Self-hosted runners have **hidden costs** including staff time for maintenance, patching, monitoring, and incident response. Factor these into any cost comparison.

---

## When to Recommend Each

### GitHub-Hosted (Standard)

✅ Best for:

- Teams that want **zero infrastructure management**
- Standard build/test workloads (Node, .NET, Java, Python, Go, etc.)
- Open-source projects (free minutes on public repos)
- Teams without strict network isolation or compliance requirements
- Small to medium build volumes

### GitHub-Hosted (Larger Runners)

✅ Best for:

- Workloads requiring **higher CPU, memory, or GPU** without managing infrastructure
- Teams that need **private networking via Azure VNET** but don't want to manage self-hosted runners
- Organizations that need **static IP addresses** for firewall rules
- Teams on **GitHub Team or Enterprise Cloud** plans
- Builds that benefit from faster hardware but want GitHub to manage the runner lifecycle

### Self-Hosted (Manual)

✅ Best for:

- Organizations with **existing on-prem infrastructure** they want to leverage
- Workloads requiring access to **air-gapped or highly restricted networks**
- Small-scale deployments (a few dedicated build machines)
- Teams with **specialized hardware** not available in GitHub-hosted options (e.g., specific ARM architectures, custom hardware)
- Strict **compliance and data sovereignty** requirements

### Self-Hosted (ARC / Kubernetes)

✅ Best for:

- Organizations already running **Kubernetes** that want to leverage existing clusters
- High build volumes requiring **dynamic autoscaling** of runners
- Teams that want **ephemeral, containerized** build environments at scale
- Enterprises needing to run hundreds or thousands of concurrent jobs cost-effectively
- Workloads requiring custom runner images with pre-baked toolchains

### Hybrid Approach

Most enterprise customers benefit from a **hybrid strategy**:

- Use **GitHub-hosted runners** (standard or larger) for general-purpose CI/CD workflows
- Use **self-hosted runners** for workloads requiring private network access, specialized hardware, or strict compliance controls
- Use **runner groups** to enforce which teams and repositories use which runner types

---

## Expected Customer Questions

### Cost & Licensing

1. **"How much will GitHub-hosted runners cost us at our current build volume?"**
   > Pull their current minutes usage and model it against GitHub plan inclusions and overage rates. Factor in standard vs. larger runner pricing.

2. **"Is it cheaper to run self-hosted runners on our existing infrastructure?"**
   > Compare per-minute GitHub costs against their infrastructure costs plus staff time for maintenance, patching, and monitoring.

3. **"Are there hidden costs with self-hosted runners?"**
   > Yes — OS patching, security hardening, monitoring, incident response, Kubernetes management (for ARC), and staff time are all costs that don't appear on a bill.

4. **"Do we get free minutes? How many? What happens when we exceed them?"**
   > Free minutes vary by plan. Overage is billed per-minute. Larger runners do not include free minutes.

5. **"What are larger runners and how are they priced?"**
   > Larger runners offer up to 64-core, 256 GB RAM, and GPU. They are available on Team and Enterprise Cloud plans and billed per-minute based on machine size.

6. **"Can we use larger GitHub-hosted runners with Azure VNET instead of building self-hosted infrastructure?"**
   > Yes. For many use cases that previously required self-hosted runners (private networking, static IPs), larger runners with VNET integration are now a managed alternative.

### Security & Compliance

7. **"Can GitHub-hosted runners access our private VNET or on-prem resources?"**
   > Standard runners cannot. Larger runners support Azure Private Networking via VNET integration. Additional patterns include OIDC, API gateways, and WireGuard tunnels.

8. **"How do we prevent secrets from leaking between jobs on self-hosted runners?"**
   > Use ephemeral runners (default in ARC). For persistent runners, implement cleanup steps and avoid storing secrets on disk.

9. **"Do GitHub-hosted runners meet SOC 2 / FedRAMP / HIPAA requirements?"**
   > Refer to GitHub's compliance documentation and security certifications. Self-hosted gives you full control if GitHub-hosted does not meet specific requirements.

10. **"Can we restrict which repos use which runners?"**
    > Yes. **Runner groups** allow you to control access at the enterprise and organization level, scoping runners to specific organizations or repositories.

11. **"How do we handle IP allow-listing for our firewalls?"**
    > Standard GitHub-hosted runners have dynamic IPs. Larger runners with VNET integration or self-hosted runners provide static/known IPs.

### Performance & Reliability

12. **"Will we experience queue delays with GitHub-hosted runners?"**
    > Possible during peak times. Larger runners with configured max concurrency can help. Self-hosted runners give you full control over capacity.

13. **"Can we get faster builds with self-hosted runners?"**
    > Potentially — via persistent caching, higher-spec hardware, or network proximity to dependencies. Larger GitHub-hosted runners also address the hardware gap.

14. **"What happens if a self-hosted runner goes offline mid-job?"**
    > The job will fail. With ARC, Kubernetes can reschedule pods. Proper monitoring and health checks are essential.

### Migration & Operations

15. **"How hard is it to migrate from GitHub-hosted to self-hosted (or vice versa)?"**
    > Workflow changes are minimal — primarily updating the `runs-on` label. The main effort is provisioning and configuring the runner infrastructure and ensuring tool compatibility.

16. **"Can we use a hybrid approach — some jobs on hosted, some on self-hosted?"**
    > Yes. This is the most common enterprise pattern. Use `runs-on` labels to target specific runner types per job.

17. **"Who is responsible for patching and updating self-hosted runner OS images?"**
    > You are. GitHub does not manage your self-hosted runner infrastructure.

18. **"How do we monitor runner health, utilization, and job metrics?"**
    > Use the GitHub Actions usage metrics in organization settings. For self-hosted, implement your own monitoring (Prometheus, Datadog, etc.) on the runner infrastructure.

### Architecture & Scaling

19. **"Should we use ephemeral self-hosted runners or persistent ones?"**
    > Ephemeral is recommended for security (clean environment each job). Persistent can improve performance via caching but requires hardening.

20. **"Can we run self-hosted runners in containers/Kubernetes?"**
    > Yes. ARC is the recommended approach. It uses Runner Scale Sets for Kubernetes-native autoscaling with ephemeral pods.

21. **"What is Actions Runner Controller (ARC) and should we use it?"**
    > ARC is a Kubernetes operator that orchestrates self-hosted runners. Use it if you have Kubernetes expertise and need dynamic autoscaling at scale.

22. **"What's the difference between ARC with Runner Scale Sets vs. the older listener mode?"**
    > Runner Scale Sets is the current recommended scaling mode. It provides more efficient scaling, better resource utilization, and is the actively developed approach.

23. **"What does GitHub actually support vs. what's on us with ARC?"**
    > GitHub supports the ARC controller and runner images. Your Kubernetes cluster (1.27+), networking, storage, and operational management are your responsibility.

24. **"Can we use runner groups to enforce which teams use which runner types?"**
    > Yes. Runner groups work at the enterprise level (scoped to organizations) and organization level (scoped to repositories) for both self-hosted and larger GitHub-hosted runners.

---

## Preparation Tips

- **Know the customer's current build volume.** Pull their minutes usage and concurrency data ahead of time so you can model cost scenarios for GitHub-hosted vs. self-hosted.
- **Understand their network topology.** If they have on-prem dependencies, private databases, or air-gapped environments, self-hosted runners (or larger runners with VNET) become much more relevant.
- **Have a hybrid recommendation ready.** Most enterprise customers end up using both GitHub-hosted and self-hosted runners. Be prepared to explain how runner groups and `runs-on` labels make this seamless.
- **Be ready to discuss ARC and Runner Scale Sets.** Kubernetes-based autoscaling is a common enterprise pattern. Know the requirements (K8s 1.27+, Helm 3) and the support boundaries.
- **Bring pricing calculators.** Have the [GitHub pricing page](https://github.com/pricing) and billing docs ready for included minutes per plan and per-minute overage rates.
- **Know the larger runner capabilities.** Many traditional reasons for self-hosted runners (private networking, static IPs, GPU, high memory) are now addressed by larger runners. This changes the conversation significantly.
- **Prepare a cost comparison template.** Build a simple spreadsheet or table comparing GitHub-hosted per-minute costs against estimated self-hosted infrastructure + staff costs at the customer's build volume.
- **Anticipate compliance questions.** Have GitHub's security certifications and compliance documentation bookmarked. Know when to recommend self-hosted for regulatory requirements GitHub-hosted cannot satisfy.

---

## Reference Documentation

| Topic | Link |
|---|---|
| GitHub-Hosted Runners | <https://docs.github.com/en/actions/concepts/runners/github-hosted-runners> |
| Self-Hosted Runners | <https://docs.github.com/en/actions/concepts/runners/self-hosted-runners> |
| Larger Runners | <https://docs.github.com/en/actions/concepts/runners/larger-runners> |
| Private Networking | <https://docs.github.com/en/actions/concepts/runners/private-networking> |
| Runner Groups | <https://docs.github.com/en/actions/concepts/runners/runner-groups> |
| Runner Scale Sets | <https://docs.github.com/en/actions/concepts/runners/runner-scale-sets> |
| Actions Runner Controller (ARC) | <https://docs.github.com/en/actions/concepts/runners/actions-runner-controller> |
| Support for ARC | <https://docs.github.com/en/actions/concepts/runners/support-for-arc> |
| GitHub Actions Pricing | <https://github.com/pricing> |