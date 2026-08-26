# ADR-016 --- GitOps Ownership Model

**Status:** Accepted\
**Date:** 2026-08-26

## Context

FluxCD is installed in flux-system with:

-   source-controller

-   kustomize-controller

-   helm-controller

The Deployments contain Flux ownership labels and Kustomize Toolkit
metadata:

kustomize.toolkit.fluxcd.io/name=flux-system

kustomize.toolkit.fluxcd.io/namespace=flux-system

## Decision

FluxCD is treated as the cluster\'s GitOps control mechanism for
resources under its management.

The Flux controllers are themselves part of the cluster\'s system
infrastructure and are scheduled according to their Kubernetes
Deployment policies.

## Consequences

-   GitOps-managed resources should be understood as declaratively
    > owned.

-   Manual changes to GitOps-managed resources may be reconciled by
    > Flux.

-   Flux controller placement is a scheduling concern separate from
    > GitOps ownership.
