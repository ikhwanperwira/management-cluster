# ADR-013 --- Container Runtime

**Status:** Accepted\
**Date:** 2026-08-26

## Context

The active nodes use CRI-O:

-   **gcp-na-00:** cri-o://1.35.1

-   **oci-sea-00:** cri-o://1.35.1

-   **onprem-sea-00:** cri-o://1.35.5

## Decision

CRI-O is the container runtime used by the active Kubernetes nodes.

Runtime version is tracked independently per node.

## Consequences

-   Container runtime behavior is based on CRI-O.

-   Runtime version differences between control-plane and worker nodes
    > are part of platform inventory.

-   Runtime version should be included in future compatibility analysis.
