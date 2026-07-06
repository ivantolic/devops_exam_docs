pitanje → Ctrl+F u GitHub dokumentima → nađeš primjer → prilagodiš ime/image/port → udariš komande → provjeriš → screenshot → zalijepiš u LibreOffice → ideš dalje
# Safe copy/paste rules for the exam

Ovo su pravila za korištenje ovih dokumenata na ispitu.

## Što je sigurno za copy/paste

Ako pitanje dođe isto kao u dokumentu, naredbe u glavnim Q/A fileovima su napisane da budu sigurne za copy/paste uz ove uvjete:

1. Radiš u istom namespaceu, najčešće `default`.
2. Kubernetes/Podman/Minikube postoje na virtualki.
3. Imageovi se mogu pullati ili imaš registry login.
4. Objekti istog imena nisu već pokvareni od prijašnjih pokušaja.
5. Ako se u pitanju promijeni ime, image ili port, moraš promijeniti i u naredbi.

## Prije zadatka očisti samo ono što treba

Ne radi `kubectl delete all --all` ako nisi siguran, jer možeš obrisati objekte potrebne za druga pitanja.

Za pojedini objekt:

```bash
kubectl delete deploy web --ignore-not-found
kubectl delete svc web --ignore-not-found
kubectl delete pod tmp --ignore-not-found
```

Za Podman container:

```bash
podman rm -f web db app c1 job mysql a b 2>/dev/null || true
```

## Uvijek provjeri ime containera

```bash
kubectl get deploy web -o jsonpath='{.spec.template.spec.containers[*].name}'; echo
```

Zato je safe rolling update:

```bash
CONTAINER=$(kubectl get deploy web -o jsonpath='{.spec.template.spec.containers[0].name}')
kubectl set image deploy/web ${CONTAINER}=nginx:1.27
```

## Ako Service već postoji

```bash
kubectl delete svc web --ignore-not-found
kubectl expose deploy web --port=80 --target-port=80
```

## Ako ConfigMap/Secret već postoji

Koristi apply pattern:

```bash
kubectl create configmap appcfg --from-literal=color=blue --from-literal=mode=dark --dry-run=client -o yaml | kubectl apply -f -
```

```bash
kubectl create secret generic dbcreds --from-literal=username=admin --from-literal=password=s3cret --dry-run=client -o yaml | kubectl apply -f -
```

## Ako YAML faila

```bash
kubectl apply --dry-run=server --validate=true -f file.yaml
```

Zatim gledaj točno polje i indentation.

## Ako Pod ne radi

```bash
kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>
kubectl logs <pod> --previous
kubectl get events --sort-by=.lastTimestamp | tail -30
```

## Ako Service ne radi

```bash
kubectl get svc
kubectl get endpoints <svc>
kubectl get pods --show-labels
kubectl describe svc <svc>
```

Ako endpoints nema, problem je selector/labels/readiness.

## Ako pitanje traži screenshot/proof

Najbolji proofovi:

```bash
kubectl get pods,rs,deploy,svc
kubectl get pods -o wide
kubectl describe pod <pod>
kubectl rollout status deploy/<name>
kubectl get endpoints <svc>
kubectl logs job/<job>
kubectl exec <pod> -- cat <file>
```

KAKO KORISTITI PROVJERE I ŠTO SCREENSHOTATI NA ISPITU

Ovaj dio služi kao podsjetnik kako koristiti komande iz dokumenata na ispitu i kako znati što screenshotati kao dokaz.

U dokumentima su provjere najčešće odmah ispod ili blizu samih komandi koje rješavaju zadatak.

Kada u dokumentu pronađem primjer koji odgovara pitanju, radim ovim redom:

prvo pročitam što pitanje traži
nađem isti ili sličan primjer u dokumentima
pokrenem glavne komande koje rješavaju zadatak
ispod toga tražim dio tipa Verification, Check, Proof, Test, Validate ili Troubleshooting
te provjere pokrenem u terminalu
screenshotam dokaz
zalijepim screenshot u LibreOffice / Word dokument

Razlika između komande koja rješava zadatak i komande koja dokazuje zadatak:

Primjer:

kubectl create deployment web --image=nginx:alpine --replicas=2

Ova komanda rješava zadatak jer kreira Deployment.

Ali profesor ne vidi da je sve ispravno dok ne pokažem dokaz.

Zato nakon toga radim provjere:

kubectl get deployment web
kubectl get pods -l app=web
kubectl describe deployment web

To su komande koje screenshotam jer dokazuju da Deployment postoji, da ima Podove i da koristi ispravan image / konfiguraciju.

Ne treba screenshotati svaku komandu koju sam ikad pokrenuo.

Najbitnije je screenshotati:

finalnu create/apply/patch/set image komandu
završnu provjeru koja dokazuje da objekt radi
output gdje se jasno vidi ime objekta, status, image, port, replicas ili file content

Dobar screenshot treba pokazati:

komandu koju sam pokrenuo
rezultat komande
dokaz da je zadatak stvarno napravljen

Ako u dokumentu nakon rješenja piše nešto poput Verification, Check, Proof, Test ili Validate, onda su to komande koje trebam pokrenuti i screenshotati.

Primjer:

kubectl apply -f yaml/redis.yaml
kubectl get pods -l app=redis
kubectl get svc redis
kubectl get statefulset redis

Ovdje screenshot pokazuje:

da sam applyao YAML manifest
da Redis Podovi postoje
da Redis Service postoji
da StatefulSet postoji

To je dobar dokaz.

Ako primjer nema posebno napisanu provjeru, onda sam napravim završnu provjeru.

Najsigurnije default provjere su:

kubectl get all
kubectl get pods --show-labels
kubectl describe <kind> <name>

ili specifično:

kubectl get <kind> <name>
kubectl describe <kind> <name>

Primjeri:

kubectl get deployment web
kubectl describe deployment web

kubectl get pod <pod-name>
kubectl describe pod <pod-name>

kubectl get svc web
kubectl get endpoints web

Ako pitanje traži Deployment, screenshotam:

kubectl get deployment <deployment-name>
kubectl get pods -l app=<label>
kubectl describe deployment <deployment-name>

Primjer:

kubectl get deployment static-site-deployment
kubectl get pods -l app=static-site-deployment
kubectl describe deployment static-site-deployment

Dokaz mora pokazati:

ime Deploymenta
broj replika
Podove
image ako je bitan

Ako pitanje traži Pod, screenshotam:

kubectl get pod <pod-name>
kubectl describe pod <pod-name>

Ako trebam dokazati da container radi:

kubectl logs <pod-name>

Ako pitanje traži Service, screenshotam:

kubectl get svc
kubectl get svc <service-name>
kubectl get endpoints <service-name>

Najbitnija provjera za Service je:

kubectl get endpoints <service-name>

Ako endpoints nema IP adrese, Service najčešće ne pogađa Podove zbog krivog selectora ili labela.

Za test iz clustera:

kubectl run tmp --rm -it --image=busybox -- wget -qO- <service-name>:<port>

Ako pitanje traži ConfigMap mount, screenshotam:

kubectl get configmap <configmap-name>
kubectl describe configmap <configmap-name>

Ali najbitniji dokaz je provjera iz Poda:

kubectl exec <pod-name> -- cat <path>

Primjer:

kubectl exec "$POD" -- cat /usr/share/nginx/html/index.html

Ako se vidi sadržaj filea, mount radi.

Ako pitanje traži Secret, screenshotam:

kubectl get secret <secret-name>
kubectl describe secret <secret-name>

Ako je Secret mountan u Pod, provjeravam iz Poda:

kubectl exec <pod-name> -- ls <mount-path>
kubectl exec <pod-name> -- cat <mount-path>/<key-name>

Napomena: Secret je base64 encoded, nije prava enkripcija. Ne treba prikazivati osjetljive vrijednosti ako nije potrebno.

Ako pitanje traži rolling update, screenshotam:

kubectl set image deployment/<deployment-name> <container-name>=<new-image>
kubectl rollout status deployment/<deployment-name>
kubectl rollout history deployment/<deployment-name>
kubectl describe deployment <deployment-name> | grep -i image

Sigurnija varijanta gdje prvo dohvatim ime containera:

CONTAINER=$(kubectl get deploy static-site-deployment -o jsonpath='{.spec.template.spec.containers[0].name}')
kubectl set image deployment/static-site-deployment ${CONTAINER}=nginx:stable-alpine
kubectl rollout status deployment/static-site-deployment
kubectl rollout history deployment/static-site-deployment
kubectl describe deployment static-site-deployment | grep -i image

Dokaz mora pokazati da je image promijenjen.

Ako pitanje traži rollback, screenshotam:

kubectl rollout undo deployment/<deployment-name>
kubectl rollout status deployment/<deployment-name>
kubectl rollout history deployment/<deployment-name>
kubectl describe deployment <deployment-name> | grep -i image

Primjer:

kubectl rollout undo deployment/static-site-deployment
kubectl rollout status deployment/static-site-deployment
kubectl rollout history deployment/static-site-deployment
kubectl describe deployment static-site-deployment | grep -i image

Dokaz mora pokazati da se image vratio na prethodnu verziju.

Ako pitanje traži StatefulSet, screenshotam:

kubectl get statefulset <name>
kubectl get pods -l app=<label>
kubectl get svc <service-name>

Ako je headless Service bitan:

kubectl get svc <service-name>

Treba se vidjeti:

CLUSTER-IP: None

Ako treba DNS dokaz:

kubectl run tmp --rm -it --image=busybox -- nslookup <pod-name>.<service-name>.default.svc.cluster.local

Primjer:

kubectl run tmp --rm -it --image=busybox -- nslookup redis-0.redis.default.svc.cluster.local

Ako pitanje traži DaemonSet, screenshotam:

kubectl get daemonset
kubectl get pods -o wide
kubectl describe daemonset <name>

Bitno je pokazati da se Podovi raspoređuju po nodeovima.

Ako pitanje traži Job, screenshotam:

kubectl get job
kubectl get pods
kubectl logs <job-pod-name>

Bitno je pokazati da je Job završio, npr. Completed.

Ako pitanje traži CronJob, screenshotam:

kubectl get cronjob
kubectl describe cronjob <name>

Ako se Job već pokrenuo:

kubectl get jobs
kubectl get pods
kubectl logs <pod-name>

Ako pitanje traži readinessProbe, livenessProbe ili startupProbe, screenshotam:

kubectl get pods
kubectl describe pod <pod-name>

U describe outputu treba se vidjeti probe konfiguracija ili eventovi.

Ako Service ne šalje promet na Pod, provjeravam:

kubectl get endpoints <service-name>

Ako readinessProbe faila, endpointi često neće sadržavati taj Pod.

Ako pitanje traži PVC, screenshotam:

kubectl get pvc
kubectl describe pvc <pvc-name>
kubectl get pods

Ako je PVC mountan u Pod:

kubectl describe pod <pod-name>

ili provjera iz Poda:

kubectl exec <pod-name> -- df -h
kubectl exec <pod-name> -- mount

Kod LO5 troubleshooting zadataka uvijek treba pokazati:

problem
uzrok
fix
verification
kratko objašnjenje

Ako je greška Podman port, npr.:

podman run --name mycontainer -d --port 8080:80 httpd

Screenshotam fixed command i provjeru:

podman rm -f mycontainer 2>/dev/null || true
podman run --name mycontainer -d -p 8080:80 docker.io/library/httpd
podman ps
podman port mycontainer
curl localhost:8080

Explanation:

The mistake is the --port flag. Podman uses -p or --publish for port publishing. The fixed command maps host port 8080 to container port 80.

Ako popravljam Dockerfile / Containerfile, screenshotam:

cat Dockerfile
podman build -t fixed-image .
podman run -d --name test-container -p 8080:80 fixed-image
curl localhost:8080

Za greške tipa FRM i yum na Ubuntu:

FRM must be FROM.
Ubuntu uses apt-get, not yum.
The install command must be inside a RUN instruction.

Ako je ImagePullBackOff / ImageInspectError, screenshotam:

kubectl get pods
kubectl describe pod <pod-name>
kubectl get events --sort-by=.lastTimestamp | tail -30

Fix najčešće uključuje:

ispravan image name
ispravan image tag
docker.io/library/... za public image ako treba
imagePullSecret ako je private image

Ako je CrashLoopBackOff, screenshotam:

kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl logs <pod-name> --previous

Explanation:

CrashLoopBackOff means the container starts and then crashes repeatedly. Logs and describe output are used to find the application error or wrong command.

Ako je Pod Pending, screenshotam:

kubectl get pods
kubectl describe pod <pod-name>
kubectl get events --sort-by=.lastTimestamp | tail -30

Najčešći uzroci:

insufficient CPU/memory
PVC not bound
nodeSelector mismatch
taints/tolerations
no suitable node

Ako Service nema promet, screenshotam:

kubectl get svc
kubectl get endpoints <service-name>
kubectl get pods --show-labels
kubectl describe svc <service-name>

Ako endpoints nema IP adrese, najčešći problem je selector mismatch.

Explanation:

The Service selector must match the Pod labels. If they do not match, the Service has no endpoints and cannot send traffic to the Pods.

Za svaki praktični zadatak u LibreOffice / Word dokumentu koristim strukturu:

LO4.X

Commands used:
[screenshot]

Verification:
[screenshot]

Short explanation:
The requested object was created and verified using kubectl get/describe/exec commands.

Za troubleshooting koristim strukturu:

LO5.X

Mistake:
...

Fix:
...

Verification:
[screenshot]

Explanation:
...

Za LO6 koristim strukturu:

LO6.X

Answer:
[theory answer in English]

Najvažnije pravilo: ne prepisujem slijepo.

Prije copy-pastea uvijek provjerim iz pitanja:

ime objekta
image
replicas
port
targetPort
nodePort
namespace
label
selector
mountPath
ConfigMap name
Secret name
container name

Ako se nešto razlikuje od primjera u dokumentu, moram to promijeniti.

Najčešća greška je da u dokumentu piše web, a u pitanju traži static-site-deployment.

Ako ne znam što točno screenshotati, koristim brzi default proof:

kubectl get all
kubectl get pods --show-labels
kubectl describe <kind> <name>

Za Service dodatno:

kubectl get endpoints <service-name>

Za file mount dodatno:

kubectl exec <pod-name> -- cat <path>

Za rollout dodatno:

kubectl rollout history deployment/<name>
kubectl describe deployment <name> | grep -i image

Finalni workflow na ispitu:

pročitam pitanje
otvorim GitHub dokument
Ctrl + F ključna riječ iz pitanja
nađem isti ili sličan primjer
kopiram komande/YAML
promijenim ime/image/port/replicas/label ako pitanje traži
pokrenem komande u terminalu
pokrenem verification/proof komande
screenshotam dokaz
zalijepim u LibreOffice / Word
napišem kratko objašnjenje ako treba
idem na sljedeći zadatak

Rješenje nije gotovo kad pokrenem glavnu komandu.
