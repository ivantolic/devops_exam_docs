# LO6 — Pravila za pisanje teorijskih odgovora na ispitu

Ovaj dokument služi kao kratki vodič kako koristiti `LO6_FULL_MARKS_SIMPLE_ENGLISH_REVIEWED.md` na ispitu.
LO6 je teorijski ishod. Tu se najčešće ne pišu komande, nego se uspoređuju tehnologije i objašnjava zašto bi odabrao jedno rješenje umjesto drugog.

---

## 1. Glavno pravilo za LO6

Za svaki LO6 odgovor pokušaj imati ovu strukturu:

1. **Define / explain topic** — prvo kratko objasni o čemu se radi.
2. **Compare both sides** — usporedi dvije opcije koje pitanje spominje.
3. **Advantages** — napiši glavne prednosti.
4. **Disadvantages / trade-offs** — napiši mane ili ograničenja.
5. **Final recommendation** — završi jasnim zaključkom.

Primjer strukture:

```text
Kubernetes is better when the company needs scaling, self-healing and many microservices.
Docker Compose is simpler and cheaper for one host and a small application.
The trade-off is that Kubernetes gives more power, but also needs more knowledge and maintenance.
For a small team I would start with Compose, but I would move to Kubernetes when scaling and resilience become important.
```

---

## 2. Nemoj pisati samo definicije

LO6 pitanja često imaju riječi kao:

- compare
- evaluate
- critically evaluate
- defend
- recommend
- would you choose
- analyze

To znači da profesor ne želi samo definiciju. Moraš pokazati **zašto** je nešto dobro ili loše u određenom slučaju.

Slab odgovor:

```text
Kubernetes is an orchestration system. Docker Compose runs containers.
```

Bolji odgovor:

```text
Kubernetes is an orchestration system for running containers across a cluster. It supports scaling, self-healing, Services and rolling updates. Docker Compose is simpler and works well on one host, but it does not provide the same cluster-level resilience. Because of that, Compose is better for small/simple systems, while Kubernetes is better for production systems that need high availability and scaling.
```

---

## 3. Za svaki odgovor moraš imati trade-off

Skoro svako LO6 pitanje traži razmišljanje o prednostima i manama.
Nemoj napisati samo da je jedna tehnologija “bolja”. Napiši **u kojem slučaju** je bolja.

Primjeri dobrih trade-off rečenica:

```text
The main trade-off is complexity. Kubernetes gives more automation and resilience, but it needs more operational knowledge.
```

```text
OpenShift gives more built-in enterprise features, but it also adds licensing cost and stronger dependency on Red Hat.
```

```text
Managed Kubernetes reduces operational overhead, but the company still needs to understand workloads, security and networking.
```

---

## 4. Koristi jednostavan engleski, ali tehnički točan

Ne moraš zvučati kao native speaker. Bitnije je da odgovor bude jasan i tehnički točan.

Dobro je pisati ovako:

```text
I would choose Kubernetes if the application has many services and needs scaling, self-healing and rolling updates.
```

Nemoj pisati previše komplicirano:

```text
Kubernetes constitutes a highly sophisticated orchestration abstraction layer for enterprise cloud-native ecosystems...
```

To zvuči umjetno i može sakriti da nisi stvarno objasnio odgovor.

---

## 5. Ako je pitanje “Would you choose...”, moraš dati jasan stav

Kod pitanja tipa:

- Would you choose Kubernetes for microservices?
- Would you choose OpenShift for enterprise applications?
- Should a small team use Docker Compose or Kubernetes?

Nemoj odgovoriti neutralno bez zaključka. Moraš završiti s preporukom.

Dobar završetak:

```text
My recommendation is to start with Docker Compose for this small team, because it is cheaper and easier to operate. I would move to Kubernetes later if the product needs scaling, high availability and more automated deployment.
```

---

## 6. Ako pitanje spominje enterprise/reguliranu industriju

Ako se spominje enterprise, banke, regulirana industrija ili security, obavezno spomeni:

- RBAC
- security policies / SCC kod OpenShifta
- image registry / image security
- monitoring / audit / compliance
- vendor support
- predictable platform

Primjer:

```text
For regulated industries, OpenShift is often attractive because it gives stronger default security, Red Hat support, integrated platform tools and better control over users, projects and permissions. This makes compliance and operations easier than building everything manually on vanilla Kubernetes.
```

Pazi: nemoj tvrditi da je OpenShift “automatically secure”. To nije točno. Bolje napiši:

```text
OpenShift has stronger security defaults, but the team still has to configure workloads, access control and updates correctly.
```

---

## 7. Ako pitanje spominje small team/startup

Ako se spominje mali tim, startup, brzo i jeftino, najčešće je odgovor:

- Docker Compose ili managed/simple platform za početak
- Kubernetes tek kasnije ako treba scaling/resilience
- izbjegavati overengineering

Primjer:

```text
For a startup with two engineers, I would not start with full Kubernetes unless there is a strong reason. Docker Compose or a simple managed container service is cheaper and easier. Kubernetes can be introduced later when the application grows and needs scaling, self-healing and better deployment control.
```

---

## 8. Ako pitanje spominje spiky/global traffic

Ako se spominje nepredvidiv promet, globalni promet, streaming ili veliki promet, spomeni:

- managed Kubernetes ili cloud container platform
- autoscaling
- load balancing
- multi-region deployment
- CDN za statički/video sadržaj
- monitoring

Primjer:

```text
For spiky global traffic I would choose a managed Kubernetes solution or another managed container platform. It can autoscale workloads and integrate with cloud load balancers and monitoring. For media streaming, CDN is also important because it reduces load on the application and improves user experience globally.
```

---

## 9. Ako pitanje uspoređuje Kubernetes i OpenShift

Osnovna ideja:

```text
Kubernetes is the open-source orchestration base. OpenShift is an enterprise platform built on Kubernetes with extra developer, security and operations features.
```

Kubernetes prednosti:

- open source
- CNCF ecosystem
- cloud-neutral
- velika zajednica
- fleksibilan

OpenShift prednosti:

- Red Hat support
- developer console
- S2I
- stricter security defaults
- integrated platform features
- enterprise-friendly

OpenShift mane:

- veća kompleksnost
- licensing cost
- više Red Hat specifičnih stvari

Zaključak često bude:

```text
I would choose vanilla Kubernetes when I need flexibility and lower platform lock-in. I would choose OpenShift when the company wants an integrated, supported enterprise platform.
```

---

## 10. Ako pitanje uspoređuje Docker i Podman

Obavezno spomeni:

- Docker koristi daemon
- Podman je daemonless
- Podman bolje podržava rootless način rada
- Podman je dobar za security-conscious timove
- Docker ima širi ecosystem i poznatiji developer workflow

Primjer:

```text
Docker normally uses a central daemon, while Podman is daemonless. This means Podman does not require a long-running privileged daemon to manage containers. Podman also supports rootless containers well, so a user can run containers without root privileges. Because of that, a security-conscious team may prefer Podman. Docker can still be easier for teams because it is very popular and has a large ecosystem.
```

---

## 11. Ako pitanje spominje storage

Za Kubernetes storage uvijek spomeni:

- PVC
- PV
- StorageClass
- CSI
- dynamic provisioning
- stateful workloads

Dobra osnovna rečenica:

```text
Kubernetes storage is more flexible for stateful workloads because it uses PVCs, PVs, StorageClasses and CSI drivers. This allows applications to request storage without knowing the exact backend. Podman storage is useful on one host, but it is not a full cluster-level storage abstraction.
```

Pazi: nemoj tvrditi da Kubernetes sam rješava sve storage probleme. Bolje napiši:

```text
Kubernetes gives the abstraction, but the real behavior still depends on the storage backend and access mode.
```

---

## 12. Ako pitanje spominje networking

Za Kubernetes networking spomeni:

- svaki Pod ima IP
- Podovi mogu komunicirati preko cluster networka
- Service daje stable endpoint
- DNS daje service discovery
- CNI plugin implementira network

Za Docker/Podman networking spomeni:

- više host-based model
- bridge network
- port mapping
- dobar za single host

Primjer:

```text
Kubernetes has a cluster networking model where Pods get their own IP addresses and Services provide stable access to changing Pods. DNS is used for service discovery. Docker networking is usually more host-based, with bridge networks and port mapping. Docker is simpler for one host, while Kubernetes is better for multi-node applications.
```

---

## 13. Ako pitanje spominje CI/CD agents in Kubernetes vs VMs

Spomeni:

Kubernetes agents:

- dynamic agents
- autoscaling
- clean environment per job
- resource limits
- possible noisy-neighbor problem

VM agents:

- stronger isolation
- easier for special tools
- more stable for heavy builds
- more maintenance

Zaključak:

```text
I would use Kubernetes agents for scalable and temporary build jobs, but dedicated VMs can be better for heavy, sensitive or special builds that need stronger isolation.
```

---

## 14. Ako pitanje spominje managed vs self-managed Kubernetes

Managed Kubernetes:

- cloud provider manages control plane
- easier upgrades
- easier node scaling
- less operational overhead

Self-managed:

- more control
- can run on-premise
- more responsibility
- upgrades/backups/security are on the team

Dobar odgovor:

```text
Managed Kubernetes is usually better when the team wants to reduce day-2 operations. The provider manages much of the control plane and integrates with cloud services. Self-managed Kubernetes gives more control, but the team must handle upgrades, backups, monitoring, security and node failures.
```

---

## 15. Ako profesor promijeni pitanje

Ako pitanje nije identično, nemoj paničariti. Nađi o čemu se stvarno pita:

- ako pita “small team” → naglasi simplicity/cost/overhead
- ako pita “enterprise” → naglasi support/security/compliance/tooling
- ako pita “microservices” → naglasi orchestration/scaling/service discovery/resilience
- ako pita “storage” → naglasi PVC/PV/StorageClass/CSI
- ako pita “networking” → naglasi Pod IP/Service/DNS/CNI
- ako pita “security” → naglasi RBAC, policies, rootless, SCC, least privilege
- ako pita “cloud” → naglasi autoscaling/load balancers/dynamic provisioning/managed services

---

## 16. Koliko dug odgovor treba biti?

Za većinu LO6 pitanja ciljaj na **6 do 10 rečenica**.

Dovoljno je:

- 1 rečenica uvoda
- 2 do 3 rečenice usporedbe
- 2 do 3 rečenice prednosti/mana
- 1 rečenica zaključka

Nemoj pisati 20 rečenica ako se ponavljaš. Bolje kraće, ali jasno.

---

## 17. Što izbjegavati

Izbjegavaj apsolutne tvrdnje kao:

```text
Kubernetes is always better.
OpenShift is always secure.
Compose is only for testing.
Managed Kubernetes means no operations.
Secrets are encrypted by default.
```

Bolje formulacije:

```text
Kubernetes is better when scaling and resilience are important.
OpenShift has stronger security defaults, but it still requires correct configuration.
Compose is good for small/simple deployments, but it is limited for cluster-level resilience.
Managed Kubernetes reduces operational work, but the team still manages applications and security.
Secrets are base64-encoded by default, and stronger protection needs RBAC and encryption at rest.
```

---

## 18. Najsigurniji završeci odgovora

Možeš često završiti s jednom od ovih rečenica:

```text
So, my recommendation depends on the size of the team, the required scalability, and the operational knowledge available.
```

```text
Because of that, I would choose the simpler solution first, and move to Kubernetes when the system really needs orchestration and resilience.
```

```text
For enterprise use, I would prefer the solution that gives better support, security controls and operational tooling, even if it adds more cost and complexity.
```

```text
The best choice depends on whether the priority is simplicity, flexibility, security, or scalability.
```

---

## 19. Mini template za bilo koje LO6 pitanje

Ako se blokiraš, koristi ovaj template:

```text
[Technology A] is better for ______ because it provides ______.
[Technology B] is better for ______ because it is simpler / cheaper / more controlled.
The main advantage of [A] is ______.
The main disadvantage is ______.
The main trade-off is between ______ and ______.
In this situation, I would choose ______ because ______.
```

Primjer:

```text
Kubernetes is better for large microservice systems because it provides scaling, self-healing and service discovery. Docker Compose is better for small applications because it is simpler and cheaper to operate. The main advantage of Kubernetes is automation across many nodes. The main disadvantage is higher complexity. The main trade-off is between operational simplicity and production resilience. In this situation, I would choose Kubernetes if the application needs high availability and scaling.
```

---

## 20. Zadnje pravilo

LO6 nije test napamet naučenih definicija. LO6 je test možeš li objasniti **zašto** bi izabrao Kubernetes, OpenShift, Podman, Docker Compose, k3s ili managed Kubernetes u nekom scenariju.

Ako u odgovoru imaš:

- kratku definiciju,
- usporedbu,
- prednosti,
- mane,
- trade-off,
- zaključak,

onda je odgovor najčešće dovoljno dobar za pune bodove.
