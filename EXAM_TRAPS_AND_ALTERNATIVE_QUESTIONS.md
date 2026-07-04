# Exam traps and alternative questions — što te može sjebati ako profesor malo promijeni pitanje

Ovaj file čitaj kao dodatak glavnim Q/A dokumentima. Glavni dokumenti su za situaciju kada pitanje dođe isto ili skoro isto. Ovaj file je za situaciju kada profesor promijeni ime objekta, image, port, namespace ili traži “dijagnosticiraj” umjesto “napravi”.

## 0. Najvažnija pravila prije svakog zadatka

```bash
kubectl get nodes
kubectl get pods,rs,deploy,svc
kubectl get events --sort-by=.lastTimestamp | tail -20
```

Ako naredba faila jer objekt već postoji:

```bash
kubectl delete <kind> <name> --ignore-not-found
```

Ako ne znaš ime poda:

```bash
kubectl get pods
kubectl get pods --show-labels
```

Ako ne znaš ime containera u Deploymentu:

```bash
kubectl get deploy <deploy-name> -o jsonpath='{.spec.template.spec.containers[*].name}'; echo
```

Ako `kubectl set image` ne radi, problem je skoro uvijek krivo ime containera. Sigurna forma:

```bash
CONTAINER=$(kubectl get deploy web -o jsonpath='{.spec.template.spec.containers[0].name}')
kubectl set image deploy/web ${CONTAINER}=nginx:1.27
```

Ako YAML ne prolazi:

```bash
kubectl apply --dry-run=server --validate=true -f file.yaml
```

Ako Pod ne radi:

```bash
kubectl describe pod <pod>
kubectl logs <pod>
kubectl logs <pod> --previous
```

---

# LO4 alternative traps

## LO4.1 Deployment/export YAML
**Može promijeniti:** ime Deploymenta, image, broj replika.

Sigurni pattern:

```bash
kubectl create deployment <name> --image=<image:tag> --replicas=<number>
kubectl get deploy <name> -o yaml > <name>.yaml
kubectl get pods -l app=<name>
```

Ako `--replicas` ne postoji u toj verziji kubectla:

```bash
kubectl create deployment <name> --image=<image:tag>
kubectl scale deploy <name> --replicas=<number>
```

## LO4.2 DockerHub authenticated pulls
**Može promijeniti:** DockerHub u Quay/private registry.

DockerHub:

```bash
kubectl create secret docker-registry dockerhub --docker-server=docker.io --docker-username=USER --docker-password=PASS --docker-email=email@example.com
kubectl patch sa default -p '{"imagePullSecrets":[{"name":"dockerhub"}]}'
```

Quay/private registry:

```bash
kubectl create secret docker-registry regcred --docker-server=quay.io --docker-username=USER --docker-password=PASS
```

Ako traži na Pod/Deployment, ne na ServiceAccount:

```yaml
spec:
  imagePullSecrets:
  - name: regcred
```

## LO4.3 Scaling
**Može promijeniti:** 3→5 u 2→4 ili traži manifest-only.

Imperative:

```bash
kubectl scale deploy <name> --replicas=<number>
```

Manifest:

```bash
kubectl get deploy <name> -o yaml > <name>.yaml
# promijeni spec.replicas
kubectl apply -f <name>.yaml
```

Ne očekuj novi ReplicaSet ako se mijenja samo broj replika.

## LO4.4 Rolling update
**Može promijeniti:** drugi image/tag.

```bash
CONTAINER=$(kubectl get deploy <name> -o jsonpath='{.spec.template.spec.containers[0].name}')
kubectl set image deploy/<name> ${CONTAINER}=<new-image:tag>
kubectl rollout status deploy/<name>
```

Ako rollout zapne:

```bash
kubectl describe deploy <name>
kubectl get pods
kubectl rollout undo deploy/<name>
```

## LO4.5 Rollback/change-cause
**Može tražiti:** rollback na specifičnu reviziju.

```bash
kubectl rollout history deploy/<name>
kubectl rollout history deploy/<name> --revision=<number>
kubectl rollout undo deploy/<name> --to-revision=<number>
```

Change-cause puniš annotationom:

```bash
kubectl annotate deploy/<name> kubernetes.io/change-cause="some change" --overwrite
```

## LO4.6/LO4.7 Strategy
Ako prebacuješ iz RollingUpdate u Recreate, moraš maknuti `rollingUpdate` blok:

```bash
kubectl patch deploy <name> --type=merge -p '{"spec":{"strategy":{"type":"Recreate","rollingUpdate":null}}}'
```

Ako vraćaš na RollingUpdate:

```bash
kubectl patch deploy <name> --type=merge -p '{"spec":{"strategy":{"type":"RollingUpdate","rollingUpdate":{"maxSurge":1,"maxUnavailable":0}}}}'
```

## LO4.8 requests/limits
**Može promijeniti:** vrijednosti CPU/memory.

```bash
kubectl set resources deploy <name> --requests=cpu=<x>,memory=<y> --limits=cpu=<x>,memory=<y>
kubectl describe pod -l app=<name> | grep -A5 -i 'Limits\|Requests'
```

Ako Pod ostane Pending nakon requests, resursi su možda veći od node capacity.

## LO4.9 revisionHistoryLimit
Ako staviš `0`, rollback praktički gubiš. Za ispit koristi broj iz pitanja:

```bash
kubectl patch deploy <name> -p '{"spec":{"revisionHistoryLimit":3}}'
```

## LO4.10 bad image tag
Ovo je očekivano da zapne. Ne paničari:

```bash
kubectl get pods
kubectl describe pod <bad-pod>
kubectl rollout undo deploy/<name>
```

Stari Podovi ostaju zbog RollingUpdate zaštite.

## LO4.11 labels/selectors
Ako label nije `app=web`, ne koristi napamet `-l app=web`:

```bash
kubectl get deploy <name> -o jsonpath='{.spec.selector.matchLabels}'; echo
kubectl get pods --show-labels
```

## LO4.12 expose Deployment
Ako Service već postoji:

```bash
kubectl delete svc <svc-name> --ignore-not-found
kubectl expose deploy <deploy-name> --name=<svc-name> --port=<service-port> --target-port=<container-port>
```

Default Service type je ClusterIP.

## LO4.13 sidecar
Ako dva containera koriste isti port u istom Podu, drugi neće moći bindati port. U istom Podu containers dijele network namespace.

Provjera shared volumea:

```bash
kubectl exec <pod> -c <container1> -- ls <path>
kubectl exec <pod> -c <container2> -- cat <same-file>
```

## LO4.14 nodeSelector
Ako Pod ostaje Pending, provjeri node labels:

```bash
kubectl get nodes --show-labels
kubectl describe pod <pod>
```

Fix:

```bash
kubectl label node <node-name> disktype=ssd
```

Ako node nije `minikube`, ne koristi napamet `minikube`; prvo nađi ime nodea.

## LO4.15 bare Pod vs managed Pod
Bare Pod se ne vraća nakon delete. Deployment-managed Pod se vraća.

```bash
kubectl delete pod <pod>
kubectl get pods -w
```

## LO4.16/21/27 StatefulSet
StatefulSet ima stabilna imena `<name>-0`, `<name>-1`. Ako traži DNS, treba headless Service i `serviceName` u StatefulSetu.

Ako traži PVC per replica, treba `volumeClaimTemplates`; obični volume nije dovoljan.

## LO4.17 DaemonSet
Na single-node minikubeu očekuješ 1 Pod. Na 3 nodea očekuješ 3 Poda.

```bash
kubectl get nodes
kubectl get ds
kubectl get pods -l app=agent -o wide
```

## LO4.18/19/20 Job/CronJob
Ako `kubectl logs job/<job>` ne radi odmah, Job možda još nije gotov:

```bash
kubectl wait --for=condition=complete job/<name> --timeout=120s
kubectl logs job/<name>
```

CronJob stvara Jobs tek po rasporedu. Za test:

```bash
kubectl create job manual-test --from=cronjob/<cronjob-name>
```

## LO4.22 emptyDir
`emptyDir` traje dok traje Pod. Ako se Pod izbriše, podaci nestaju. Ne koristi za trajnu bazu.

## LO4.23 StorageClasses/PV/PVC
Ako nema PV/PVC, to nije nužno greška — možda još nisi kreirao claim.

```bash
kubectl get sc
kubectl get pv
kubectl get pvc -A
```

## LO4.24/25 ConfigMap volume/subPath
Ako ConfigMap već postoji, koristi apply pattern:

```bash
kubectl create configmap appcfg --from-literal=color=blue --from-literal=mode=dark --dry-run=client -o yaml | kubectl apply -f -
```

`subPath` mount ne dobiva live updates. Restartaj Pod za promjenu.

## LO4.26–37 Secrets
Secret je base64 encoded, ne encrypted.

Ako Secret već postoji:

```bash
kubectl create secret generic dbcreds --from-literal=username=admin --from-literal=password=s3cret --dry-run=client -o yaml | kubectl apply -f -
```

Ako Pod ne može mountati Secret, provjeri namespace:

```bash
kubectl get secret -n <namespace>
kubectl get pod <pod> -n <namespace>
```

Secrets su namespaced — Pod i Secret moraju biti u istom namespaceu.

## LO4.38–49 Services
Ako Service ne radi, idi ovim redom:

```bash
kubectl get svc <svc>
kubectl get endpoints <svc>
kubectl get pods --show-labels
kubectl describe svc <svc>
kubectl run tmp --rm -it --image=busybox -- nslookup <svc>
kubectl run tmp --rm -it --image=busybox -- wget -qO- <svc>:<port>
```

Ako endpoints empty: selector ne matcha Pod labels ili Pod nije Ready.

NodePort radi na `<nodeIP>:<nodePort>`. Na minikube:

```bash
minikube ip
minikube service <svc> --url
```

LoadBalancer na minikube često ostaje `<pending>` dok ne pokreneš:

```bash
minikube tunnel
```

Ako nema minikubea, ne koristi minikube naredbe — objasni da LoadBalancer treba cloud provider ili MetalLB.

## LO4.50–53 probes
Readiness ne restarta container; samo ga makne iz Service endpointa. Liveness restarta container. Startup štiti slow-starting aplikacije.

Ako professor promijeni port/path, promijeni ih u probeu:

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
```

Startup budget:

```text
failureThreshold * periodSeconds >= expected boot time
```

Za 2 minute možeš koristiti `30 * 5 = 150s`.

---

# LO5 alternative traps

## Podman general
Ako container već postoji:

```bash
podman rm -f <name>
```

Ako container exited:

```bash
podman ps -a
podman logs <name>
podman inspect <name>
```

Ako port mapping ne radi:

```bash
podman port <name>
podman inspect <name> | grep -i port -A5
```

Ako bind mount ne radi na SELinux:

```bash
-v /absolute/path:/container/path:Z
```

Ako DNS između containera ne radi, koristi user-defined network:

```bash
podman network create mynet
podman run --network mynet --name a ...
podman run --network mynet --name b ...
```

## Containerfile/Dockerfile general
Ako apt paket nije pronađen:

```Dockerfile
RUN apt-get update && apt-get install -y package
```

Ako image postane ogroman, očisti cache u istom RUN layeru:

```Dockerfile
RUN apt-get update && apt-get install -y package && rm -rf /var/lib/apt/lists/*
```

Ako CMD ne prima signale dobro, koristi exec form:

```Dockerfile
CMD ["python", "app.py"]
```

Ako Node build stalno radi npm install:

```Dockerfile
COPY package*.json ./
RUN npm install
COPY . .
```

Ako PATH pokvari shell/curl:

```Dockerfile
ENV PATH="/app/bin:${PATH}"
```

## Kubernetes troubleshooting general

### Pending
```bash
kubectl describe pod <pod>
```
Traži: `Insufficient cpu`, `Insufficient memory`, `nodeSelector`, `unbound PVC`, taints.

### ImagePullBackOff
```bash
kubectl describe pod <pod>
```
Traži: image typo, tag ne postoji, private registry bez Secret.

### CrashLoopBackOff
```bash
kubectl logs <pod> --previous
```
App se ruši nakon starta.

### OOMKilled
```bash
kubectl get pod <pod> -o yaml | grep -i -A5 lastState
kubectl describe pod <pod>
```
Fix: povećaj memory limit ili popravi app.

### Service ne radi
```bash
kubectl get endpoints <svc>
kubectl get svc <svc> -o yaml
kubectl get pods --show-labels
```
Ako endpoints empty, Service selector ne matcha Pod labels ili Pod nije Ready.

### YAML typo/indentation
```bash
kubectl apply --dry-run=server --validate=true -f file.yaml
```

### Secret/ConfigMap key missing
```bash
kubectl describe pod <pod>
kubectl get configmap <name> -o yaml
kubectl get secret <name> -o yaml
```

---

# LO6 alternative traps

LO6 odgovor uvijek piši u ovom formatu:

```text
Option A is better for ...
Option B is better for ...
The main trade-off is ...
I would choose ... because ...
```

Ako pitanje kaže **analyze**: usporedi model i objasni posljedice.

Ako kaže **evaluate**: navedi prednosti, mane i trade-off.

Ako kaže **defend recommendation**: odaberi jednu opciju i obrani je.

Ako kaže **critically evaluate**: nemoj pisati samo pozitivno; moraš navesti i rizike/mane.

## Brzi izbor rješenja

- Mali tim, jedan server, mora brzo i jeftino: **Docker Compose / Podman Compose**.
- Mali on-prem cluster, malo resursa: **k3s**.
- Enterprise, regulacija, support, security defaults: **OpenShift**.
- Cloud-native microservices, skaliranje, ecosystem: **Kubernetes / managed Kubernetes**.
- Tim nema ops znanje: izbjegni self-managed Kubernetes; bolje managed service ili OpenShift.
- Spiky global traffic: managed Kubernetes + autoscaling + cloud load balancer/CDN.
- Security-conscious single-host/container tooling: **Podman rootless/daemonless**.

## Rečenice koje pašu skoro svugdje

```text
The best choice depends on team size, operational maturity, scalability requirements, security requirements and budget.
```

```text
Kubernetes provides more scalability and resilience, but it also adds operational complexity.
```

```text
OpenShift reduces integration work and improves enterprise support, but it adds licensing cost and more Red Hat-specific abstractions.
```

```text
Docker Compose is simpler and cheaper, but it lacks cluster-level self-healing, scheduling and advanced orchestration.
```
