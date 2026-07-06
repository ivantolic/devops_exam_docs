# PROCEDURA — Ručno kreiranje YAML fileova na ispitu

Ovaj dokument je za situaciju kada na ispitu ne želiš skidati cijeli `yaml/` folder s GitHuba, nego želiš ručno napraviti YAML file u VM-u i deployati ga s `kubectl apply -f`.

Koristi ga kada u glavnim dokumentima vidiš komandu tipa:

```bash
kubectl apply -f yaml/redis.yaml
kubectl apply -f yaml/web-sidecar.yaml
kubectl apply -f yaml/httpd.yaml
```

To znači: u trenutnom folderu mora postojati folder `yaml/`, a u njemu file npr. `redis.yaml`.

---

## 1. Što je YAML file?

YAML file je običan tekstualni file u kojem piše što Kubernetes treba napraviti.

Mentalni model:

```text
YAML file = recept
kubectl apply -f = daj Kubernetesu recept da napravi objekt
```

Primjer:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 2
```

Kubernetes neće ništa napraviti dok mu ne daš file:

```bash
kubectl apply -f ime-filea.yaml
```

Ako je file u folderu `yaml`, onda:

```bash
kubectl apply -f yaml/ime-filea.yaml
```

---

## 2. Kada radiš YAML ručno?

YAML ručno radiš kada pitanje traži nešto složenije, npr.:

```text
StatefulSet
DaemonSet
multi-container Pod
sidecar container
ConfigMap volume
Secret volume
subPath mount
initContainer
readOnly volumeMount
emptyDir sizeLimit
Deployment + Service u istom fileu
```

Za jednostavan Deployment često ti je dovoljan imperative command:

```bash
kubectl create deployment static-site-deployment --image=nginx:alpine --replicas=2
```

---

## 3. Glavni postupak za svaki YAML zadatak

Uvijek idi ovim redom:

```text
1. napravi folder yaml
2. napravi YAML file
3. zalijepi YAML sadržaj
4. spremi file
5. validiraj YAML
6. apply YAML
7. provjeri rezultat
8. screenshotaj dokaz
```

---

## 4. Napravi folder `yaml`

```bash
mkdir -p yaml
```

Objašnjenje:

```text
mkdir = napravi folder
-p = ako folder već postoji, ne bacaj error
yaml = ime foldera
```

Provjera:

```bash
ls
```

Trebaš vidjeti:

```text
yaml
```

---

## 5. Napravi YAML file — opcija A: nano

Primjer:

```bash
nano yaml/redis.yaml
```

Otvori se editor u terminalu.

Zalijepi YAML sadržaj.

Spremanje:

```text
Ctrl + O
Enter
Ctrl + X
```

Značenje:

```text
Ctrl + O = spremi
Enter = potvrdi ime filea
Ctrl + X = izađi
```

---

## 6. Napravi YAML file — opcija B: cat EOF

Ovo je često brže jer samo zalijepiš cijeli blok.

Primjer:

```bash
mkdir -p yaml

cat > yaml/redis.yaml <<'EOF'
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  clusterIP: None
  selector:
    app: redis
  ports:
  - name: redis
    port: 6379
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
spec:
  serviceName: redis
  replicas: 3
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7
        ports:
        - name: redis
          containerPort: 6379
EOF
```

Bitno:

```text
EOF na kraju mora biti sam u liniji.
Ne smije imati razmake ispred.
```

---

## 7. Provjeri da file postoji

```bash
ls yaml
```

Trebaš vidjeti npr.:

```text
redis.yaml
```

Pogledaj sadržaj:

```bash
cat yaml/redis.yaml
```

ili:

```bash
head -20 yaml/redis.yaml
```

Ako file nije tamo, nisi ga dobro spremio ili si u krivom folderu.

---

## 8. Validiraj YAML prije deploya

Prije pravog applya pokreni:

```bash
kubectl apply --dry-run=server --validate=true -f yaml/redis.yaml
```

Objašnjenje:

```text
--dry-run=server = Kubernetes provjeri manifest, ali ga ne napravi stvarno
--validate=true = provjeri fieldove i schema
-f yaml/redis.yaml = koristi taj file
```

Ako prođe bez errora, tek onda deployaj.

Ako faila, najčešći razlozi su:

```text
kriva indentacija
tabovi umjesto razmaka
selector ne matcha labels
krivi apiVersion/kind
field je na krivoj razini
fali metadata.name
```

---

## 9. Deployaj YAML

```bash
kubectl apply -f yaml/redis.yaml
```

Mogući output:

```text
service/redis created
statefulset.apps/redis created
```

ili:

```text
service/redis configured
statefulset.apps/redis configured
```

`created` znači da je prvi put napravljeno.

`configured` znači da je objekt već postojao i sada je ažuriran.

---

## 10. Provjeri rezultat

Za StatefulSet:

```bash
kubectl get statefulset redis
kubectl get pods -l app=redis
kubectl get svc redis
```

Za Deployment:

```bash
kubectl get deployment
kubectl get pods
kubectl describe deployment <deployment-name>
```

Za Service:

```bash
kubectl get svc
kubectl get endpoints <service-name>
```

Za ConfigMap/Secret mount:

```bash
kubectl exec <pod-name> -- cat <path>
```

Za više containera u Podu:

```bash
kubectl exec <pod-name> -c <container-name> -- <command>
```

---

## 11. Što screenshotati?

Screenshot treba pokazati:

```text
komandu
rezultat komande
dokaz da objekt radi
```

Dobri proofovi:

```bash
kubectl apply -f yaml/redis.yaml
kubectl get pods -l app=redis
kubectl get svc redis
kubectl get statefulset redis
```

Za ConfigMap file:

```bash
kubectl exec <pod> -- cat /usr/share/nginx/html/index.html
```

Za rollout:

```bash
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl describe deployment <name> | grep -i image
```

Za Service:

```bash
kubectl get endpoints <service>
kubectl run tmp --rm -it --image=busybox -- wget -qO- <service>:<port>
```

---

## 12. Najveće YAML zamke

### Zamka 1 — nisi u pravom folderu

Ako dobiješ:

```text
the path "yaml/redis.yaml" does not exist
```

pokreni:

```bash
pwd
ls
ls yaml
```

Ako `ls` ne pokazuje folder `yaml`, nisi u pravom folderu ili folder ne postoji.

---

### Zamka 2 — krivi naziv filea

Ovo nije isto:

```text
redis.yml
redis.yaml
```

Ako napraviš `redis.yml`, a pokreneš `redis.yaml`, failat će.

Provjeri:

```bash
ls yaml
```

---

### Zamka 3 — tabovi umjesto razmaka

YAML koristi razmake. Nemoj koristiti tabove.

Ako kopiraš iz browsera, pazi da se razmaci nisu raspali.

---

### Zamka 4 — selector ne matcha labels

Ako Deployment ima:

```yaml
selector:
  matchLabels:
    app: nginx
```

onda template mora imati:

```yaml
template:
  metadata:
    labels:
      app: nginx
```

Ako je jedno `app: nginx`, a drugo `app: webserver`, krivo je.

---

### Zamka 5 — ime/image/replicas nisu kao u pitanju

Ako profesor kaže:

```text
Deployment named static-site-deployment with image nginx:alpine and 2 replicas
```

onda ne smije ostati:

```yaml
name: web
image: nginx:1.25
replicas: 3
```

Promijeni točno prema pitanju.

---

### Zamka 6 — namespace

Ako pitanje kaže namespace `dev`, onda koristi:

```bash
kubectl create namespace dev
kubectl apply -n dev -f yaml/file.yaml
```

ili u YAML dodaj:

```yaml
metadata:
  namespace: dev
```

Ne radi sve u `default` ako pitanje traži drugi namespace.

---

### Zamka 7 — subPath mount

Ako pitanje kaže:

```text
mount index.html at /usr/share/nginx/html/index.html using subPath
```

onda moraš imati file mount:

```yaml
volumeMounts:
- name: site-content
  mountPath: /usr/share/nginx/html/index.html
  subPath: index.html
```

Ako mountaš cijeli folder `/usr/share/nginx/html`, to nije isto.

---

## 13. Kako promijeniti YAML ako profesor promijeni uvjete?

Otvori file:

```bash
nano yaml/redis.yaml
```

Najčešće mijenjaš:

```yaml
metadata:
  name: ...
```

```yaml
replicas: ...
```

```yaml
image: ...
```

```yaml
labels:
  app: ...
```

```yaml
selector:
  matchLabels:
    app: ...
```

```yaml
containerPort: ...
```

Spremi:

```text
Ctrl + O
Enter
Ctrl + X
```

Validiraj:

```bash
kubectl apply --dry-run=server --validate=true -f yaml/redis.yaml
```

Deploy:

```bash
kubectl apply -f yaml/redis.yaml
```

Proof:

```bash
kubectl get pods
kubectl describe pod <pod>
```

---

## 14. Primjer — Redis StatefulSet + headless Service

Koristi ako pitanje traži StatefulSet za Redis plus headless Service.

```bash
mkdir -p yaml

cat > yaml/redis.yaml <<'EOF'
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  clusterIP: None
  selector:
    app: redis
  ports:
  - name: redis
    port: 6379
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
spec:
  serviceName: redis
  replicas: 3
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7
        ports:
        - name: redis
          containerPort: 6379
EOF

kubectl apply --dry-run=server --validate=true -f yaml/redis.yaml
kubectl apply -f yaml/redis.yaml
kubectl get pods -l app=redis
kubectl get svc redis
kubectl get statefulset redis
```

DNS proof:

```bash
kubectl run tmp --rm -it --image=busybox -- nslookup redis-0.redis.default.svc.cluster.local
```

English explanation:

```text
This manifest creates a headless Service and a StatefulSet. The headless Service gives stable DNS records for StatefulSet Pods, and the StatefulSet creates stable Pod names such as redis-0, redis-1 and redis-2.
```

---

## 15. Primjer — sidecar Deployment

Koristi ako pitanje traži sidecar container ili dva containera koji dijele volume.

```bash
mkdir -p yaml

cat > yaml/web-sidecar.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-sidecar
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web-sidecar
  template:
    metadata:
      labels:
        app: web-sidecar
    spec:
      volumes:
      - name: shared
        emptyDir: {}
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        volumeMounts:
        - name: shared
          mountPath: /usr/share/nginx/html
      - name: sidecar
        image: busybox
        command: ["sh", "-c", "while true; do date > /data/index.html; sleep 10; done"]
        volumeMounts:
        - name: shared
          mountPath: /data
EOF

kubectl apply --dry-run=server --validate=true -f yaml/web-sidecar.yaml
kubectl apply -f yaml/web-sidecar.yaml
kubectl get pods -l app=web-sidecar
```

Proof:

```bash
POD=$(kubectl get pod -l app=web-sidecar -o jsonpath='{.items[0].metadata.name}')
kubectl exec "$POD" -c sidecar -- cat /data/index.html
kubectl exec "$POD" -c nginx -- sh -c 'wget -qO- localhost:80'
```

English explanation:

```text
The Deployment creates a Pod with two containers. Both containers share the same Pod network and the same emptyDir volume. The sidecar writes index.html into the shared volume, and nginx serves that file.
```

---

## 16. Primjer — ConfigMap subPath za nginx index.html

Koristi ako pitanje traži ConfigMap s keyem `index.html` i mount preko `subPath`.

Create Deployment:

```bash
kubectl delete deployment static-site-deployment --ignore-not-found
kubectl create deployment static-site-deployment --image=nginx:alpine --replicas=2
```

Create HTML file:

```bash
cat > index.html <<'EOF'
<html>
  <body>
    <h1>Ivan Tolic - Exam Group A</h1>
  </body>
</html>
EOF
```

Create ConfigMap:

```bash
kubectl create configmap site-content   --from-file=index.html=index.html   --dry-run=client -o yaml | kubectl apply -f -
```

Export Deployment YAML:

```bash
kubectl get deployment static-site-deployment -o yaml > static-site-deployment.yaml
```

Open it:

```bash
nano static-site-deployment.yaml
```

Add under `spec.template.spec`:

```yaml
volumes:
- name: site-content
  configMap:
    name: site-content
    items:
    - key: index.html
      path: index.html
```

Add under the nginx container:

```yaml
volumeMounts:
- name: site-content
  mountPath: /usr/share/nginx/html/index.html
  subPath: index.html
```

Save:

```text
Ctrl + O
Enter
Ctrl + X
```

Apply:

```bash
kubectl apply --dry-run=server --validate=true -f static-site-deployment.yaml
kubectl apply -f static-site-deployment.yaml
kubectl rollout status deployment/static-site-deployment
```

Proof:

```bash
POD=$(kubectl get pod -l app=static-site-deployment -o jsonpath='{.items[0].metadata.name}')
kubectl exec "$POD" -- cat /usr/share/nginx/html/index.html
```

Optional web proof:

```bash
kubectl port-forward deployment/static-site-deployment 8080:80
```

Open in browser inside VM:

```text
http://localhost:8080
```

English explanation:

```text
I created a ConfigMap named site-content with the key index.html. I mounted only that key into the nginx container using subPath, so the file appears at /usr/share/nginx/html/index.html. I verified it with kubectl exec by reading the file inside the Pod.
```

---

## 17. Primjer — httpd Deployment s nodeSelectorom

```bash
mkdir -p yaml

cat > yaml/httpd.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
spec:
  replicas: 2
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      nodeSelector:
        disktype: ssd
      containers:
      - name: httpd
        image: httpd:2.4
        ports:
        - name: http
          containerPort: 80
EOF

kubectl apply --dry-run=server --validate=true -f yaml/httpd.yaml
kubectl apply -f yaml/httpd.yaml
kubectl get pods -l app=httpd
kubectl describe pod -l app=httpd
```

Ako su Podovi Pending jer node nema labelu:

```bash
kubectl get nodes
kubectl label node <node-name> disktype=ssd
kubectl get pods -l app=httpd
```

English explanation:

```text
The Deployment uses nodeSelector, so its Pods can only run on nodes with the label disktype=ssd. If no node has that label, the Pods stay Pending until the node is labeled or the selector is changed.
```

---

## 18. Ako se pogubiš na ispitu

Radi ovim redom:

```bash
pwd
ls
ls yaml
```

Ako file postoji:

```bash
cat yaml/file.yaml
```

Validacija:

```bash
kubectl apply --dry-run=server --validate=true -f yaml/file.yaml
```

Apply:

```bash
kubectl apply -f yaml/file.yaml
```

Ako Pod ne radi:

```bash
kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>
kubectl get events --sort-by=.lastTimestamp | tail -30
```

Ako Service ne radi:

```bash
kubectl get svc
kubectl get endpoints <service>
kubectl get pods --show-labels
```

Ako Deployment ne updatea:

```bash
kubectl rollout status deployment/<name>
kubectl describe deployment <name>
kubectl get rs
kubectl get pods
```

---

## 19. Finalna rečenica za glavu

Za YAML zadatak uvijek:

```text
1. napravi file
2. validiraj file
3. apply file
4. provjeri objekt
5. screenshotaj dokaz
6. napiši kratko što objekt radi
```

Ne moraš znati sve napamet. Moraš znati promijeniti ime, image, port, label i namespace ako pitanje to traži.
