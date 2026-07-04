# Komentirani yaml/ folder za DevOps ispit

Ovaj folder sadrži iste Kubernetes YAML manifeste kao raniji `yaml/` folder, ali sada svaki file ima komentare.

Komentari objašnjavaju:
- što taj Kubernetes objekt radi,
- zašto se koristi baš taj `kind`,
- što rade `selector` i `labels`,
- zašto treba `volumeMounts`, `emptyDir`, `headless Service`, `volumeClaimTemplates`, probe itd.,
- što profesor može pitati kao objašnjenje.

Na ispitu i dalje koristiš iste naredbe iz LO4 dokumenta, npr.:

```bash
kubectl apply -f yaml/redis.yaml
kubectl apply -f yaml/web-sidecar.yaml
kubectl apply -f yaml/httpd.yaml
```

Razlika je samo što sada možeš otvoriti YAML file i odmah vidjeti što radi koji dio.

Kubernetes ignorira linije koje počinju s `#`, tako da komentari ne smetaju izvršavanju.
