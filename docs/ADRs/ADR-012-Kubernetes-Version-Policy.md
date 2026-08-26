# ADR-012 --- Kubernetes Version Policy

**Status:** Accepted\
**Date:** 2026-08-26

## Context

Current Kubernetes versions are:

-   **gcp-na-00:** v1.35.1

-   **oci-sea-00:** v1.35.1

-   **onprem-sea-00:** v1.35.6

The control-plane nodes are therefore on the same Kubernetes version,
while the worker is on a later patch release.

## Decision

The cluster records Kubernetes version at node level and treats
control-plane version consistency as a distinct architectural concern
from worker versioning.

The current versions are:

-   **Control plane:** v1.35.1

-   **Worker:** v1.35.6

## Consequences

-   Version differences between nodes must be considered when evaluating
    > cluster behavior, upgrades, and compatibility.
