# Argo CD Agent Mode — App of Apps

Deploys guestbook apps to HCP spoke clusters using Argo CD Agent **managed mode**.

## Architecture

```
┌─────────────────────────────────────────────────┐
│                 Hub / SNO                        │
│                                                  │
│  ┌────────────┐    ┌─────────────────────────┐   │
│  │ ACM        │    │ OpenShift GitOps         │   │
│  │            │    │                          │   │
│  │ cluster    │    │  Principal (gRPC:443)    │   │
│  │ secrets    │    │  controller: disabled    │   │
│  └─────┬──────┘    └───────┬─────────────────┘   │
│        │                   │                     │
│   agent-name label    mTLS │ gRPC                │
│        │                   │                     │
│  ┌─────┴───────────────────┴──────────────────┐  │
│  │  ns: hcp000          ns: hcp001            │  │
│  │  ├─ guestbook-hcp000 ├─ guestbook-hcp001   │  │
│  │  └─ guestbook-beta-  └─ guestbook-beta-    │  │
│  │       hcp000              hcp001            │  │
│  └────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
          │                        │
          ▼                        ▼
┌──────────────────┐    ┌──────────────────┐
│    HCP000        │    │    HCP001        │
│                  │    │                  │
│  Agent (managed) │    │  Agent (managed) │
│                  │    │                  │
│  dropzone/       │    │  dropzone/       │
│  dropzone-beta/  │    │  dropzone-beta/  │
└──────────────────┘    └──────────────────┘
```

Apps are plain `Application` resources (no ApplicationSets) placed in per-agent namespaces on the hub. The principal routes them to agents by namespace name. Agents sync locally and relay status back.

See `SKILL.md` for the full deployment runbook.
