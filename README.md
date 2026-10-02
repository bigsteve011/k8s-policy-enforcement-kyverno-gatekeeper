# GitOps Policy Enforcement: Hardening Kubernetes Clusters with OPA Gatekeeper & Kyverno Admission Controllers

> Hardening multi-tenant Kubernetes clusters by enforcing Pod Security Standards (PSS) at admission time, mutating incoming workloads with secure defaults, and validating container provenance before scheduling.

Companion repository for the article **[GitOps Policy Enforcement: Hardening Kubernetes Clusters with OPA Gatekeeper & Kyverno Admission Controllers](https://stephen-omowumi.pages.dev/articles/devops-kubernetes-opa-kyverno)** by Stephen Omowumi.

## The problem

Default Kubernetes configurations permit root container execution, host namespace sharing, and unbounded resource consumption, allowing compromised pods to achieve node takeover and lateral container breakout.

## Solution architecture

Deploy Kyverno and OPA Gatekeeper as validating and mutating Webhooks. Enforce baseline and restricted Pod Security Standards cluster-wide. Automatically inject non-root security contexts, read-only root filesystems, and network policy labels upon deployment.

## How it works

1. **Kubernetes Admission API Interception** — Kubernetes API server intercepts workload manifests and transmits admission reviews to Kyverno.
2. **Policy Evaluation & Mutation** — Evaluate manifests against restricted security standards; inject drop ALL capabilities and runAsNonRoot: true.
3. **Image Registry & Signature Verification** — Confirm container image origin from approved corporate registry and verify Sigstore Cosign signature.
4. **Audit Logging & Violation Routing** — Denied deployments stream structured logs to Microsoft Sentinel / Datadog for security operations alert triage.

## Repository layout

```
kyverno/policies/      restrict privileged/root containers, require signed-registry images pinned by digest
kyverno/tests/         Kyverno CLI test suite (compliant pod passes, privileged pod is blocked)
gatekeeper/            OPA Gatekeeper ConstraintTemplate + Constraint (required governance labels)
test-resources/        sample pods used by the tests
```

## Usage

Test locally:
```bash
kyverno test kyverno/tests
```
Apply to a cluster (via Argo CD / Flux in a GitOps setup, or directly):
```bash
kubectl apply -f kyverno/policies/
kubectl apply -f gatekeeper/templates/ && kubectl apply -f gatekeeper/constraints/
```
Replace `ghcr.io/your-org/` in `require-trusted-registry.yaml` with your registry. Start with `validationFailureAction: Audit` on existing clusters, then switch to `Enforce`.

## Outcomes (as described in the article)

| Measure | Result |
|---|---|
| Privileged Pods | 0 Allowed |
| Admission Latency | <12ms |
| CIS K8s Compliance | 98.5% |

## Key takeaways

- Eliminates container breakout risk by enforcing readOnlyRootFilesystem and non-root execution
- Admission controllers block vulnerable or unsigned images before they consume cluster resources
- Provides declarative, auditable compliance aligned with CIS Kubernetes Benchmark v1.8

## Notes

This is a reference implementation. Names, account IDs, regions and allow-lists are placeholders: review and adapt them, and test in a non-production environment before rolling out.

## License

MIT — see [LICENSE](LICENSE).
