# PAST EXAM EXPANDED ADDENDUM — dodatna pitanja, odgovori, objašnjenja i slične situacije

Ovaj dokument je **dodatak** tvojim finalnim DevOps materijalima.

Glavni dokumenti i dalje ostaju:

```text
LO4_QA_FINAL_EXPANDED.md
LO5_QA_FINAL_EXPANDED.md
LO6_FULL_MARKS_SIMPLE_ENGLISH_REVIEWED.md
EXAM_TRAPS_AND_ALTERNATIVE_QUESTIONS.md
SAFE_COPY_PASTE_RULES.md
yaml/
```

Ovaj dokument služi za:
- pitanja s prošlog ispita,
- objašnjenje što je tu dodatno u odnosu na tvoje glavne dokumente,
- “što ako profesor promijeni ime/image/port/ConfigMap/selector” situacije,
- typed answers na engleskom ako moraš pisati objašnjenje.

---

# 0. Najvažnije pravilo

Nemoj slijepo kopirati `web` ako pitanje kaže drugo ime.

Ako pitanje kaže:

```text
static-site-deployment
```

onda komanda mora koristiti:

```bash
static-site-deployment
```

ne:

```bash
web
```

Isto vrijedi za:
- image,
- port,
- ConfigMap name,
- Secret name,
- namespace,
- label,
- path u containeru.

---

# LO4 — pitanja s prošlog ispita

## LO4.1 — Deployment `static-site-deployment` s 2 replike i `nginx:alpine`

### Pitanje

Deploy a deployment named `static-site-deployment` with two replicas using the `nginx:alpine` container image.

### Što je dodatno u odnosu na tvoje glavne dokumente?

U glavnom LO4 dokumentu imaš isti tip zadatka, ali tamo se često koristi Deployment `web`.

Ovdje profesor traži drugo ime:

```text
static-site-deployment
```

Zato ne smiješ zalijepiti:

```bash
kubectl create deployment web --image=nginx:alpine --replicas=2
```

To bi tehnički napravilo Deployment, ali **ne s traženim imenom**, pa možeš izgubiti bodove.

### Ispravno rješenje

```bash
kubectl delete deployment static-site-deployment --ignore-not-found
kubectl create deployment static-site-deployment --image=nginx:alpine --replicas=2
```

### Što radi koja komanda?

```bash
kubectl delete deployment static-site-deployment --ignore-not-found
```

Briše stari Deployment ako već postoji. `--ignore-not-found` znači da neće baciti problem ako objekt ne postoji.

```bash
kubectl create deployment static-site-deployment --image=nginx:alpine --replicas=2
```

Kreira Deployment s točnim imenom, imageom i brojem replika.

### Provjera

```bash
kubectl get deployment static-site-deployment
kubectl get pods -l app=static-site-deployment
kubectl describe deployment static-site-deployment | grep -i image
```

### Što gledam u provjeri?

- `kubectl get deployment` mora pokazati `READY 2/2`.
- `kubectl get pods -l app=static-site-deployment` mora pokazati dva Poda.
- `describe deployment` mora pokazati image `nginx:alpine`.

### Typed answer in English

I created a Deployment named `static-site-deployment` with two replicas using the `nginx:alpine` image. I verified it by listing the Deployment and its Pods, and by checking the image in the Deployment description.

---

## LO4.2 — ConfigMap `site-content`, key `index.html`, mount preko `subPath`

### Pitanje

Create a ConfigMap named `site-content` holding a key `index.html` whose value is a small HTML page containing your name, surname and exam group. Mount that key into the deployment at `/usr/share/nginx/html/index.html` using `subPath`. Verify the file inside a pod with `kubectl exec`.

### Što je dodatno u odnosu na tvoje glavne dokumente?

U glavnom LO4 dokumentu imaš:
- ConfigMap kao volume,
- ConfigMap `subPath`,
- nginx Deployment,
- `kubectl exec` provjeru.

Ali ovdje je zadatak **kombinacija više stvari**:
1. napravi ConfigMap s HTML sadržajem,
2. mountaj samo jedan key (`index.html`),
3. mount ide u postojeći Deployment,
4. path mora biti točno `/usr/share/nginx/html/index.html`,
5. moraš dokazati s `kubectl exec`.

Zato ovo nije samo “apply yaml/cm-subpath.yaml”. Ovdje moraš patchati postojeći Deployment.

---

### Korak 1 — napravi HTML file

Promijeni ime, prezime i grupu prema sebi.

```bash
cat > index.html <<'EOF'
<html>
  <body>
    <h1>Ivan Tolic - Exam Group A</h1>
  </body>
</html>
EOF
```

### Što ovo radi?

Ovo lokalno kreira datoteku `index.html`. Ta datoteka kasnije postaje key u ConfigMapu.

---

### Korak 2 — napravi ConfigMap `site-content`

```bash
kubectl create configmap site-content \
  --from-file=index.html=index.html \
  --dry-run=client -o yaml | kubectl apply -f -
```

### Što radi koja komanda?

```bash
kubectl create configmap site-content
```

Kreira ConfigMap imena `site-content`.

```bash
--from-file=index.html=index.html
```

Lijevo `index.html` je ime keya u ConfigMapu. Desno `index.html` je lokalna datoteka.

Rezultat je ConfigMap koji ima key:

```text
index.html
```

i value koji je sadržaj HTML stranice.

```bash
--dry-run=client -o yaml | kubectl apply -f -
```

Ovo je safe pattern. Ako ConfigMap već postoji, `apply` ga može ažurirati umjesto da command faila.

### Provjera ConfigMapa

```bash
kubectl get configmap site-content -o yaml
```

Trebaš vidjeti nešto poput:

```yaml
data:
  index.html: |
    <html>
      <body>
        <h1>Ivan Tolic - Exam Group A</h1>
      </body>
    </html>
```

---

### Korak 3 — mountaj ConfigMap key u Deployment preko `subPath`

```bash
kubectl patch deployment static-site-deployment --type=json -p='[
  {
    "op":"add",
    "path":"/spec/template/spec/volumes",
    "value":[
      {
        "name":"site-content",
        "configMap":{
          "name":"site-content",
          "items":[
            {"key":"index.html","path":"index.html"}
          ]
        }
      }
    ]
  },
  {
    "op":"add",
    "path":"/spec/template/spec/containers/0/volumeMounts",
    "value":[
      {
        "name":"site-content",
        "mountPath":"/usr/share/nginx/html/index.html",
        "subPath":"index.html"
      }
    ]
  }
]'
```

### Što ovo radi?

Ovo patcha Deployment i dodaje dvije stvari u Pod template:

#### 1. Volume

```json
{
  "name":"site-content",
  "configMap":{
    "name":"site-content",
    "items":[
      {"key":"index.html","path":"index.html"}
    ]
  }
}
```

To znači: uzmi ConfigMap `site-content` i iz njega koristi key `index.html`.

#### 2. VolumeMount

```json
{
  "name":"site-content",
  "mountPath":"/usr/share/nginx/html/index.html",
  "subPath":"index.html"
}
```

To znači: mountaj samo file `index.html` na nginx default index path.

---

### Zašto treba `subPath`?

Bez `subPath`, Kubernetes bi mountao ConfigMap kao cijeli folder i mogao bi prekriti cijeli `/usr/share/nginx/html` direktorij.

Sa `subPath`, mounta se samo jedan file:

```text
/usr/share/nginx/html/index.html
```

To je preciznije i baš odgovara pitanju.

---

### Važna napomena za ispit

Ako Deployment već ima `volumes` ili `volumeMounts`, ovaj JSON patch s `"op":"add"` može failati ako path već postoji ili ako treba append umjesto add.

U tom slučaju koristi YAML export/edit/apply pristup:

```bash
kubectl get deployment static-site-deployment -o yaml > static-site-deployment.yaml
```

U YAML dodaš u:

```yaml
spec:
  template:
    spec:
      volumes:
      - name: site-content
        configMap:
          name: site-content
          items:
          - key: index.html
            path: index.html
```

i u container:

```yaml
volumeMounts:
- name: site-content
  mountPath: /usr/share/nginx/html/index.html
  subPath: index.html
```

Zatim:

```bash
kubectl apply -f static-site-deployment.yaml
```

---

### Čekaj rollout

```bash
kubectl rollout status deployment/static-site-deployment
```

Patch mijenja Pod template, zato Kubernetes mora napraviti novi rollout.

---

### Provjera preko `kubectl exec`

```bash
POD=$(kubectl get pod -l app=static-site-deployment -o jsonpath='{.items[0].metadata.name}')
kubectl exec "$POD" -- cat /usr/share/nginx/html/index.html
```

### Što ovo radi?

```bash
POD=$(kubectl get pod -l app=static-site-deployment -o jsonpath='{.items[0].metadata.name}')
```

Pronađe ime jednog Poda iz Deploymenta i spremi ga u varijablu `POD`.

```bash
kubectl exec "$POD" -- cat /usr/share/nginx/html/index.html
```

Uđe u Pod i ispiše HTML file koji je mountan iz ConfigMapa.

### Dodatna provjera preko HTTP-a

Ako imaš Service ili port-forward:

```bash
kubectl port-forward deployment/static-site-deployment 8080:80
```

U drugom terminalu:

```bash
curl localhost:8080
```

Trebao bi vidjeti svoju HTML stranicu.

### Typed answer in English

I created a ConfigMap named `site-content` with the key `index.html`. Then I mounted only that key into the nginx container at `/usr/share/nginx/html/index.html` using `subPath`. I verified the result with `kubectl exec` by reading the file inside a running Pod.

---

## LO4.3 — Rolling update na `nginx:stable-alpine`, history, rollback i dokaz da je image opet `nginx:alpine`

### Pitanje

Update the deployment's image to `nginx:stable-alpine`. Follow the rolling update with `kubectl rollout status`. Show the rollout history with `kubectl rollout history`, then roll back to the previous revision. Prove with `kubectl describe deployment` or `-o yaml` that the running image afterwards is again `nginx:alpine`.

### Što je dodatno u odnosu na tvoje glavne dokumente?

Ovo već imaš u LO4 expanded dokumentu, ali ovdje je bitno:
- deployment se ne zove `web`,
- početni image je `nginx:alpine`,
- update image je `nginx:stable-alpine`,
- nakon rollbacka moraš dokazati da je image opet `nginx:alpine`.

---

### Korak 1 — update image sigurno

```bash
CONTAINER=$(kubectl get deploy static-site-deployment -o jsonpath='{.spec.template.spec.containers[0].name}')
kubectl set image deployment/static-site-deployment ${CONTAINER}=nginx:stable-alpine
kubectl rollout status deployment/static-site-deployment
```

### Što radi koja komanda?

```bash
CONTAINER=$(kubectl get deploy static-site-deployment -o jsonpath='{.spec.template.spec.containers[0].name}')
```

Čita stvarno ime containera. Ovo je bolje nego pogađati ime containera.

```bash
kubectl set image deployment/static-site-deployment ${CONTAINER}=nginx:stable-alpine
```

Mijenja image i pokreće rolling update.

```bash
kubectl rollout status deployment/static-site-deployment
```

Prati rollout dok ne završi ili ne zapne.

---

### Korak 2 — rollout history

```bash
kubectl rollout history deployment/static-site-deployment
```

Prikazuje revizije Deploymenta.

---

### Korak 3 — rollback

```bash
kubectl rollout undo deployment/static-site-deployment
kubectl rollout status deployment/static-site-deployment
```

Vraća prethodnu reviziju i čeka da rollback završi.

---

### Korak 4 — dokaz da je image opet `nginx:alpine`

```bash
kubectl describe deployment static-site-deployment | grep -i image
```

ili:

```bash
kubectl get deployment static-site-deployment -o yaml | grep -i image
```

Treba pisati:

```text
nginx:alpine
```

### Typed answer in English

I updated the Deployment image to `nginx:stable-alpine` and followed the rolling update with `kubectl rollout status`. Then I checked the rollout history and used `kubectl rollout undo` to roll back to the previous revision. After rollback, I verified that the running image is again `nginx:alpine`.

---

# LO5 — pitanja s prošlog ispita

## LO5.1 — Podman komanda koristi `--port`

### Pitanje

Explain what is wrong with the following Podman command. Identify the mistake, fix it and explain what was wrong.

```bash
podman run --name mycontainer -d --port 8080:80 httpd
```

### Što je dodatno u odnosu na tvoje glavne dokumente?

U glavnom LO5 dokumentu imaš port mapping problem `host_port:container_port`.

Ovdje je greška drugačija: flag `--port` nije ispravan za Podman. Treba koristiti:

```bash
-p
```

ili:

```bash
--publish
```

---

### Greška

```bash
--port 8080:80
```

Nije ispravan Podman flag za publishanje porta.

### Ispravno

```bash
podman rm -f mycontainer 2>/dev/null || true
podman run --name mycontainer -d -p 8080:80 docker.io/library/httpd
```

ili:

```bash
podman run --name mycontainer -d --publish 8080:80 docker.io/library/httpd
```

### Što radi koja komanda?

```bash
podman rm -f mycontainer 2>/dev/null || true
```

Briše stari container ako postoji.

```bash
podman run --name mycontainer -d -p 8080:80 docker.io/library/httpd
```

Pokreće `httpd` container u backgroundu i mapira host port 8080 na container port 80.

### Provjera

```bash
podman ps
podman port mycontainer
curl localhost:8080
```

### Što gledam u provjeri?

- `podman ps` pokazuje da container radi.
- `podman port mycontainer` pokazuje mapping `8080 -> 80`.
- `curl localhost:8080` mora vratiti httpd stranicu ili HTML odgovor.

### Typed answer in English

The mistake is the `--port` flag. Podman uses `-p` or `--publish` for port mapping. The fixed command maps host port 8080 to container port 80, so the httpd server can be reached on `localhost:8080`.

---

## LO5.2 — Dockerfile: `FRM`, Ubuntu + `yum`, krivi install dio

### Pitanje

Explain what is wrong with the following Dockerfile. Identify the mistake, fix it and explain what was wrong.

```Dockerfile
# Use the official Ubuntu base image
FRM ubuntu:latest

# Set an environment variable
ENV student="your_value_here"

# Install Nginx RUN yum update && \ yum install -y nginx && \ rm -rf /var/lib/apt/lists/*

# Define the running port
EXPOSE 80

# Start Nginx and keep it from running in background
CMD ["nginx", "-g", "daemon off;"]
```

### Što je dodatno u odnosu na tvoje glavne dokumente?

Tvoj LO5 dokument pokriva:
- missing `apt-get update`,
- Dockerfile greške,
- image size cleanup,
- CMD exec form.

Ovdje su dodatne konkretne greške:
1. `FRM` mora biti `FROM`,
2. Ubuntu koristi `apt-get`, ne `yum`,
3. install komanda je zapisana kao komentar, nije pravi `RUN`,
4. cleanup `/var/lib/apt/lists/*` ima smisla s `apt`, ne s `yum`,
5. `ubuntu:latest` radi, ali je bolje pinati verziju, npr. `ubuntu:24.04`.

---

### Ispravljeni Dockerfile

```Dockerfile
FROM ubuntu:latest

ENV student="your_value_here"

RUN apt-get update && \
    apt-get install -y nginx && \
    rm -rf /var/lib/apt/lists/*

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Još bolja verzija s pinned base imageom

```Dockerfile
FROM ubuntu:24.04

ENV student="your_value_here"

RUN apt-get update && \
    apt-get install -y --no-install-recommends nginx && \
    rm -rf /var/lib/apt/lists/*

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Što radi koji dio?

```Dockerfile
FROM ubuntu:latest
```

Postavlja base image. `FRM` je typo i Dockerfile build neće proći.

```Dockerfile
ENV student="your_value_here"
```

Postavlja environment variable.

```Dockerfile
RUN apt-get update && apt-get install -y nginx
```

Na Ubuntu/Debian sustavima koristi se `apt-get`, ne `yum`.

```Dockerfile
rm -rf /var/lib/apt/lists/*
```

Briše apt cache u istom layeru da image bude manji.

```Dockerfile
CMD ["nginx", "-g", "daemon off;"]
```

Pokreće nginx u foregroundu. To je bitno jer container ostaje živ dok radi glavni proces.

---

### Build i provjera

Spremi kao `Dockerfile`, zatim:

```bash
podman build -t nginx-exam-fixed .
podman run -d --name nginx-exam -p 8080:80 nginx-exam-fixed
curl localhost:8080
```

Ako container s tim imenom već postoji:

```bash
podman rm -f nginx-exam
```

### Typed answer in English

The Dockerfile is wrong because `FRM` should be `FROM`, Ubuntu uses `apt-get` instead of `yum`, and the package installation must be written as a real `RUN` instruction. The fixed Dockerfile installs nginx with `apt-get`, cleans the apt cache, exposes port 80, and starts nginx in the foreground with `daemon off`.

---

## LO5.3 — Kubernetes manifest: indentation + selector mismatch

### Pitanje

Explain what is wrong with the following Kubernetes manifest. Identify the mistake, fix it and explain what was wrong.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
name: nginx-deployment
labels:
app: nginx
spec:
replicas: 1
selector:
matchLabels:
app: nginx
template:
metadata:
labels:
app: webserver
spec:
containers:
- name: nginx
image: nginx:latest
ports:
- containerPort: 80
```

### Što je dodatno u odnosu na tvoje glavne dokumente?

Tvoj LO5 dokument već pokriva:
- YAML indentation greške,
- selector mismatch,
- `kubectl apply --dry-run=server --validate=true`.

Ovdje je konkretna kombinacija:
1. indentation je potpuno pokvaren,
2. `selector.matchLabels.app` je `nginx`,
3. ali Pod template label je `webserver`,
4. Deployment ne može upravljati Podovima ako selector ne matcha template labels.

---

### Glavna greška

Deployment selector:

```yaml
selector:
  matchLabels:
    app: nginx
```

mora matchati Pod template labels:

```yaml
template:
  metadata:
    labels:
      app: nginx
```

U lošem manifestu template ima:

```yaml
app: webserver
```

To je mismatch.

---

### Ispravljeni manifest

Spremi kao `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

### Validate prije applya

```bash
kubectl apply --dry-run=server --validate=true -f deployment.yaml
```

### Apply

```bash
kubectl apply -f deployment.yaml
```

### Provjera

```bash
kubectl get deployment nginx-deployment
kubectl get pods -l app=nginx
kubectl describe deployment nginx-deployment
kubectl get deployment nginx-deployment -o jsonpath='{.spec.selector.matchLabels}'; echo
```

### Što gledam u provjeri?

- Deployment postoji.
- Pod postoji s labelom `app=nginx`.
- selector je `app=nginx`.
- nema validation errora.

### Typed answer in English

The manifest is wrong because the YAML indentation is broken and the Deployment selector does not match the Pod template labels. A Deployment must select the Pods it creates, so `spec.selector.matchLabels` and `spec.template.metadata.labels` must match. I fixed the indentation and changed the template label to `app: nginx`.

---

# LO6 — pitanja s prošlog ispita

## LO6.1 — Kubernetes vs Docker Swarm za startup s malo DevOps znanja

### Pitanje

Critically evaluate Kubernetes and Docker Swarm with respect to their scheduling capabilities, fault tolerance, and ease of deployment. Which orchestration system would you recommend for a startup with limited DevOps expertise and why?

### Što je dodatno u odnosu na tvoje glavne dokumente?

Tvoj LO6 dokument već pokriva Kubernetes vs Docker Compose, Kubernetes vs Podman, managed Kubernetes i startup situacije.

Ali ovdje je specifično:

```text
Kubernetes vs Docker Swarm
```

Docker Swarm nije isto što i Docker Compose:
- Compose je uglavnom single-host/multi-container alat.
- Swarm je Dockerov orchestration mode za više nodeova, ali jednostavniji i slabiji od Kubernetes ekosustava.

---

### Odgovor na engleskom

Kubernetes has stronger scheduling capabilities than Docker Swarm. It supports labels, selectors, resource requests, affinity, taints and tolerations, autoscaling integrations, and many workload types. It also has strong fault tolerance because controllers can restart Pods, reschedule workloads and support rolling updates. The disadvantage is that Kubernetes is more complex to deploy and operate.

Docker Swarm is easier to deploy and easier to understand. It can run services across multiple nodes and provides basic scheduling, scaling, service discovery and fault tolerance. For a startup with limited DevOps expertise, this simplicity can be useful. The disadvantage is that Swarm has a smaller ecosystem and fewer advanced features than Kubernetes.

For a small startup, I would recommend Docker Swarm if the application is simple and the team wants the easiest multi-node orchestration setup. However, if the startup expects fast growth, microservices and cloud integrations, I would recommend managed Kubernetes because it gives better long-term scalability while reducing operational overhead.

### Key points

- Kubernetes: stronger scheduling, stronger ecosystem, more features.
- Swarm: easier deployment, simpler operation.
- Startup with limited expertise may prefer simpler tooling at first.
- If growth is expected, managed Kubernetes is better long-term.

---

## LO6.2 — Healthcare hybrid cloud: OpenShift vs Kubernetes

### Pitanje

A healthcare organization must deploy containerized applications in a hybrid cloud environment with strict compliance requirements. Evaluate the suitability of OpenShift and Kubernetes for this use case. Consider compliance, vendor lock-in, and operational complexity in your response.

### Što je dodatno u odnosu na tvoje glavne dokumente?

Ovo je jako blizu tvojih LO6 OpenShift odgovora, ali ovdje moraš posebno naglasiti:
- healthcare,
- compliance,
- hybrid cloud,
- vendor lock-in,
- operational complexity.

---

### Odgovor na engleskom

OpenShift is a strong option for a healthcare organization because it is an enterprise platform built on Kubernetes. It provides stricter default security, Security Context Constraints, RBAC, integrated monitoring, image registry integration, lifecycle management and Red Hat support. These features help with standardization, auditability and compliance in a regulated environment. OpenShift can also run in hybrid cloud environments, so the organization can use a similar platform on-premise and in the cloud.

Vanilla Kubernetes can also be used, but the organization must integrate and maintain more components itself. The team must configure RBAC, admission policies, network policies, monitoring, logging, image scanning, registry, ingress, secrets management, upgrades and compliance processes. This gives more flexibility and less dependency on Red Hat, but it increases operational complexity and the risk of misconfiguration.

The trade-off is that OpenShift reduces integration and compliance work, but increases licensing cost and vendor lock-in. Vanilla Kubernetes gives more freedom, but it needs stronger internal platform engineering knowledge.

For a healthcare organization with strict compliance requirements, I would usually recommend OpenShift if the budget allows it, because support, safer defaults, standardization and auditability are very important in regulated environments.

### Key points

- Healthcare = compliance, auditability, security, support.
- OpenShift = enterprise platform, stronger defaults, Red Hat support.
- Kubernetes = flexible but needs more manual integration.
- Trade-off = OpenShift lock-in/cost vs Kubernetes operational complexity.

---

## LO6.3 — Global e-commerce: Kubernetes over Podman on multiple servers

### Pitanje

You have been tasked with selecting a container orchestration platform for a global e-commerce company expecting rapid scaling and microservice growth. Why would you choose Kubernetes over Podman on multiple servers?

### Što je dodatno u odnosu na tvoje glavne dokumente?

Tvoj LO6 dokument već pokriva Kubernetes vs Podman, ali ovdje je bitno jasno reći:

```text
Podman is a container engine, not full multi-node orchestration.
```

Profesor te može navesti da kažeš “Podman na više servera”, ali tu je poanta da bi to bilo ručno i teško za scale/failover.

---

### Odgovor na engleskom

I would choose Kubernetes because it is designed for orchestration across many servers. It provides scheduling, self-healing, rolling updates, service discovery, load balancing, autoscaling integrations, Secrets, ConfigMaps, storage abstractions and a large ecosystem. These features are important for a global e-commerce company with many microservices and rapid scaling needs.

Podman is good for running containers on a single host, for development, testing and security-focused local container workflows. It is daemonless and supports rootless containers well. However, Podman by itself is not a full multi-node orchestration platform like Kubernetes. Running Podman manually on multiple servers would make scheduling, service discovery, failover, scaling and updates much harder.

For global e-commerce, the platform must handle traffic growth, failures and many services. Kubernetes is better because it automates the cluster-level operations that Podman does not provide by itself.

### Key points

- Kubernetes = cluster orchestration.
- Podman = container engine/local container runtime.
- Kubernetes automates scheduling, failover, service discovery and scaling.
- Global e-commerce needs automation and ecosystem support.

---

# 7 dodatnih sličnih situacija koje se mogu pojaviti

## Extra LO4.1 — Deployment name/image/replicas changed again

### Possible question

Deploy a Deployment named `frontend` with 4 replicas using image `httpd:2.4`.

### Solution

```bash
kubectl delete deployment frontend --ignore-not-found
kubectl create deployment frontend --image=httpd:2.4 --replicas=4
kubectl get deploy frontend
kubectl get pods -l app=frontend
```

### Typed answer in English

I changed the object name, image and replica count according to the question. The Deployment controller creates and maintains four Pods.

---

## Extra LO4.2 — ConfigMap from literal instead of file

### Possible question

Create a ConfigMap named `app-config` with key `MODE=production` and load it into a Pod as environment variables.

### Solution

```bash
kubectl create configmap app-config \
  --from-literal=MODE=production \
  --dry-run=client -o yaml | kubectl apply -f -
```

Example Pod snippet:

```yaml
envFrom:
- configMapRef:
    name: app-config
```

### Typed answer in English

The ConfigMap stores non-sensitive configuration. `envFrom` loads all keys from the ConfigMap as environment variables inside the container.

---

## Extra LO4.3 — Secret instead of ConfigMap

### Possible question

Create a Secret named `db-login` with username and password and mount it into a Pod.

### Solution

```bash
kubectl create secret generic db-login \
  --from-literal=username=admin \
  --from-literal=password=s3cret \
  --dry-run=client -o yaml | kubectl apply -f -

kubectl get secret db-login -o yaml
```

### Typed answer in English

A Secret is used for sensitive values such as passwords. Kubernetes stores Secret data base64-encoded by default, which is encoding, not real encryption.

---

## Extra LO5.1 — Wrong image name or tag

### Possible question

A Pod is in `ImagePullBackOff`. Diagnose and fix it.

### Commands

```bash
kubectl get pods
kubectl describe pod <pod>
kubectl get deployment <deploy-name> -o yaml | grep -i image
```

Fix example:

```bash
CONTAINER=$(kubectl get deploy <deploy-name> -o jsonpath='{.spec.template.spec.containers[0].name}')
kubectl set image deployment/<deploy-name> ${CONTAINER}=nginx:stable
kubectl rollout status deployment/<deploy-name>
```

### Typed answer in English

ImagePullBackOff usually means Kubernetes cannot pull the image. The cause can be a wrong image name, wrong tag, private registry without imagePullSecret, or registry pull limit.

---

## Extra LO5.2 — Service has no endpoints

### Possible question

A Service exists, but it returns nothing. Diagnose and fix it.

### Commands

```bash
kubectl get svc
kubectl get endpoints <service>
kubectl get pods --show-labels
kubectl get svc <service> -o yaml
```

### Fix idea

If Service selector is:

```yaml
selector:
  app: nginx
```

then Pod labels must include:

```yaml
labels:
  app: nginx
```

### Typed answer in English

The Service has no endpoints because its selector does not match any Ready Pod labels. I fix the selector or the Pod labels so the Service can route traffic to the Pods.

---

## Extra LO5.3 — Readiness probe wrong path or port

### Possible question

Deployment Pods are Running but not Ready.

### Commands

```bash
kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>
```

### Fix idea

Check the readiness probe:

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80
```

If the app listens on another port or path, fix the probe.

### Typed answer in English

The Pod can be Running but not Ready if the readiness probe fails. The Service sends traffic only to Ready Pods, so I must fix the probe path, port or the application itself.

---

## Extra LO6.1 — Managed Kubernetes vs self-managed Kubernetes

### Possible question

Compare self-managed Kubernetes with managed Kubernetes for a small company.

### Answer

Self-managed Kubernetes gives more control, but the team must handle upgrades, control plane availability, backups, monitoring, security and node scaling. Managed Kubernetes reduces this operational overhead because the cloud provider manages part of the platform. The trade-off is less control and some cloud-provider dependency.

For a small company, I would usually recommend managed Kubernetes if Kubernetes is really needed, because it reduces operational complexity.

---

# Final exam strategy

If the question is practical, answer in this order:

```text
1. Create/fix the object.
2. Verify with kubectl/podman.
3. Explain what was wrong or what the object does.
4. If the professor changed a name/image/port, adapt the command.
```

If the question is theory, answer in this order:

```text
1. Define both options.
2. Compare strengths.
3. Compare weaknesses.
4. Mention trade-off.
5. Give final recommendation.
```

