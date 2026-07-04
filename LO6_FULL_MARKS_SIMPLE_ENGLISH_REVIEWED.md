# LO6 — Full-mark theory answers in simple English

**Purpose:** These answers are written for an Intro to DevOps exam. They use simple English, but they keep the technical meaning correct.  

**Reviewed version:** I checked the answers again for exam use. I removed broad claims, clarified edge cases, and kept the English simple. The answers are written to be safe for theory questions, but the professor can still grade by his own rubric.

**How to use:** If the exam question is the same, copy the answer and adapt names/examples if needed. If the professor changes the wording, keep the same idea: compare both sides, mention the trade-off, and finish with a clear recommendation.

---

## LO6.1 — Kubernetes networking model vs Docker networking

**Question:** Analyze the networking model of Kubernetes versus Docker's network.

**Answer:**  
Docker networking is usually simpler and more host-based. On one host, containers often use a bridge network, NAT, and port mapping. If I want to reach a Docker container from outside, I usually publish a port from the host to the container, for example host port 8080 to container port 80.

Kubernetes networking is designed for a cluster, not only one host. Each Pod normally gets its own IP address, and Pods can communicate with other Pods across nodes. Kubernetes also uses Services to give applications a stable virtual endpoint, because Pod IPs can change when Pods are restarted. DNS, usually through CoreDNS, allows applications to reach a Service by name instead of by IP. The real network implementation is done by a CNI plugin.

Docker networking is easier for local development and single-host containers. Kubernetes networking is better for microservices, service discovery, scaling, and multi-node applications.

**Key points for full marks:**
- Docker: bridge, NAT, port mapping, single-host focus.
- Kubernetes: Pod IPs, Services, DNS, CNI, cluster-wide model.
- Kubernetes is more complex but better for distributed applications.

---

## LO6.2 — Kubernetes storage abstraction vs Podman storage

**Question:** Evaluate Kubernetes storage abstraction (CSI, StorageClasses, dynamic provisioning) against Podman's storage plugins. Which is more flexible for stateful workloads, and why?

**Answer:**  
Podman storage is good for local containers on one machine. It can use local volumes and bind mounts, and this is enough for development, testing, or simple single-host applications. But Podman does not manage storage across a whole cluster in the same way Kubernetes does.

Kubernetes has a stronger storage abstraction. Applications request storage through PersistentVolumeClaims. A StorageClass describes the type of storage, and CSI drivers connect Kubernetes to different storage backends, such as cloud disks, network storage, or enterprise storage systems. With dynamic provisioning, Kubernetes can create the volume automatically when a PVC is created.

For serious stateful workloads, Kubernetes is more flexible because it separates the application from the storage implementation. The same application can request storage in a declarative way, while the cluster decides how to provision it. This is especially useful with StatefulSets, databases, and applications that need persistent data.

**Key points for full marks:**
- Podman is good for local/single-host storage.
- Kubernetes uses PV, PVC, StorageClass, CSI, and dynamic provisioning.
- Kubernetes is more flexible for stateful workloads in a cluster.

---

## LO6.3 — Operator pattern vs only Kubernetes manifests

**Question:** Evaluate the operator pattern (CRDs + controllers) for managing stateful services versus using only Kubernetes manifests, in terms of control and effort.

**Answer:**  
Normal Kubernetes manifests define the desired state of objects like Deployments, Services, StatefulSets, and PVCs. This is enough for many simple applications. For example, I can define how many replicas I want and which image should run.

For complex stateful services, normal manifests are often not enough. Databases and similar systems need application-specific operations such as backup, restore, failover, upgrades, user management, cluster rebalancing, and safe scaling. An Operator solves this by using Custom Resource Definitions and a controller. The user creates a higher-level custom object, and the Operator watches it and performs the needed actions.

The advantage of Operators is more automation and better control over complex lifecycle tasks. The disadvantage is more complexity, because the Operator itself must be installed, trusted, upgraded, and understood. A bad Operator can also create problems.

**Conclusion:** For simple workloads, normal manifests are enough. For complex stateful services, an Operator is usually better because it automates expert operational knowledge.

**Key points for full marks:**
- Manifests define desired state but have limited application logic.
- Operators use CRDs and controllers.
- Operators help with backup, failover, upgrades, and scaling.
- More automation, but also more complexity.

---

## LO6.4 — Plain Kubernetes vs OpenShift S2I and developer console

**Question:** Assess the developer experience of plain Kubernetes (kubectl + manifests) versus OpenShift's Source-to-Image (S2I) and developer console. What does each optimize for?

**Answer:**  
Plain Kubernetes usually requires the developer or DevOps engineer to work with kubectl, YAML manifests, container images, registries, Services, and Ingress. This gives a lot of control and portability, because standard Kubernetes objects can run on many Kubernetes distributions. However, it has a steeper learning curve, especially for developers who only want to deploy code.

OpenShift improves the developer experience by adding tools on top of Kubernetes. Source-to-Image can build a runnable container image directly from source code and a builder image. The developer console gives a graphical interface for creating, deploying, and observing applications. This means developers can ship applications with less manual work around image building and YAML, while still using Kubernetes objects underneath.

Plain Kubernetes optimizes for flexibility, portability, and control. OpenShift optimizes for productivity, faster onboarding, integrated workflows, and enterprise support. The trade-off is that OpenShift adds extra platform concepts and more dependency on Red Hat tools.

**Key points for full marks:**
- Kubernetes: kubectl + YAML, flexible, portable, but harder.
- OpenShift: S2I + console, easier for developers, integrated workflow.
- Kubernetes optimizes control; OpenShift optimizes productivity and enterprise integration.

---

## LO6.5 — Migrating a workload from OpenShift to Kubernetes

**Question:** Critically evaluate migrating a workload from OpenShift to Kubernetes.

**Answer:**  
Migrating from OpenShift to plain Kubernetes is possible because OpenShift is built on Kubernetes. Basic Kubernetes objects like Deployments, Services, ConfigMaps, Secrets, and PVCs can often be reused or adapted. However, the migration is not automatic if the application uses OpenShift-specific features.

The main problem is replacing OpenShift-specific objects and workflows. For example, OpenShift Routes may need to become Kubernetes Ingress resources or another external access solution. BuildConfigs, ImageStreams, and S2I pipelines may need to be replaced with a normal CI/CD pipeline. Security Context Constraints must be mapped to Kubernetes security mechanisms such as Pod Security Admission, RBAC, and admission policies.

The benefit of migration is better portability and less dependency on Red Hat. It can also reduce license cost if the team can operate Kubernetes by itself. The disadvantage is losing integrated OpenShift features, vendor support, and some default security and developer tooling.

**Conclusion:** I would migrate only if portability, cost, or platform strategy are more important than OpenShift integration. The team must plan the replacement of Routes, Builds, ImageStreams, SCC, monitoring, and CI/CD.

**Key points for full marks:**
- OpenShift is Kubernetes-based, but not identical.
- OpenShift-specific objects must be replaced.
- Migration improves portability but can lose support and integrated tooling.

---

## LO6.6 — CI/CD build agents in Kubernetes vs dedicated VMs

**Question:** Evaluate running CI/CD build agents (Jenkins agents, GitLab runners, Tekton) inside Kubernetes versus on dedicated VMs, considering isolation, autoscaling, and noisy-neighbor effects.

**Answer:**  
Running CI/CD build agents inside Kubernetes is useful because agents can be created as Pods only when jobs are needed. After the job finishes, the Pod can be removed. This gives good elasticity and can reduce wasted resources. It also gives isolation between jobs, because each job can run in a separate Pod with its own resource limits and service account.

Dedicated VMs are simpler and sometimes more predictable. They can be better for legacy build tools, special hardware, or jobs that need a very stable environment. The problem is that VMs can sit idle when there are no builds, and scaling them up and down is slower than starting Pods.

The risk in Kubernetes is the noisy-neighbor problem. Heavy builds can use too much CPU, memory, disk I/O, or network and affect other workloads. This should be controlled with requests, limits, quotas, separate node pools, and good monitoring.

**Conclusion:** For modern and dynamic CI/CD, I would prefer Kubernetes agents. For legacy or special build environments, dedicated VMs can still be a safer and simpler option.

**Key points for full marks:**
- Kubernetes agents: elastic, disposable, scalable, isolated by Pod.
- VMs: simpler, stable, good for legacy/special builds.
- Noisy-neighbor risk must be controlled with limits and node separation.

---

## LO6.7 — Kubernetes cloud integration and dynamic capacity provisioning

**Question:** How does Kubernetes integrate with cloud environments and dynamic capacity provisioning?

**Answer:**  
Kubernetes integrates with cloud environments through cloud controllers, storage drivers, load balancers, and autoscaling components. For example, a Service of type LoadBalancer can create a cloud load balancer. A PersistentVolumeClaim can use a StorageClass to create a cloud disk automatically. This allows applications to request infrastructure in a declarative way.

Kubernetes can also support dynamic capacity. The Horizontal Pod Autoscaler can increase or decrease the number of Pods based on metrics such as CPU or custom metrics. The Cluster Autoscaler can add or remove worker nodes when the cluster and cloud or infrastructure provider support node autoscaling. In managed Kubernetes services, the cloud provider handles part of the control plane and node integration.

The benefit is that applications can scale with demand. The risk is cost and complexity. If autoscaling is not controlled with requests, limits, quotas, and monitoring, the cluster can become expensive or unstable.

**Key points for full marks:**
- Cloud load balancers through Service type LoadBalancer.
- Cloud disks through PVC, StorageClass, and CSI.
- HPA scales Pods; Cluster Autoscaler scales nodes.
- Good monitoring and cost control are required.

---

## LO6.8 — OpenShift security and hardening vs vanilla Kubernetes

**Question:** Critically evaluate the default security and hardening of OpenShift versus vanilla Kubernetes. Why do enterprises in regulated industries often choose OpenShift?

**Answer:**  
OpenShift is usually more opinionated and stricter by default than vanilla Kubernetes. It includes enterprise security features such as Security Context Constraints, integrated RBAC, stronger default restrictions, image registry integration, built-in cluster monitoring, optional logging integrations, and Red Hat support. OpenShift often prevents insecure container behavior by default, for example running containers as privileged users without proper permission.

Vanilla Kubernetes is very flexible, but many security controls must be designed and configured by the organization. The team must configure RBAC, admission control, network policies, image scanning, audit logging, secrets management, and cluster hardening. This gives freedom, but it also creates more responsibility and more chances for misconfiguration.

Enterprises in regulated industries often choose OpenShift because they need standardization, vendor support, lifecycle management, auditability, and safer defaults. It does not remove the need for security work, but it gives a stronger integrated platform.

**Conclusion:** OpenShift is often better for regulated enterprise environments because it provides stronger defaults and support. Vanilla Kubernetes is flexible, but it requires more internal expertise to harden correctly.

**Key points for full marks:**
- OpenShift: stricter defaults, SCC, support, integrated security tooling.
- Kubernetes: flexible but needs more manual hardening.
- Regulated industries value support, auditability, and standardization.

---

## LO6.9 — Learning curve and skill set: OpenShift vs plain Kubernetes

**Question:** Critically evaluate the learning curve and required skill set of OpenShift versus plain Kubernetes. Does OpenShift's added abstraction help or hinder a team new to containers?

**Answer:**  
Plain Kubernetes has a steep learning curve. A team must understand Pods, Deployments, Services, Ingress, storage, YAML, labels, selectors, RBAC, security, and troubleshooting. This is powerful, but it can be hard for a team that is new to containers.

OpenShift adds more abstractions and tools, such as the developer console, Routes, S2I, Projects, and stricter security defaults. These tools can help beginners because they make common tasks easier and more guided. Developers can deploy code faster without learning every Kubernetes detail immediately.

However, OpenShift can also hinder learning if the team only uses the graphical tools and never understands the Kubernetes objects underneath. The team must eventually learn both Kubernetes basics and OpenShift-specific concepts. That means OpenShift can reduce the first barrier, but it can increase the total platform knowledge needed later.

**Conclusion:** OpenShift helps a new team start faster, but it does not replace the need to learn Kubernetes fundamentals.

**Key points for full marks:**
- Kubernetes: harder start, more direct control.
- OpenShift: easier onboarding, more built-in tools.
- Long term, the team must learn both Kubernetes and OpenShift concepts.

---

## LO6.10 — Full Kubernetes vs k3s for small on-prem deployment

**Question:** Compare the resource footprint of full Kubernetes versus k3s for a small on-premise deployment. When is full Kubernetes overkill?

**Answer:**  
Full Kubernetes has more components and usually needs more CPU, memory, storage, and operational knowledge. It is suitable for larger production environments where the team needs full control, high availability, many integrations, and enterprise features.

k3s is a lightweight Kubernetes distribution. It is designed for smaller environments, edge computing, labs, and small on-premise deployments. It has a smaller resource footprint and is easier to install and operate. It still provides Kubernetes APIs, but with a simpler and lighter setup.

Full Kubernetes is overkill when the team has only a few services, limited hardware, no need for a complex multi-node cluster, and no dedicated platform team. In that case, k3s can provide enough orchestration with less overhead.

**Conclusion:** I would choose k3s for small on-prem or edge deployments, and full Kubernetes only when the environment really needs its scale and complexity.

**Key points for full marks:**
- Full Kubernetes: heavier, more complex, good for larger production.
- k3s: lightweight, easier, good for small/edge/on-prem.
- Full Kubernetes is overkill for small teams and simple workloads.

---

## LO6.11 — Docker Compose on one host vs Kubernetes for a small team

**Question:** Defend whether a small team should use Docker Compose on a single host or move straight to Kubernetes, considering complexity, scaling needs, and resilience.

**Answer:**  
For a small team with a simple application, Docker Compose on one host is often the better first choice. It is easy to understand, cheap to operate, and fast to set up. The team can define multiple containers in one compose file and ship the product without learning a full orchestration platform.

Kubernetes gives stronger features: self-healing, rolling updates, service discovery, scaling, resource management, and better resilience across nodes. But these benefits come with a lot of operational complexity. The team must understand manifests, networking, storage, monitoring, security, upgrades, and troubleshooting.

If the application has low traffic, few services, and no strict high availability requirement, Compose is enough. If the application needs multiple replicas, automatic recovery, zero-downtime updates, and growth across nodes, Kubernetes becomes more reasonable.

**Conclusion:** I would start with Docker Compose for a small team unless there is a clear need for Kubernetes features. Moving to Kubernetes should be justified by real scaling and resilience requirements.

**Key points for full marks:**
- Compose: simple, cheap, good for single host.
- Kubernetes: scalable and resilient, but complex.
- The best choice depends on real requirements, not hype.

---

## LO6.12 — Vendor lock-in: Kubernetes vs OpenShift

**Question:** Analyze vendor lock-in across Kubernetes (open source / CNCF) and OpenShift (Red Hat). How does the choice affect long-term flexibility?

**Answer:**  
Kubernetes is open source and is the common standard for container orchestration. If a company uses mostly standard Kubernetes objects, it can move more easily between different Kubernetes distributions or cloud providers. This gives better long-term flexibility.

OpenShift is built on Kubernetes, but it adds Red Hat-specific features such as Routes, BuildConfigs, ImageStreams, S2I, and Security Context Constraints. These features are useful and can speed up development and operations, but they can make migration harder because another Kubernetes platform may not have the same objects.

The trade-off is between portability and integrated support. Kubernetes gives more flexibility and less platform lock-in. OpenShift gives more built-in functionality, stronger enterprise support, and a more complete platform, but with more dependency on Red Hat.

**Conclusion:** If long-term portability is the main goal, plain Kubernetes with standard objects is better. If enterprise support and integrated tooling are more important, OpenShift can be worth the lock-in risk.

**Key points for full marks:**
- Kubernetes: open standard, more portable.
- OpenShift: Kubernetes-based but has Red Hat-specific features.
- OpenShift increases integration and support, but can increase lock-in.

---

## LO6.13 — Recommend OpenShift over vanilla Kubernetes

**Question:** Defend a recommendation of OpenShift over vanilla Kubernetes for a company that wants an integrated, vendor-supported platform with built-in developer and operations tooling.

**Answer:**  
I would recommend OpenShift in this case because the company wants an integrated and vendor-supported platform, not only the basic Kubernetes engine. OpenShift includes Kubernetes, but adds enterprise features around it, such as the developer console, Routes, S2I/build workflows, integrated registry, monitoring, logging, stricter security defaults, and Red Hat support.

With vanilla Kubernetes, the company must choose, install, integrate, secure, and maintain many separate tools. This can be powerful, but it needs more internal expertise and more operational work. OpenShift reduces this work by providing a more complete platform with tested integrations.

The disadvantages are higher cost, higher resource usage, and more dependency on Red Hat. However, for a company that values support, standardization, governance, and faster developer onboarding, those disadvantages can be acceptable.

**Conclusion:** I would recommend OpenShift over vanilla Kubernetes when the company wants an enterprise platform with support and built-in tooling, and not only a raw orchestration layer.

**Key points for full marks:**
- OpenShift = Kubernetes + enterprise platform features.
- Better support, security defaults, developer tools, operations tools.
- More cost and lock-in, but less integration work.

---

## LO6.14 — Kubernetes storage provisioning vs regular VM storage

**Question:** Compare storage provisioning between Kubernetes and regular VM workloads.

**Answer:**  
In regular VM workloads, storage is often provisioned manually. An administrator creates a disk, attaches it to a VM, formats it, mounts it, and configures the application to use it. This is simple for traditional applications, but it is less flexible when applications move or scale.

In Kubernetes, storage is requested declaratively. A Pod does not usually request a disk directly. Instead, the application uses a PersistentVolumeClaim. The cluster can bind that claim to an existing PersistentVolume or create a new volume dynamically through a StorageClass and CSI driver.

This makes Kubernetes storage better for dynamic and cloud-native environments. The workload can be rescheduled to another node while keeping its persistent storage attached through Kubernetes mechanisms, if the storage backend supports this behavior. However, stateful workloads still require careful planning because not all storage supports all access modes, performance levels, or multi-node access.

**Conclusion:** VM storage is simpler and familiar for traditional workloads. Kubernetes storage is more flexible and automated for containerized applications, but it needs correct StorageClass, access mode, and backup design.

**Key points for full marks:**
- VM storage: manual disk attach/mount/configure.
- Kubernetes storage: PVC, PV, StorageClass, CSI.
- Kubernetes is more dynamic, but stateful workloads still need careful design.

---

## LO6.15 — Startup with two engineers: recommend a container solution

**Question:** A startup with two engineers must ship a containerized product quickly and cheaply. Recommend container solution and justify your choice on cost and operational overhead.

**Answer:**  
For a startup with only two engineers, I would not start with self-managed Kubernetes unless there is a strong reason. Self-managed Kubernetes has a lot of operational overhead: cluster setup, upgrades, networking, storage, monitoring, security, backups, and troubleshooting. This can take time away from building the product.

If the product is simple, I would recommend Docker Compose on a single VM or a simple managed container/PaaS platform. Compose is cheap, fast, and easy to understand. It is good for an MVP or early production if the traffic is small and high availability is not critical.

If the startup expects fast growth or needs high availability, I would choose a managed platform, such as managed Kubernetes or another managed container service. This gives scaling options without forcing the two engineers to operate the whole control plane.

**Conclusion:** For fastest and cheapest delivery, I would start with Docker Compose or a managed container platform. I would avoid self-managed Kubernetes at the beginning because the operational overhead is too high for two engineers.

**Key points for full marks:**
- Small team should minimize operational overhead.
- Compose or PaaS is good for MVP.
- Managed Kubernetes can be used later if scaling is needed.
- Self-managed Kubernetes is usually too much at the start.

---

## LO6.16 — Docker vs Podman: daemon vs daemonless and rootless security

**Question:** Compare Docker and Podman in terms of architecture (daemon vs daemonless) and rootless security. Why might a security-conscious team prefer Podman?

**Answer:**  
Docker traditionally uses a central daemon. The Docker client talks to the Docker daemon, and the daemon manages images, containers, networks, and volumes. This model is common and easy to use, but the daemon is a sensitive component because it often runs with high privileges.

Podman is daemonless. It does not need one central long-running daemon to manage containers. Podman can also run containers rootless, meaning a normal user can run containers without giving the container runtime root privileges on the host. This reduces the damage if a container or runtime process is compromised.

A security-conscious team might prefer Podman because daemonless and rootless operation can reduce privilege and attack surface. Podman also works well in Linux environments where users should not have access to a powerful root-level Docker socket.

**Conclusion:** Docker is very popular and easy to use, but Podman has a security advantage in environments that care about rootless containers and avoiding a privileged central daemon.

**Key points for full marks:**
- Docker: client-server model with daemon.
- Podman: daemonless.
- Podman supports rootless containers well.
- Security benefit: less privilege and smaller attack surface.

---

## LO6.17 — Self-managed Kubernetes vs managed Kubernetes service

**Question:** Compare the day-2 operational overhead (cluster upgrades, scaling nodes, backups) of self-managed Kubernetes versus a managed Kubernetes service.

**Answer:**  
Self-managed Kubernetes gives the team the most control, but it also creates the most day-2 operational work. The team must handle cluster upgrades, control plane health, etcd backup and restore, node scaling, patching, networking, storage, monitoring, security hardening, and incident response.

A managed Kubernetes service reduces this overhead because the cloud provider manages at least part of the control plane and integrates the cluster with cloud services. This makes upgrades, availability, load balancers, and storage integration easier. The team can focus more on applications and less on operating the cluster.

The trade-off is less control and more dependency on the cloud provider. Some settings may be limited, and the company can become more tied to one cloud ecosystem.

**Conclusion:** For most teams, managed Kubernetes is better because it reduces day-2 operational overhead. Self-managed Kubernetes is only worth it when the company needs special control, on-prem operation, or has a skilled platform team.

**Key points for full marks:**
- Self-managed: maximum control, maximum operational work.
- Managed: less control, less overhead, cloud integration.
- Day-2 operations include upgrades, backups, scaling, monitoring, security.

---

## LO6.18 — Container solution for spiky global media-streaming traffic

**Question:** A media-streaming company expects spiky, unpredictable global traffic. Recommend a containerized solution. Elaborate your choice.

**Answer:**  
For spiky and global media-streaming traffic, I would recommend a cloud-native solution using managed Kubernetes, autoscaling, global load balancing, CDN, and strong monitoring. Managed Kubernetes can run the application services and scale them horizontally when traffic increases. A Cluster Autoscaler can also add nodes when the cluster needs more capacity.

However, video or media traffic should not be served only from application Pods. A CDN is important because it caches content closer to users around the world. This reduces latency, lowers load on the origin systems, and handles large traffic spikes better.

The system should also use metrics, logging, alerts, and resource limits. Without monitoring and limits, autoscaling can become expensive or unstable. Multi-region design may be needed if the company requires global availability.

**Conclusion:** I would use managed Kubernetes for application orchestration, plus CDN and global load balancing for media delivery. Kubernetes alone is not enough for global streaming; it must be combined with cloud and CDN services.

**Key points for full marks:**
- Managed Kubernetes for scalable services.
- HPA and Cluster Autoscaler for spikes.
- CDN is important for global media content.
- Monitoring and cost control are required.

---

## LO6.19 — Outgrown Docker Swarm or Compose: migrate to Kubernetes?

**Question:** Defend whether a company that has outgrown Docker Swarm (or a single Compose host) should migrate to Kubernetes — weigh the migration effort against the long-term benefits.

**Answer:**  
If a company has outgrown Docker Swarm or a single Compose host, Kubernetes can be a good next step. Kubernetes gives better support for scaling, self-healing, rolling updates, service discovery, storage integration, secrets/config management, and a large ecosystem. It is also widely supported in cloud and enterprise environments.

However, migration is not free. The team must convert Compose or Swarm definitions into Kubernetes manifests or Helm charts. They must also design networking, ingress, persistent storage, monitoring, logging, security, and CI/CD. Developers and operators need training.

The decision should depend on whether the long-term benefits are worth the migration cost. If the system has many services, needs high availability, and must scale across nodes, Kubernetes is worth considering. If the workload is still small and stable, Compose may still be enough.

**Conclusion:** I would migrate to Kubernetes when the current platform is blocking scaling, reliability, or operations. I would not migrate only because Kubernetes is popular.

**Key points for full marks:**
- Kubernetes gives scaling, resilience, ecosystem, standardization.
- Migration requires manifests, training, monitoring, storage, security.
- Migrate only when benefits outweigh complexity.

---

## LO6.20 — Choose Kubernetes for a microservices architecture?

**Question:** Would you choose Kubernetes as a solution for a microservices architecture, focus on its orchestration, resilience and ecosystem. Elaborate your standing with arguments.

**Answer:**  
Yes, I would choose Kubernetes for a serious microservices architecture. Microservices usually have many independent services, and Kubernetes is good at running many containers across a cluster. It provides Deployments, Services, ConfigMaps, Secrets, rolling updates, self-healing, and scaling.

Kubernetes improves resilience because if a Pod crashes, the controller can create a new one. If a node fails, workloads can be rescheduled on other healthy nodes if there is enough capacity and the workload is managed by a controller. Services give stable access to changing Pods, and readiness probes help avoid sending traffic to unhealthy Pods.

The ecosystem is also a strong reason. Kubernetes works with tools for ingress, service mesh, monitoring, logging, CI/CD, policy, and security. The main disadvantage is complexity. For a very small application with only one or two services, Kubernetes may be too much.

**Conclusion:** I would choose Kubernetes for microservices when the system has real scaling, resilience, and operational needs. For very small applications, a simpler solution can be better.

**Key points for full marks:**
- Kubernetes fits many independent services.
- Orchestration: Deployments, Services, scaling, rolling updates.
- Resilience: self-healing, probes, rescheduling.
- Ecosystem is strong, but complexity is the trade-off.

---

## LO6.21 — Choose Kubernetes for enterprise-grade applications?

**Question:** Would you choose Kubernetes for enterprise-grade applications, focus on its scalability, community support and feature richness. Elaborate your standing with arguments.

**Answer:**  
Yes, I would choose Kubernetes for many enterprise-grade applications, but only if the company has the skills or uses a managed Kubernetes service. Kubernetes is very scalable and can run many applications and services across many nodes. It supports rolling updates, self-healing, service discovery through Services and DNS, storage integration, and declarative configuration.

Kubernetes also has a very large community and ecosystem. Many cloud providers, vendors, and open-source tools support it. This gives companies flexibility and avoids depending on one small platform. Features like RBAC, NetworkPolicy, namespaces, resource quotas, probes, and autoscaling are useful in enterprise environments.

However, Kubernetes is not automatically enterprise-ready just because it is powerful. The company must design security, governance, monitoring, backup, cost management, and upgrade processes. Without that, Kubernetes can become complex and risky.

**Conclusion:** Kubernetes is a strong choice for enterprise-grade applications because of scalability, ecosystem, and feature richness. But it needs good operations and security practices.

**Key points for full marks:**
- Scalable and feature-rich orchestration platform.
- Large community and vendor support.
- Enterprise use needs governance, security, monitoring, and skilled operations.

---

## LO6.22 — Choose OpenShift for a microservices architecture?

**Question:** Would you choose OpenShift as a solution for a microservices architecture, focus on its orchestration, resilience and ecosystem. Elaborate your standing with arguments.

**Answer:**  
Yes, I would choose OpenShift for a microservices architecture in an enterprise environment. OpenShift uses Kubernetes for orchestration, so it supports Deployments, Services, scaling, self-healing, rolling updates, and many Kubernetes ecosystem tools. This makes it suitable for running many microservices.

OpenShift adds extra value for developers and operations teams. It provides a developer console, Routes, S2I/build workflows, integrated registry, monitoring, and stronger security defaults. These features can make microservices delivery more standardized and easier for teams.

The trade-off is cost, resource footprint, and platform complexity. OpenShift can also create more dependency on Red Hat-specific features. For a small team or simple application, plain Kubernetes, managed Kubernetes, or Compose may be cheaper and simpler.

**Conclusion:** I would choose OpenShift for microservices when the company needs enterprise support, security, and integrated tooling. For small or low-cost projects, it may be too heavy.

**Key points for full marks:**
- OpenShift includes Kubernetes orchestration.
- Good for enterprise microservices with built-in tooling.
- Trade-off: cost, resource use, complexity, Red Hat dependency.

---

## LO6.23 — Choose OpenShift for enterprise-grade applications?

**Question:** Would you choose OpenShift for enterprise-grade applications, focus on its scalability, community support and feature richness. Elaborate your standing with arguments.

**Answer:**  
Yes, OpenShift is a strong choice for enterprise-grade applications. It is based on Kubernetes, so it supports scalable container orchestration, self-healing, rolling updates, Services, storage integration, and many cloud-native patterns. On top of that, it adds Red Hat enterprise support, the OKD/OpenShift ecosystem, and integrated platform features.

OpenShift is useful for enterprises because it provides stronger default security, Security Context Constraints, RBAC integration, developer and operations tooling, integrated registry, built-in monitoring, logging options through platform integrations, and lifecycle management. This reduces the amount of separate tools that the company must integrate by itself.

The disadvantages are cost, higher resource requirements, and more dependency on Red Hat. Also, teams must learn both Kubernetes and OpenShift-specific concepts. Still, for regulated or large enterprise environments, support and standardization can be more important than having the lightest platform.

**Conclusion:** I would choose OpenShift for enterprise-grade applications when security, support, governance, and integrated tooling are important. For smaller or cost-sensitive environments, plain or managed Kubernetes may be enough.

**Key points for full marks:**
- OpenShift = Kubernetes + enterprise platform features.
- Strong for scalability, enterprise support, ecosystem, security, and standardization.
- Trade-off: cost, footprint, and Red Hat-specific knowledge.

---

# Possible extra LO6 exam questions and ready answers

These are not in the original list, but they are realistic variations based on the same topics.

---

## Extra 1 — Kubernetes Deployment vs StatefulSet

**Possible question:** Compare Deployment and StatefulSet. When would you use each?

**Answer:**  
A Deployment is used for stateless applications. Its Pods have random names and can be replaced at any time. This is good for web applications or APIs where any replica can handle the request.

A StatefulSet is used for stateful applications. It gives stable Pod names, stable identity, ordered startup and shutdown by default, and usually one PVC per replica. This is useful for databases or clustered systems that need stable identity and persistent data.

**Conclusion:** I would use Deployment for stateless services and StatefulSet for stateful services that need stable names and storage.

**Key points:**
- Deployment: stateless, random Pod names, easy scaling.
- StatefulSet: stable identity, ordered operations by default, persistent storage.

---

## Extra 2 — Kubernetes Service vs Ingress/Route

**Possible question:** Compare Service, Ingress, and OpenShift Route.

**Answer:**  
A Kubernetes Service gives a stable way to access Pods. Inside the cluster, a ClusterIP Service gives a stable virtual IP and DNS name. NodePort and LoadBalancer can expose the application outside the cluster.

Ingress is a Kubernetes object used for HTTP/HTTPS routing from outside the cluster to Services. It normally needs an Ingress controller. OpenShift Route is an OpenShift-specific object that also exposes HTTP/HTTPS applications externally, but it is integrated into OpenShift.

**Conclusion:** Service connects traffic to Pods. Ingress and Route are used to expose HTTP/HTTPS applications externally with hostnames and routing rules.

**Key points:**
- Service: stable internal access to Pods.
- Ingress: Kubernetes HTTP/HTTPS external routing.
- Route: OpenShift-specific external HTTP/HTTPS routing.

---

## Extra 3 — Why are probes important in Kubernetes?

**Possible question:** Explain liveness, readiness, and startup probes.

**Answer:**  
Probes help Kubernetes understand the real health of an application, not only whether the process is running. A liveness probe checks if the container is still alive. If it fails, Kubernetes restarts the container.

A readiness probe checks if the application is ready to receive traffic. If it fails, the Pod is removed from Service endpoints, but it is not restarted only because of readiness failure. A startup probe is useful for slow-starting applications. It gives the application more time to start before liveness checks can kill it.

**Conclusion:** Probes improve reliability because Kubernetes can restart broken containers and avoid sending traffic to Pods that are not ready.

**Key points:**
- Liveness: restart if unhealthy.
- Readiness: receive traffic only when ready.
- Startup: protect slow startup from premature restart.

---

## Extra 4 — ConfigMap vs Secret

**Possible question:** Compare ConfigMap and Secret.

**Answer:**  
A ConfigMap stores non-sensitive configuration, such as application settings, feature flags, or configuration files. It can be mounted as files or loaded as environment variables.

A Secret stores sensitive data, such as passwords, tokens, or certificates. In Kubernetes, Secret data in YAML is base64-encoded by default. Base64 is not encryption. Secrets can be protected better than ConfigMaps because Kubernetes treats them as sensitive objects, they can be mounted with restrictive permissions, access can be controlled with RBAC, and encryption at rest can be enabled for etcd.

**Conclusion:** I would use ConfigMap for normal configuration and Secret for sensitive values. I would also protect Secret access with RBAC and enable encryption at rest if required.

**Key points:**
- ConfigMap: non-sensitive config.
- Secret: sensitive data.
- Base64 is encoding, not encryption.
- Use RBAC and encryption at rest for real security.

---

## Extra 5 — Why use namespaces in Kubernetes?

**Possible question:** Explain the purpose of namespaces.

**Answer:**  
Namespaces separate resources inside the same Kubernetes cluster. They are useful for separating teams, environments, or applications, for example dev, test, and production.

Namespaces help with organization, RBAC permissions, resource quotas, and avoiding name conflicts. However, namespaces are not a full security boundary by themselves. For stronger isolation, the cluster also needs RBAC, NetworkPolicies, admission controls, and sometimes separate clusters.

**Conclusion:** Namespaces are useful for logical separation and access control, but they must be combined with other controls for strong security.

**Key points:**
- Logical separation of resources.
- Useful for teams/environments.
- Works with RBAC and quotas.
- Not a complete security boundary alone.

---

## Extra 6 — Helm vs plain YAML manifests

**Possible question:** Compare Helm charts with plain Kubernetes YAML manifests.

**Answer:**  
Plain YAML manifests are simple and direct. They are good for learning and for small applications. However, when an application has many Kubernetes objects, managing many YAML files can become repetitive.

Helm packages Kubernetes resources into charts. It supports templates, values files, versioning, installation, upgrade, and rollback. This is useful for complex applications and reusable deployments. The disadvantage is that Helm adds another tool and template layer that the team must understand.

**Conclusion:** I would use plain YAML for simple tasks and learning. I would use Helm for larger applications that need reusable and configurable deployments.

**Key points:**
- YAML: simple and transparent.
- Helm: templates, packaging, values, upgrades.
- Helm adds power but also complexity.

---

## Extra 7 — Kubernetes vs serverless containers

**Possible question:** Compare Kubernetes with serverless container platforms.

**Answer:**  
Kubernetes gives more control over networking, storage, scaling, deployment strategy, and cluster configuration. It is good when the team needs a flexible platform for many services and custom requirements.

Serverless container platforms reduce operational overhead. The cloud provider manages most infrastructure, and the team focuses on the container image and application configuration. This is good for small teams or workloads with variable traffic. The trade-off is less control and more dependency on the provider.

**Conclusion:** I would choose Kubernetes for control and complex platforms. I would choose serverless containers when speed, simplicity, and low operational overhead are more important.

**Key points:**
- Kubernetes: more control, more complexity.
- Serverless containers: less ops, less control.
- Choice depends on team size and workload complexity.

---

## Extra 8 — Why is observability important in Kubernetes?

**Possible question:** Why do Kubernetes systems need monitoring, logging, and tracing?

**Answer:**  
Kubernetes systems are dynamic. Pods can restart, move between nodes, scale up, and scale down. Because of that, it is not enough to only check one server manually. The team needs observability to understand what is happening.

Monitoring shows metrics such as CPU, memory, request rate, and errors. Logging shows application and system messages. Tracing helps follow one request across multiple microservices. Together, these tools help detect problems, troubleshoot incidents, and prove system health.

**Conclusion:** Observability is necessary in Kubernetes because applications are distributed and constantly changing.

**Key points:**
- Monitoring: metrics and alerts.
- Logging: application/system messages.
- Tracing: request path across services.
- Needed for troubleshooting and reliability.

---

## Extra 9 — Kubernetes security basics for enterprise use

**Possible question:** List important security controls for Kubernetes enterprise use.

**Answer:**  
Important Kubernetes security controls include RBAC, namespaces, NetworkPolicies, admission control, image scanning, Secrets management, resource limits, audit logging, and regular patching. Workloads should run with least privilege, not as root if possible, and without unnecessary capabilities.

The cluster should also protect the API server, restrict access to kubeconfig files, and use secure CI/CD pipelines. For sensitive data, Secrets should be protected with RBAC and encryption at rest.

**Conclusion:** Kubernetes security is not one feature. It is a combination of identity, permissions, network control, image security, runtime security, and monitoring.

**Key points:**
- RBAC, NetworkPolicy, admission control.
- Least privilege and non-root containers.
- Protect Secrets and API access.
- Audit, patching, monitoring.

---

## Extra 10 — When is Kubernetes not a good choice?

**Possible question:** Give situations where Kubernetes is not the best solution.

**Answer:**  
Kubernetes is not always the best choice. It can be too complex for a small application, a small team, or a simple single-server deployment. If the team does not need scaling, high availability, self-healing, or many services, Kubernetes may create more work than value.

Kubernetes is also not ideal if the team has no operational knowledge and no managed service option. In that case, a simpler solution like Docker Compose, PaaS, or serverless containers may be safer and faster.

**Conclusion:** Kubernetes is powerful, but it should solve a real problem. I would not choose it only because it is popular.

**Key points:**
- Not ideal for very small/simple apps.
- High operational complexity.
- Needs skilled team or managed service.
- Use simpler tools when they meet the requirements.

---

# Very short LO6 answer template

If the professor changes the question, use this structure:

```text
Option A is good when ...
Option B is good when ...
The main trade-off is ...
In this case, I would choose ... because ...
```

For full marks, always include:

1. **Both sides of the comparison**
2. **At least one advantage and one disadvantage**
3. **A technical reason**
4. **A clear final recommendation**
