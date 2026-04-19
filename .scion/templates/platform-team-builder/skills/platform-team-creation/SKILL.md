---
name: platform-team-creation
description: >-
  Interview a user about their cloud application's stack and produce a brief
  describing a focused platform engineering / SRE agent team, then hand the
  brief off to the team-creation skill which emits scion templates. Use when
  the user wants to bootstrap or maintain platform/SRE agents for a specific
  application — e.g. "set up a platform team for my service", "create SRE
  agents for our infra", "help me design ops automation for this stack".
---

# Platform Team Creation Skill

You design **focused platform-engineering and SRE agent teams** for a specific cloud application. You do this in two phases:

1. **Phase 1 (this skill)**: interview the user about their stack across every platform-engineering concern, then propose a role decomposition.
2. **Phase 2 (the `team-creation` skill)**: take a natural-language brief from this skill and emit the actual scion templates.

You are responsible only for phase 1: the interview, the role design, and the handoff. You MUST end by invoking the `team-creation` skill via the Skill tool with the synthesized brief — you do not write template files yourself.

## Why two phases

The `team-creation` skill knows how to format scion templates, status boilerplate, the orchestrator pattern, and skill attachment via `npx skills find <keyword>`. This skill knows how to interrogate a real platform stack and translate vendor choices into well-scoped agent roles. Keeping these separate means platform-team-creation can evolve independently of the template format, and team-creation stays generic enough to serve other team types (debate panels, code-review panels, etc.).

## Platform Engineering Concerns Inventory

This is the canonical inventory. Every interview MUST touch every category — even briefly — so you can mark it confirmed-present, confirmed-absent (N/A), or unknown. Do not skip categories silently; missed categories become missed agent responsibilities downstream.

1. **Compute & serving** — kubernetes (EKS/GKE/AKS/self-hosted), serverless (Lambda/Cloud Run/Functions), VMs, edge runtimes (Cloudflare Workers/Vercel), batch jobs, queue workers
2. **Data stores** — relational (Postgres/MySQL/Aurora/Cloud SQL), NoSQL (DynamoDB/Mongo/Cassandra), cache (Redis/Memcached), search (Elasticsearch/OpenSearch/Algolia), message queues (SQS/Kafka/RabbitMQ/Pub-Sub), object storage (S3/GCS/Blob), warehouse (BigQuery/Snowflake/Redshift)
3. **CI/CD** — build (GitHub Actions/CircleCI/GitLab/Buildkite/Jenkins), test orchestration, artifact storage, deploy mechanism (ArgoCD/Spinnaker/Flux/manual)
4. **IaC & config management** — terraform, pulumi, CDK, helm, kustomize, ansible, crossplane
5. **Networking & edge** — DNS, CDN (CloudFront/Fastly/Cloudflare), load balancers, service mesh (Istio/Linkerd/Consul), ingress controllers, API gateway, WAF
6. **Identity & access** — workforce IAM/SSO (Okta/Entra/Google Workspace), workload IAM (cloud-native, Vault, OIDC federation), RBAC, JIT access
7. **Secrets & runtime config** — vaults (HashiCorp Vault/Secrets Manager/Parameter Store/SOPS), rotation, env config, dynamic credentials
8. **Observability** — metrics (Prometheus/Datadog/CloudWatch/New Relic), logs (Loki/Splunk/ELK/Datadog), traces (Jaeger/Tempo/Datadog APM/Honeycomb), dashboards, alerting, SLOs/SLIs
9. **Incident response & on-call** — paging (PagerDuty/Opsgenie/Grafana OnCall), runbooks, blameless postmortems, status pages
10. **Security** — vuln scanning, SAST/DAST, supply-chain (SBOM/Sigstore/SLSA), image/registry scanning, policy enforcement (OPA/Kyverno/Conftest)
11. **Compliance** — SOC2, HIPAA, PCI, FedRAMP, ISO 27001, GDPR — declared scope drives audit posture and tooling requirements
12. **FinOps / cost** — tagging discipline, budgets, rightsizing tools (Kubecost/Vantage/native cost explorer), anomaly detection, savings plans
13. **Backup, DR & data lifecycle** — snapshots, RPO/RTO targets, retention policies, restore drills, cross-region replication
14. **Release management** — feature flags (LaunchDarkly/GrowthBook/Unleash/Flagsmith), progressive rollouts, canaries, blue/green
15. **Data pipelines** — ETL/ELT (Airflow/dbt/Dagster/Prefect), streaming (Kafka/Flink/Kinesis/Beam)
16. **ML/AI infra** — model serving, training infra, vector stores, eval harnesses, GPU pools (often N/A)
17. **Container/image supply chain** — registries (ECR/GAR/GHCR/Harbor), image signing (Cosign), provenance attestations
18. **Documentation & runbooks** — where ops knowledge lives (Notion/Confluence/repo READMEs/Backstage/internal wiki)

## Interview Methodology

Use a **conversational drill-down**, not a flat questionnaire. The user should feel they are having a working conversation, not filling out a form.

### Opening

Open with three context-setters in one message:
1. **App shape** — "What does the application do? Roughly what scale (users / req-per-sec / data volume)?"
2. **Cloud(s)** — "Which cloud provider(s)? Single-region, multi-region, or multi-cloud?"
3. **Maturity** — "Production today, pre-launch, or migrating from existing infra?"

These framings color every later answer. A pre-launch hobby app on Vercel does not need the same team as a multi-region production HIPAA workload on AWS.

### Drill-down

Walk the inventory in order. For each category:
- Ask one focused opening question that names the category
- If the user names a vendor or tool, branch into 1–3 targeted follow-ups specific to that vendor. Examples:
  - "Postgres on RDS" → Multi-AZ? Read replicas? What manages migrations? Who owns backup verification?
  - "GitHub Actions" → Self-hosted runners? OIDC to cloud? How is deploy gated?
  - "Datadog" → APM enabled? SLO monitors? Synthetic checks? Cost tier?
- If the user says "we don't have that" or "skip that" — record N/A and move on without belaboring the point
- If the answer is unclear, ask one clarifier; if still unclear, mark "unknown" and move on

You may collapse adjacent categories into one message when the user's stack is small (e.g. "Does N/A also apply to compliance, FinOps, ML infra, and data pipelines?"). Don't make a serverless side-project answer 18 separate questions.

### Confirmation

Once the inventory is captured, present a **role decomposition proposal** — typically 3–8 worker roles — for the user to confirm or revise. Each role should:
- Own a coherent slice of the platform (one role per major concern cluster, not one per vendor)
- Have clear inputs (what triggers its work) and outputs (what it produces / changes)
- Be sized so a single agent can reason about it without context overload

Common role archetypes — use the ones that fit, name them however suits the stack:
- `data-platform-engineer` — owns categories 2, 13, 15
- `ci-cd-engineer` — owns categories 3, 14, 17
- `infra-engineer` — owns categories 1, 4, 5
- `observability-engineer` — owns categories 8, 9
- `security-engineer` — owns categories 6, 7, 10, 11
- `cost-engineer` — owns category 12
- `docs-engineer` — owns category 18 (often folded into another role rather than standalone)

Do not invent roles for categories the user marked N/A. Do not split a role across two agents to give every category its own owner — bundle aggressively.

After the user confirms (or revises) the decomposition, proceed to handoff.

## Handoff to team-creation

End the skill by invoking the `team-creation` skill via the Skill tool, passing a **rich natural-language brief** as the args. The brief should:

- Open with one paragraph describing the application and its platform context (cloud, region topology, scale, maturity, compliance posture)
- Have one paragraph per proposed role describing:
  - Responsibilities and the specific vendors/tools that role owns
  - The workflows it participates in (deploy flow, incident response, capacity planning, etc.)
  - 2–4 keywords suitable for `npx skills find <keyword>` lookup (e.g. "terraform", "datadog", "argocd", "rds")
- Note any cross-role workflows (e.g. "incident-responder pages on observability-engineer's alerts and pulls in data-platform-engineer for DB-level incidents")
- Defer orchestrator design to team-creation — phase 2 may add an SRE-lead orchestrator, replace it with a custom one, or omit it entirely if the user prefers to drive workers directly

Do not produce template YAML or markdown yourself. The team-creation skill owns scion template structure, status boilerplate, and the orchestrator pattern.

### Skills attachment hint

For each role, mention 2–4 keywords suitable for `npx skills find <keyword>`. Phase 2 (team-creation) may use these to attach existing skills to each generated agent's template. You don't run skills.sh yourself, but good keyword hints make phase 2's job much easier.

## Worked Example

**User**: "Help me set up a platform team for my SaaS app — it's a Rails app on AWS with Postgres, deployed via GitHub Actions, monitored with Datadog."

You open with the three context questions and learn it's a 50-engineer B2B SaaS, single-region us-east-1, in production for 3 years, SOC2 Type II in scope. You then drill down through the inventory:

- Compute: ECS Fargate, no k8s, ~40 services
- Data: RDS Postgres Multi-AZ, ElastiCache Redis, S3 for uploads, no warehouse
- CI/CD: GitHub Actions → ECR → ECS deploy via CodeDeploy, OIDC to AWS
- IaC: Terraform in a separate repo, applied via Atlantis on PR merge
- Networking: ALB, CloudFront, Route53, no service mesh, AWS WAF
- Identity: Okta SSO for workforce, IAM roles for workloads, no Vault
- Secrets: AWS Secrets Manager, manual rotation
- Observability: Datadog (metrics, logs, APM), PagerDuty for paging, custom SLO dashboards
- Incident response: PagerDuty, Notion runbooks, blameless postmortems
- Security: Snyk for SAST, ECR vulnerability scanning, AWS WAF, no image signing yet
- Compliance: SOC2 Type II annual audit
- FinOps: native AWS Cost Explorer, no specialized tooling
- Backup/DR: RDS automated backups (35-day retention), no formal DR drill
- Release management: LaunchDarkly feature flags, no canary deploys
- Data pipelines: N/A (analytics goes to a separate Snowflake org owned by data team)
- ML infra: N/A
- Image supply chain: ECR, no signing
- Docs: Notion for runbooks, repo READMEs for service-level docs

You propose a 5-agent decomposition:
1. `data-platform-engineer` — RDS, ElastiCache, S3, backup/DR posture
2. `ci-cd-engineer` — GitHub Actions, ECR, CodeDeploy, LaunchDarkly
3. `infra-engineer` — ECS, Terraform/Atlantis, ALB, CloudFront, Route53, WAF
4. `observability-engineer` — Datadog, PagerDuty, runbooks, postmortems, SLO dashboards
5. `security-engineer` — Okta, IAM, Secrets Manager, Snyk, ECR scanning, SOC2 evidence

User confirms. You invoke the `team-creation` skill with a brief that opens "This is a SOC2 Type II B2B SaaS Rails app on AWS in us-east-1, 50 engineers, 3 years in production, ~40 ECS Fargate services…" and continues with one paragraph per role describing responsibilities, vendors, workflows, and `npx skills find` keyword hints (e.g. for `infra-engineer`: "terraform, ecs, atlantis, route53").

## Gotchas

- **Always touch every inventory category**, even just to mark N/A. Silent gaps lead to missing roles later.
- **Don't proliferate roles** — one agent per major slice of the platform, not one per vendor. A team with 12 narrow agents is harder to use than one with 5 well-scoped agents.
- **Don't write templates yourself.** This skill stops at the handoff. The team-creation skill owns scion template structure, orchestrator pattern, and status boilerplate.
- **Confirm before handoff.** Always show the proposed role decomposition to the user before invoking team-creation — bad role decomposition is hard to fix once templates exist.
- **Don't assume orchestrator.** Whether and how to add an SRE-lead orchestrator is a phase-2 decision the user may want to make differently each time. Mention it in the brief but don't prescribe.
- **Maturity matters.** A pre-launch app's "platform team" may be a single broad agent. A regulated production stack may need 7+ specialists. Let the answers to the opening context-setters scale the proposal.
- **Compliance scope is load-bearing.** SOC2/HIPAA/PCI/FedRAMP changes which categories require dedicated ownership (especially security, secrets, identity, observability, backup/DR). Always re-check the proposal against declared compliance scope before handoff.
