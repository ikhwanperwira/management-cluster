# ADR-008 --- FluxCD Controller Placement

**Status:** Accepted\
**Date:** 2026-08-26

## Context

Flux controllers currently run on oci-sea-00:

-   helm-controller

-   kustomize-controller

-   source-controller

Their common scheduling configuration is:

nodeSelector:

kubernetes.io/os: linux

tolerations:

\- key: node-role.kubernetes.io/control-plane

operator: Exists

effect: NoSchedule

There is no explicit nodeName, node affinity, or hostname selector in
their Deployment specifications.

## Decision

Flux controllers are permitted to run on Linux control-plane nodes by
virtue of tolerating the control-plane taint.

Their current placement on oci-sea-00 is therefore considered a valid
result of Kubernetes scheduling rather than an explicit hostname pinning
decision.

The Flux controllers are not considered statically bound to oci-sea-00.

## Consequences

-   Flux controller placement may change if scheduling conditions
    > change.

-   The Deployment configuration expresses eligibility rather than a
    > fixed node assignment.

-   Current pod placement should not be interpreted as hostname
    > affinity.
