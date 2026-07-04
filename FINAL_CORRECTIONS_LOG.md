# Final corrections log

Ova finalna verzija uključuje prethodne ispravke i dodatne zaštite za alternativna pitanja.

## Prethodne ključne ispravke koje ostaju

- `kubectl set image` više ne pogađa ime containera napamet, nego ga izvlači s `jsonpath`.
- `Recreate` strategy patch briše `rollingUpdate` blok pomoću `rollingUpdate:null`.
- `redis-readiness` label mismatch je ispravljen.
- `startupProbe` ima ispravno odvojene linije `failureThreshold` i `periodSeconds`.
- `redis-pvc.yaml` ima `volumeClaimTemplates` na ispravnoj StatefulSet razini.
- `shop.yaml` ima ispravan `kind` i `metadata`, bez spojenog `Deploymentmetadata`.
- `envFrom.secretRef.name` je pravilno indentiran.
- Secret/ConfigMap primjeri koriste `--dry-run=client -o yaml | kubectl apply -f -` gdje je korisno.
- Namespace `other` se kreira idempotentno gdje je potrebno.

## Novo dodano u finalnoj verziji

- `EXAM_TRAPS_AND_ALTERNATIVE_QUESTIONS.md` s fallback naredbama ako profesor promijeni ime, image, port, namespace, broj replika ili tip pitanja.
- `SAFE_COPY_PASTE_RULES.md` s pravilima za sigurno korištenje na ispitu.
- Na kraj LO4/LO5/LO6 dodan je kratki dio što napraviti ako pitanje nije potpuno isto.
- README je ažuriran da jasno kaže koji file koristiti za koju situaciju.

## Provjera

- Svi YAML fileovi u folderu `yaml/` su sintaktički provjereni YAML parserom.
- Naredbe su logički usklađene s pripadajućim pitanjima i dodani su najvažniji preduvjeti/fallbackovi.

## Granica garancije

Za identično pitanje naredbe su pripremljene da budu safe copy/paste. Ipak, nijedan dokument ne može garantirati izvršavanje na nepoznatoj virtualki ako: nema interneta, image pull ne radi, Minikube nije dostupan, koristi se drugi namespace, objekt već postoji u krivom stanju ili profesor promijeni uvjete zadatka.
