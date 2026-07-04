# LO6 review notes

I reviewed every main LO6 answer and the extra questions for technical correctness and exam usefulness.

Changes made in this reviewed version:

1. Clarified OpenShift logging wording: OpenShift has built-in monitoring, while logging is usually handled through platform/operator integrations.
2. Clarified StatefulSet ordering: ordered startup/shutdown is the default behavior, not the only possible behavior.
3. Clarified Kubernetes Secrets: base64 is encoding, not encryption; etcd encryption at rest and RBAC are needed for stronger protection.
4. Clarified Kubernetes rescheduling: it depends on controller-managed workloads, healthy nodes, and available capacity.
5. Clarified PVC portability: rescheduling with persistent storage depends on the storage backend and access mode.
6. Clarified Cluster Autoscaler: it needs supported cloud/infrastructure integration.
7. Improved OpenShift enterprise/community wording: OpenShift has Red Hat enterprise support and the OKD/OpenShift/Kubernetes ecosystem.
8. Removed or softened claims that were too broad, such as CDN being absolutely essential in all cases.

Result: the document is more precise, still simple, and better suited for full-mark theory answers.
