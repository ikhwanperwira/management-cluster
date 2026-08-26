# ADR-014 --- Node Lifecycle and Decommissioning

**Status:** Accepted\
**Date:** 2026-08-26

## Context

euserv-eu-00 remains represented as a Kubernetes Node but is
unavailable:

-   **Ready:** Unknown

-   **Unschedulable:** true

-   It carries unreachable and unschedulable taints and has no running
    > pods.

## Decision

Node lifecycle state is distinguished from the existence of a Kubernetes
Node object.

A node can be:

-   active and schedulable,

-   active but unschedulable,

-   unreachable/unavailable,

-   historical/decommissioned.

euserv-eu-00 is currently classified as unavailable.

## Consequences

-   Cluster inventory must not equate \'Node object exists\' with \'node
    > is operational.\'
