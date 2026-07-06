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
