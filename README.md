# Kustomize Lesson

**Kustomize** lets you customize raw, template-free YAML files for multiple purposes, leaving the original YAML untouched and usable as is.

A practical guide to managing Kubernetes configurations with Kustomize — covering everything from bases and overlays to advanced patching, generators, components, and transformers.
---

## Overview

This lesson is designed for **intermediate Kubernetes users** who are already comfortable with core K8s concepts (Deployments, Services, ConfigMaps, etc.) and want a structured, scalable way to manage manifests across multiple environments — without templating engines.

By the end of this lesson, you will be able to:

- Structure Kubernetes configs using **bases and overlays**
- Apply **strategic merge patches** and **JSON 6902 patches** to customize resources
- Generate **ConfigMaps and Secrets** dynamically from files and literals
- Use **components** to share reusable configuration chunks across overlays
- Apply **transformers** to consistently modify resources at scale

---

## Prerequisites

- Kubernetes cluster access (local cluster via `kind`, `minikube`, or similar is fine)
- `kubectl` v1.14+ (Kustomize is built in via `kubectl apply -k`)
- Optionally, the standalone [`kustomize` CLI](https://kubectl.docs.kubernetes.io/installation/kustomize/)
- Familiarity with YAML and basic Kubernetes resource types

---

## Topics Covered

### 1. Bases & Overlays

Learn how to separate your **base configuration** (shared across all environments) from **overlays** (environment-specific customizations like `dev`, `staging`, and `prod`).

```
k8s/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/
    │   └── kustomization.yaml
    └── prod/
        └── kustomization.yaml
```

Key concepts:
- `kustomization.yaml` structure and the `resources` field
- Referencing a base from an overlay
- Building and previewing output with `kubectl kustomize` or `kustomize build`

---

### 2. Patches — Strategic Merge & JSON 6902

Customize specific fields of base resources without duplicating entire manifests.

**Strategic Merge Patch** — patch using a partial resource definition:

```yaml
# overlays/prod/deployment-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 5
```

**JSON 6902 Patch** — precise, operation-based patching:

```yaml
# overlays/prod/kustomization.yaml
patches:
  - target:
      kind: Deployment
      name: my-app
    patch: |-
      - op: replace
        path: /spec/replicas
        value: 5
      - op: add
        path: /spec/template/spec/containers/0/env/-
        value:
          name: ENV
          value: production
```

---

### 3. ConfigMap & Secret Generators

Generate ConfigMaps and Secrets directly from files or literals — with automatic hash suffixes to trigger rolling updates when content changes.

**ConfigMap from literals:**

```yaml
configMapGenerator:
  - name: app-config
    literals:
      - LOG_LEVEL=info
      - CACHE_TTL=300
```

**Secret from a file:**

```yaml
secretGenerator:
  - name: db-credentials
    files:
      - secret.env
    type: Opaque
```

Key concepts:
- Hash suffix behavior and `disableNameSuffixHash`
- Referencing generated ConfigMaps/Secrets in Deployments

---

### 4. Components

**Components** allow you to define reusable, self-contained configuration chunks that can be mixed into multiple overlays — ideal for optional features like monitoring, autoscaling, or feature flags.

```
k8s/
└── components/
    └── autoscaling/
        ├── hpa.yaml
        └── kustomization.yaml
```

```yaml
# overlays/prod/kustomization.yaml
components:
  - ../../components/autoscaling
```

Key concepts:
- Difference between components and bases
- When to use components vs overlays

---

### 5. Transformers

Apply bulk transformations across all resources in a kustomization — such as adding labels, annotations, namespace, or name prefixes.

```yaml
# kustomization.yaml
namespace: production

namePrefix: prod-

commonLabels:
  app.kubernetes.io/env: production
  app.kubernetes.io/managed-by: kustomize

commonAnnotations:
  team: platform-engineering
```

Custom transformers using `replacements` for cross-resource field injection:

```yaml
replacements:
  - source:
      kind: ConfigMap
      name: app-config
      fieldPath: data.APP_VERSION
    targets:
      - select:
          kind: Deployment
        fieldPaths:
          - spec.template.spec.containers.[name=my-app].image
```

---

## Usage

Preview rendered output without applying:

```bash
kubectl kustomize overlays/prod
# or
kustomize build overlays/prod
```

Apply to your cluster:

```bash
kubectl apply -k overlays/prod
```

---

## NOTE for sample folder

```bash 
kubectl kustomize ... 
kubectl version --client 

# external
kustomize version



# to render the values 
kubectl kustomize overlays/prod
kubectl kustomize overlays/dev 
kubectl apply -k  overlays/dev
kubectl kustomize overlays/dev > rendered.yaml 
kubectl apply | replace -f rendered.yaml 

```


## NOTE for example-002
```bash 
helm create sample-chart
```

## Resources

- [Kustomize Official Docs](https://kubectl.docs.kubernetes.io/guides/introduction/kustomize/)
- [Kustomize GitHub](https://github.com/kubernetes-sigs/kustomize)
- [Kubernetes Reference — kustomization.yaml](https://kubectl.docs.kubernetes.io/references/kustomize/kustomization/)