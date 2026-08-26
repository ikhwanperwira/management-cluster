# ADR-002 --- Control-Plane Placement

**Status:** Accepted\
**Date:** 2026-08-26

## Context

gcp-na-00 and oci-sea-00 both operate Kubernetes control-plane
components and etcd.

Both nodes have:

-   node-role.kubernetes.io/control-plane=

-   node-role.kubernetes.io/etcd=

and both have:

-   node-role.kubernetes.io/control-plane:NoSchedule

The cluster therefore explicitly prevents ordinary workloads from being
scheduled onto control-plane nodes unless those workloads tolerate the
control-plane taint.

Several cluster-critical components currently tolerate this taint.

## Decision

Control-plane nodes are treated as dedicated control-plane
infrastructure by default.

The following nodes are control-plane nodes:

-   **gcp-na-00**

-   **oci-sea-00**

The standard control-plane taint remains the mechanism used to prevent
ordinary workloads from being scheduled there.

Cluster-system workloads may explicitly tolerate the control-plane taint
when their placement requires or permits execution on control-plane
nodes.

## Consequences

-   Ordinary application workloads do not automatically consume
    > control-plane capacity.

-   System workloads can intentionally run on control-plane nodes.

-   Scheduling behavior must be evaluated using both node taints and pod
    > tolerations.

-   A pod running on a control-plane node is not, by itself, evidence of
    > incorrect scheduling.
