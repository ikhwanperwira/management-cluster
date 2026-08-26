# ADR-007 --- Pod CIDR Allocation

**Status:** Accepted\
**Date:** 2026-08-26

## Context

The cluster currently assigns individual /24 PodCIDRs to nodes.

**Observed allocation:**

  -----------------------------------------------------------------------
  **Node**                            **PodCIDR**
  ----------------------------------- -----------------------------------
  onprem-sea-00                       198.18.1.0/24

  euserv-eu-00                        198.18.2.0/24

  gcp-na-00                           198.18.3.0/24

  oci-sea-00                          198.18.4.0/24
  -----------------------------------------------------------------------

## Decision

The cluster uses node-specific /24 PodCIDRs within the 198.18.0.0/16 pod
network.

The existing PodCIDR assignments are part of the cluster\'s current
networking state.

## Consequences

-   Each node has an identifiable PodCIDR.

-   The unavailable euserv-eu-00 retains its previously assigned PodCIDR
    > in the Kubernetes inventory.

-   Pod IP addressing must not be confused with node InternalIP
    > addressing.
