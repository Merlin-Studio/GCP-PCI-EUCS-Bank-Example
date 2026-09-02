# A PCI + EUCS Bank Foundation on GCP — A Complete Example

---

## The Problem

Your team knows how to build infrastructure on GCP. But when you're asked to build the backend foundation for a European bank, the question changes shape: five compliance frameworks apply at once — PCI DSS for cardholder data, ISO 27001 for the ISMS, EUCS and GDPR for European data sovereignty, CIS as the technical baseline — and DORA sits above all of them asking whether the platform would actually survive a bad day.

Each framework has hundreds of requirements. Maybe forty of them change what your Terraform looks like. Knowing *which* forty — and being able to show an auditor why each value is set the way it is — is the actual work.

This repository is a complete working example for exactly that profile. Here's what's in it and how it was built.

## The Scenario

AcmeBank is a fictional EU bank building its GCP backend foundation:

- **Financial industry, 2000+ employees**, expert platform team, hybrid estate with on-premises connectivity
- **Frameworks**: PCI DSS, ISO 27001, CIS Benchmarks, EUCS, GDPR — with DORA's resilience obligations reflected in the DR and logging choices
- **Paris first**: europe-west9 primary, europe-west4 as a warm-standby DR region
- **Highly regulated data**, customer-managed encryption keys on every data service
- **Three environments** (dev / staging / prod), workloads on GKE, Cloud Run and Cloud Functions, a data platform on BigQuery, Cloud SQL, Cloud Storage, Pub/Sub and Dataflow

Stacking EUCS onto the framework list is what pushes the wizard into its **Advanced profile** — the configuration surface a bank actually needs: VPC Service Controls, per-service CMEK with dual keyrings, org-level policy enforcement, and a DR posture with real subnets in the second region.

## From Requirements to Configuration

Merlin Studio compiles compliance knowledge at design time, so generation is deterministic: the same answers always produce the same artifact, and every framework-driven value is traceable to the rule that set it.

The README's **"What fired"** table walks the eight controls a EU bank reviewer checks first — PCI scope segmentation, the EU region lock, CMEK, centralized egress, immutable audit logs, DLP, privileged access, and dual-region resilience — and points at the exact file where each landed. A few examples of the reasoning:

- **GDPR + EUCS** both demand European data residency, so `gcp.resourceLocations` is enforced org-wide with the `in:eu-locations` allow-list. Nothing can be created outside the EU, in any project, by anyone.
- **CIS GCP 1.10** wants keys rotated within 90 days; PCI and EUCS would accept a year. The strictest framework wins: every key in both keyrings rotates at 90 days.
- **PCI DSS Req 10** plus EUCS logging requirements drive 10-year audit retention — with the `audit-logs` bucket **locked**, so not even an org admin can shorten it after deployment.
- **Warm-standby DR** replicates every subnet into europe-west4 at a +128 CIDR offset and builds a full replica keyring there, so a regional failover doesn't strand encrypted data.

## A Tour of the Repository Outputs

### Architecture Diagram

`architecture.mmd` renders directly on GitHub — the org tree, hub-and-spoke networking across both regions, the security services, and the DR layout in one picture.

### Scorecards

`SECURITY_SCORECARD.md` (94/100, A-) and `ARCHITECTURE_SCORECARD.md` (100/100, A) grade the generated configuration against the selected frameworks, check by check, with the reasoning shown for every row.

### FAST YAML Files

The five stage directories — `org-setup/`, `networking/`, `security/`, `project-factory/`, `vpcsc/` — are data files for Google's [Cloud Foundation Fabric FAST](https://github.com/GoogleCloudPlatform/cloud-foundation-fabric/tree/master/fast) framework. They're not standalone Terraform; they plug into FAST's stages, and `$`-interpolation tokens resolve cross-stage references at plan time. Every YAML file carries a comment header naming the framework clauses it satisfies.

### CMEK Wiring

`CMEK_WIRING.md` lists the post-deployment IAM bindings that connect each service agent to its key — the step teams most often miss when they roll their own CMEK.

## Try It Yourself

This bundle came out of a wizard session, not a hand-crafted repo. Merlin is open — no signup, no email; guest mode lets you start immediately: [app.merlin-studio.cloud](https://app.merlin-studio.cloud). Pick your own frameworks and regions and compare what fires.
