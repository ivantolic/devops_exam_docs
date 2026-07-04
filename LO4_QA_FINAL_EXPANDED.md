# LO4 — FINAL EXPANDED Q/A WITH COMPLETE QUESTIONS AND EXPLAINED COMMANDS

# IMPORTANT EXAM USE RULES

These documents are written for the Intro to DevOps LO4/LO5 exam practice questions.
They are intentionally more explicit than a normal cheat sheet because they explain:

- the full original-style question,
- what the task is asking,
- exact commands to run,
- what each command is doing,
- how to verify that the fix/task worked,
- and a short English answer you can type if the professor asks for explanation.

General rule: if the professor changes the object name, image, tag, port, namespace or number of replicas, replace those values in the commands.

Before practical Kubernetes tasks, check the cluster:

```bash
kubectl get nodes
kubectl get pods,rs,deploy,svc
kubectl get events --sort-by=.lastTimestamp | tail -20
```

Before Podman troubleshooting tasks, check existing containers:

```bash
podman ps -a
```



## How to answer LO4 if professor asks you to type explanation

Use simple English: say what object you created, why it is needed, how you verified it, and what Kubernetes behavior it proves.


---

## LO4.1 — Create a Deployment named web running nginx:1.25 with 3 replicas using...

**Full question:** Create a Deployment named web running nginx:1.25 with 3 replicas using a single imperative kubectl create deployment command, then export its generated YAML to a file.

**What the task wants:** Create a Deployment with three nginx Pods and save the generated YAML so it can be edited or submitted.

**Commands and what they do:**

```bash
kubectl delete deployment web --ignore-not-found
# Creates the Deployment named web with image nginx:1.25 and 3 desired replicas.
kubectl create deployment web --image=nginx:1.25 --replicas=3
# Exports the live Deployment object from the cluster into a YAML file.
kubectl get deployment web -o yaml > web.yaml
```

**Command-by-command explanation:**
- `kubectl delete deployment web --ignore-not-found` — removes the old object if it exists, so the next create/apply command does not fail because of a name conflict.
- `kubectl create deployment web --image=nginx:1.25 --replicas=3` — creates the Deployment imperatively with the requested image and replica count.
- `kubectl get deployment web -o yaml > web.yaml` — exports the live Deployment YAML so you can inspect or edit the manifest.

**Verification / proof that it worked:**

```bash
kubectl get deploy web
kubectl get pods -l app=web
cat web.yaml | head -40
```

**Typed answer in English:**
The Deployment named web runs nginx:1.25 with three replicas. The YAML export shows the live Kubernetes object in manifest form.


---

## LO4.2 — Enable pulling images from DockerHub as an authenticated user to lower...

**Full question:** Enable pulling images from DockerHub as an authenticated user to lower the chance of hitting the pull limit. What kind of resource do you need to create?

**What the task wants:** Create a docker-registry Secret and attach it to the default ServiceAccount or to a Pod/Deployment.

**Commands and what they do:**

```bash
# Creates a Secret of type kubernetes.io/dockerconfigjson with DockerHub credentials.
kubectl create secret docker-registry dockerhub \
  --docker-server=docker.io \
  --docker-username=USER \
  --docker-password=PASS \
  --docker-email=email@example.com \
  --dry-run=client -o yaml | kubectl apply -f -
# Makes Pods in this namespace use that Secret when pulling images.
kubectl patch serviceaccount default -p '{"imagePullSecrets":[{"name":"dockerhub"}]}'
```

**Command-by-command explanation:**
- `kubectl create secret docker-registry dockerhub` — creates an image-pull Secret for authenticated registry access.
- `--docker-server=docker.io` — run this command as part of the task; then use the verification commands to confirm the result.
- `--docker-username=USER` — run this command as part of the task; then use the verification commands to confirm the result.
- `--docker-password=PASS` — run this command as part of the task; then use the verification commands to confirm the result.
- `--docker-email=email@example.com` — run this command as part of the task; then use the verification commands to confirm the result.
- `--dry-run=client -o yaml | kubectl apply -f -` — run this command as part of the task; then use the verification commands to confirm the result.
- `kubectl patch serviceaccount default -p '{"imagePullSecrets":[{"name":"dockerhub"}]}'` — patches only the requested fields on an existing Kubernetes object.

**Verification / proof that it worked:**

```bash
kubectl get secret dockerhub -o jsonpath='{.type}'; echo
kubectl get sa default -o jsonpath='{.imagePullSecrets}'; echo
```

**Typed answer in English:**
The needed resource is a docker-registry Secret. Kubernetes uses it as imagePullSecrets for authenticated image pulls.


---

## LO4.3 — Scale web from 3 to 5 replicas two different ways — once with kubectl ...

**Full question:** Scale web from 3 to 5 replicas two different ways — once with kubectl scale, once by editing the manifest — and show the ReplicaSet that results.

**What the task wants:** Change spec.replicas imperatively and also through YAML. Scaling alone should not create a new ReplicaSet because the Pod template did not change.

**Commands and what they do:**

```bash
# Imperative scaling: changes only the replica count.
kubectl scale deployment web --replicas=5
# Export YAML, edit spec.replicas if needed, and apply it again.
kubectl get deployment web -o yaml > web.yaml
# In web.yaml set: spec.replicas: 5
kubectl apply -f web.yaml
```

**Command-by-command explanation:**
- `kubectl scale deployment web --replicas=5` — changes the desired replica count for the controller.
- `kubectl get deployment web -o yaml > web.yaml` — exports the live Deployment YAML so you can inspect or edit the manifest.
- `kubectl apply -f web.yaml` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.

**Verification / proof that it worked:**

```bash
kubectl get deploy web
kubectl get pods -l app=web
kubectl get rs -l app=web
```

**Typed answer in English:**
Both methods change the desired replica count to 5. Because the Pod template is not changed, the existing ReplicaSet normally scales instead of creating a new revision.


---

## LO4.4 — Perform a rolling update of web from nginx:1.25 to nginx:1.27 and watc...

**Full question:** Perform a rolling update of web from nginx:1.25 to nginx:1.27 and watch it with kubectl rollout status.

**What the task wants:** Change the container image and watch Kubernetes replace old Pods with new Pods gradually.

**Commands and what they do:**

```bash
# First read the real container name. Do not guess it.
CONTAINER=$(kubectl get deploy web -o jsonpath='{.spec.template.spec.containers[0].name}')
# Updates that container image to nginx:1.27.
kubectl set image deployment/web ${CONTAINER}=nginx:1.27
# Watches until the rolling update finishes or gets stuck.
kubectl rollout status deployment/web
```

**Command-by-command explanation:**
- `CONTAINER=$(kubectl get deploy web -o jsonpath='{.spec.template.spec.containers[0].name}')` — stores the real container name in a shell variable; this avoids guessing whether the container is called `web`, `nginx`, etc.
- `kubectl set image deployment/web ${CONTAINER}=nginx:1.27` — changes the image in the Deployment Pod template, which starts a new rollout.
- `kubectl rollout status deployment/web` — waits for the rollout to finish or shows that the rollout is stuck.

**Verification / proof that it worked:**

```bash
kubectl get rs -l app=web
kubectl get pods -l app=web
kubectl describe deployment web | grep -i image
```

**Typed answer in English:**
A rolling update changes the Pod template, so Kubernetes creates a new ReplicaSet and gradually replaces old Pods with new ones.


---

## LO4.5 — View a Deployment's rollout history and roll back to the previous revi...

**Full question:** View a Deployment's rollout history and roll back to the previous revision. Explain what the CHANGE-CAUSE column shows and how to populate it.

**What the task wants:** Show Deployment revisions, annotate the change-cause, and roll back.

**Commands and what they do:**

```bash
# Adds text that appears in the CHANGE-CAUSE column.
kubectl annotate deployment/web kubernetes.io/change-cause="update nginx version" --overwrite
# Shows recorded rollout revisions.
kubectl rollout history deployment/web
# Rolls back to the previous revision.
kubectl rollout undo deployment/web
kubectl rollout status deployment/web
```

**Command-by-command explanation:**
- `kubectl annotate deployment/web kubernetes.io/change-cause="update nginx version" --overwrite` — writes a change-cause annotation so rollout history has a meaningful description.
- `kubectl rollout history deployment/web` — shows Deployment revision history, useful for rollback and change tracking.
- `kubectl rollout undo deployment/web` — rolls the Deployment back to a previous working revision.
- `kubectl rollout status deployment/web` — waits for the rollout to finish or shows that the rollout is stuck.

**Verification / proof that it worked:**

```bash
kubectl rollout history deployment/web
kubectl get rs -l app=web
```

**Typed answer in English:**
CHANGE-CAUSE shows the kubernetes.io/change-cause annotation for a revision. Rollback restores the previous Deployment Pod template.


---

## LO4.6 — Set the update strategy to RollingUpdate with maxSurge: 1 and maxUnava...

**Full question:** Set the update strategy to RollingUpdate with maxSurge: 1 and maxUnavailable: 0, and explain the effect on availability during an update.

**What the task wants:** Patch the Deployment strategy so one extra Pod is allowed and zero desired Pods may be unavailable during update.

**Commands and what they do:**

```bash
# Sets RollingUpdate strategy and availability limits.
kubectl patch deployment web --type=merge -p '{"spec":{"strategy":{"type":"RollingUpdate","rollingUpdate":{"maxSurge":1,"maxUnavailable":0}}}}'
```

**Command-by-command explanation:**
- `kubectl patch deployment web --type=merge -p '{"spec":{"strategy":{"type":"RollingUpdate","rollingUpdate":{"maxSurge":1,"maxUnavailable":0}}}}'` — patches only the requested fields on an existing Kubernetes object.

**Verification / proof that it worked:**

```bash
kubectl get deploy web -o jsonpath='{.spec.strategy}'; echo
```

**Typed answer in English:**
maxUnavailable 0 keeps all desired replicas available during update. maxSurge 1 allows one extra temporary Pod above the desired count.


---

## LO4.7 — Switch a Deployment's strategy to Recreate and describe a concrete sce...

**Full question:** Switch a Deployment's strategy to Recreate and describe a concrete scenario where this is required instead of RollingUpdate.

**What the task wants:** Use Recreate when old and new versions cannot run at the same time.

**Commands and what they do:**

```bash
# Recreate cannot keep the rollingUpdate block, so this also removes it.
kubectl patch deployment web --type=merge -p '{"spec":{"strategy":{"type":"Recreate","rollingUpdate":null}}}'
```

**Command-by-command explanation:**
- `kubectl patch deployment web --type=merge -p '{"spec":{"strategy":{"type":"Recreate","rollingUpdate":null}}}'` — patches only the requested fields on an existing Kubernetes object.

**Verification / proof that it worked:**

```bash
kubectl get deployment web -o jsonpath='{.spec.strategy.type}'; echo
```

**Typed answer in English:**
Recreate stops all old Pods before starting new ones, causing downtime. It is useful for singleton apps, incompatible database migrations, or workloads that cannot run old and new versions together.


---

## LO4.8 — Add CPU/memory requests and limits to a Deployment's container and ver...

**Full question:** Add CPU/memory requests and limits to a Deployment's container and verify they appear in the running pod spec.

**What the task wants:** Set scheduler requests and runtime limits for the Deployment container.

**Commands and what they do:**

```bash
# Requests are used for scheduling; limits are hard caps.
kubectl set resources deployment web \
  --requests=cpu=100m,memory=128Mi \
  --limits=cpu=500m,memory=256Mi
```

**Command-by-command explanation:**
- `kubectl set resources deployment web` — sets CPU/memory requests and limits on the workload container.
- `--requests=cpu=100m,memory=128Mi` — run this command as part of the task; then use the verification commands to confirm the result.
- `--limits=cpu=500m,memory=256Mi` — run this command as part of the task; then use the verification commands to confirm the result.

**Verification / proof that it worked:**

```bash
kubectl describe pod -l app=web | grep -A8 -i 'Limits\|Requests'
kubectl get pod -l app=web -o jsonpath='{.items[0].spec.containers[0].resources}'; echo
```

**Typed answer in English:**
Requests reserve resources for scheduling. Limits cap resource usage; memory above the limit can cause OOMKilled.


---

## LO4.9 — Set revisionHistoryLimit: 3 and explain how it affects your ability to...

**Full question:** Set revisionHistoryLimit: 3 and explain how it affects your ability to roll back and how many old ReplicaSets are kept.

**What the task wants:** Keep only three old ReplicaSet revisions for rollback.

**Commands and what they do:**

```bash
kubectl patch deployment web --type=merge -p '{"spec":{"revisionHistoryLimit":3}}'
```

**Command-by-command explanation:**
- `kubectl patch deployment web --type=merge -p '{"spec":{"revisionHistoryLimit":3}}'` — patches only the requested fields on an existing Kubernetes object.

**Verification / proof that it worked:**

```bash
kubectl get deploy web -o jsonpath='{.spec.revisionHistoryLimit}'; echo
kubectl get rs -l app=web
```

**Typed answer in English:**
revisionHistoryLimit 3 means Kubernetes keeps up to three old ReplicaSets for rollback. Older revisions are garbage-collected.


---

## LO4.10 — Set a Deployment's image to a non-existent tag, observe the rollout ge...

**Full question:** Set a Deployment's image to a non-existent tag, observe the rollout get stuck, and explain why the old pods keep serving traffic.

**What the task wants:** Trigger a failed rollout and show that old Pods continue serving because new Pods are not Ready.

**Commands and what they do:**

```bash
CONTAINER=$(kubectl get deploy web -o jsonpath='{.spec.template.spec.containers[0].name}')
# This image tag does not exist, so new Pods should fail with ImagePullBackOff.
kubectl set image deployment/web ${CONTAINER}=nginx:doesnotexist
kubectl rollout status deployment/web
```

**Command-by-command explanation:**
- `CONTAINER=$(kubectl get deploy web -o jsonpath='{.spec.template.spec.containers[0].name}')` — stores the real container name in a shell variable; this avoids guessing whether the container is called `web`, `nginx`, etc.
- `kubectl set image deployment/web ${CONTAINER}=nginx:doesnotexist` — changes the image in the Deployment Pod template, which starts a new rollout.
- `kubectl rollout status deployment/web` — waits for the rollout to finish or shows that the rollout is stuck.

**Verification / proof that it worked:**

```bash
kubectl get pods -l app=web
kubectl get rs -l app=web
kubectl describe pod <new-bad-pod>
# Fix after proof:
kubectl rollout undo deployment/web
```

**Typed answer in English:**
The rollout gets stuck because new Pods cannot pull the image. With RollingUpdate, old Ready Pods keep running until new Pods become Ready.


---

## LO4.11 — Use a label selector to list only the pods belonging to one Deployment...

**Full question:** Use a label selector to list only the pods belonging to one Deployment, and explain the Deployment → ReplicaSet → Pod selector relationship.

**What the task wants:** Use the Deployment labels to select its Pods.

**Commands and what they do:**

```bash
# Shows the selector that the Deployment uses.
kubectl get deploy web -o jsonpath='{.spec.selector.matchLabels}'; echo
# Lists Pods with the matching label.
kubectl get pods -l app=web -o wide
kubectl get rs -l app=web
```

**Command-by-command explanation:**
- `kubectl get deploy web -o jsonpath='{.spec.selector.matchLabels}'; echo` — run this command as part of the task; then use the verification commands to confirm the result.
- `kubectl get pods -l app=web -o wide` — lists Pods and shows whether they are Running, Ready, Pending, failing, or recreated.
- `kubectl get rs -l app=web` — shows ReplicaSets created by the Deployment and their desired/current/ready Pod counts.

**Verification / proof that it worked:**

```bash
kubectl get pods --show-labels
kubectl describe rs <replicaset-name> | grep -i selector
```

**Typed answer in English:**
The Deployment selector matches ReplicaSets, and each ReplicaSet selector matches Pods. The Pod template labels must match the selector.


---

## LO4.12 — Expose a Deployment with kubectl expose, then explain what object was ...

**Full question:** Expose a Deployment with kubectl expose, then explain what object was created and how its selector was derived.

**What the task wants:** Create a Service in front of the Deployment.

**Commands and what they do:**

```bash
# Delete old Service only if it exists, so expose does not fail.
kubectl delete svc web --ignore-not-found
# Creates a ClusterIP Service named web that forwards port 80 to targetPort 80.
kubectl expose deployment web --port=80 --target-port=80
```

**Command-by-command explanation:**
- `kubectl delete svc web --ignore-not-found` — removes the old object if it exists, so the next create/apply command does not fail because of a name conflict.
- `kubectl expose deployment web --port=80 --target-port=80` — creates a Service in front of Pods selected from the Deployment labels.

**Verification / proof that it worked:**

```bash
kubectl get svc web
kubectl get svc web -o yaml
kubectl get endpoints web
```

**Typed answer in English:**
kubectl expose creates a Service. Its selector is derived from the Deployment Pod labels, so it sends traffic to those Pods.


---

## LO4.13 — Add a sidecar (second) container to a Deployment's pod template and ex...

**Full question:** Add a sidecar (second) container to a Deployment's pod template and explain how the two containers share the pod network and volumes.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl apply -f yaml/web-sidecar.yaml
```

**Command-by-command explanation:**
- `kubectl apply -f yaml/web-sidecar.yaml` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.

**Verification / proof that it worked:**

```bash
POD=$(kubectl get pod -l app=web-sidecar -o jsonpath='{.items[0].metadata.name}')
kubectl exec $POD -c sidecar -- cat /data/index.html
kubectl exec $POD -c nginx -- sh -c 'wget -qO- localhost:80'
```

**Typed answer in English:**
The YAML creates one Pod template with nginx and a busybox sidecar. Both containers share the same Pod network namespace and the same emptyDir volume.


---

## LO4.14 — Write a Deployment manifest for httpd:2.4 with 2 replicas, a named con...

**Full question:** Write a Deployment manifest for httpd:2.4 with 2 replicas, a named container port, and a nodeSelector; explain why it stays Pending if no node matches.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl apply -f yaml/httpd.yaml
```

**Command-by-command explanation:**
- `kubectl apply -f yaml/httpd.yaml` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.

**Verification / proof that it worked:**

```bash
kubectl get pods -l app=httpd
kubectl describe pod -l app=httpd
kubectl get nodes --show-labels
```

**Typed answer in English:**
The nodeSelector requires a node label such as disktype=ssd. If no node has that label, the scheduler cannot place the Pods and they stay Pending.


---

## LO4.15 — Create a bare Pod (no controller) running busybox that sleeps, then ex...

**Full question:** Create a bare Pod (no controller) running busybox that sleeps, then explain what happens when you delete it versus a Deployment-managed pod.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl run busy --image=busybox --restart=Never -- sleep 3600
kubectl delete pod busy
POD=$(kubectl get pod -l app=web -o jsonpath='{.items[0].metadata.name}')
kubectl delete pod $POD
```

**Command-by-command explanation:**
- `kubectl run busy --image=busybox --restart=Never -- sleep 3600` — run this command as part of the task; then use the verification commands to confirm the result.
- `kubectl delete pod busy` — removes the old object if it exists, so the next create/apply command does not fail because of a name conflict.
- `POD=$(kubectl get pod -l app=web -o jsonpath='{.items[0].metadata.name}')` — run this command as part of the task; then use the verification commands to confirm the result.
- `kubectl delete pod $POD` — removes the old object if it exists, so the next create/apply command does not fail because of a name conflict.

**Verification / proof that it worked:**

```bash
kubectl get pod busy
kubectl get pods -l app=web
```

**Typed answer in English:**
A bare Pod is not recreated after delete. A Deployment-managed Pod is recreated by its ReplicaSet to maintain the desired replica count.


---

## LO4.16 — Write a StatefulSet for redis:7 with 3 replicas plus a headless Servic...

**Full question:** Write a StatefulSet for redis:7 with 3 replicas plus a headless Service, and show the stable pod names and per-pod DNS records.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl apply -f yaml/redis.yaml
```

**Command-by-command explanation:**
- `kubectl apply -f yaml/redis.yaml` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.

**Verification / proof that it worked:**

```bash
kubectl get pods -l app=redis
kubectl get svc redis
kubectl run tmp --rm -it --image=busybox -- nslookup redis-0.redis.default.svc.cluster.local
```

**Typed answer in English:**
StatefulSet gives stable names such as redis-0, redis-1, redis-2. The headless Service with clusterIP None gives per-Pod DNS records.


---

## LO4.17 — Create a DaemonSet running a busybox agent and explain why exactly one...

**Full question:** Create a DaemonSet running a busybox agent and explain why exactly one pod runs per node.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl apply -f yaml/agent.yaml
```

**Command-by-command explanation:**
- `kubectl apply -f yaml/agent.yaml` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.

**Verification / proof that it worked:**

```bash
kubectl get ds agent
kubectl get pods -l app=agent -o wide
kubectl get nodes
```

**Typed answer in English:**
A DaemonSet runs one Pod on every eligible node. On single-node Minikube, one Pod is expected.


---

## LO4.18 — Create a Job using busybox/perl that computes something once and compl...

**Full question:** Create a Job using busybox/perl that computes something once and completes; show COMPLETIONS and read the result from logs.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl create job calc --image=busybox -- sh -c 'echo $((6*7))'
```

**Command-by-command explanation:**
- `kubectl create job calc --image=busybox -- sh -c 'echo $((6*7))'` — run this command as part of the task; then use the verification commands to confirm the result.

**Verification / proof that it worked:**

```bash
kubectl wait --for=condition=complete job/calc --timeout=120s
kubectl get jobs
kubectl logs job/calc
```

**Typed answer in English:**
A Job runs a task to completion. COMPLETIONS shows succeeded/desired, and logs show the result.


---

## LO4.19 — Create a CronJob that prints the date every minute; show how to suspen...

**Full question:** Create a CronJob that prints the date every minute; show how to suspend it and how to list the Jobs it spawns.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl create cronjob date --image=busybox --schedule='* * * * *' -- date
kubectl patch cronjob date -p '{"spec":{"suspend":true}}'
```

**Command-by-command explanation:**
- `kubectl create cronjob date --image=busybox --schedule='* * * * *' -- date` — run this command as part of the task; then use the verification commands to confirm the result.
- `kubectl patch cronjob date -p '{"spec":{"suspend":true}}'` — patches only the requested fields on an existing Kubernetes object.

**Verification / proof that it worked:**

```bash
kubectl get cronjob date
kubectl get jobs
kubectl get cronjob date -o jsonpath='{.spec.suspend}'; echo
```

**Typed answer in English:**
A CronJob creates Jobs on a schedule. suspend true stops new Jobs from being created.


---

## LO4.20 — Manually run a Job from an existing CronJob (kubectl create job --from...

**Full question:** Manually run a Job from an existing CronJob (kubectl create job --from=cronjob/...) to test it on demand.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl create job manual-date --from=cronjob/date
```

**Command-by-command explanation:**
- `kubectl create job manual-date --from=cronjob/date` — run this command as part of the task; then use the verification commands to confirm the result.

**Verification / proof that it worked:**

```bash
kubectl wait --for=condition=complete job/manual-date --timeout=120s
kubectl logs job/manual-date
```

**Typed answer in English:**
This creates one Job from the CronJob template immediately, without waiting for the schedule.


---

## LO4.21 — Show that a StatefulSet creates and terminates pods in order, and cont...

**Full question:** Show that a StatefulSet creates and terminates pods in order, and contrast this with a Deployment.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl scale statefulset redis --replicas=3
kubectl get pods -l app=redis -w
kubectl scale statefulset redis --replicas=1
```

**Command-by-command explanation:**
- `kubectl scale statefulset redis --replicas=3` — changes the desired replica count for the controller.
- `kubectl get pods -l app=redis -w` — lists Pods and shows whether they are Running, Ready, Pending, failing, or recreated.
- `kubectl scale statefulset redis --replicas=1` — changes the desired replica count for the controller.

**Verification / proof that it worked:**

```bash
kubectl get pods -l app=redis -w
```

**Typed answer in English:**
By default, StatefulSet creates Pods in ordinal order and deletes them in reverse order. Deployment Pods have random names and no stable order.


---

## LO4.22 — Create a multi-container Pod where two containers share an emptyDir — ...

**Full question:** Create a multi-container Pod where two containers share an emptyDir — one writes a file, the other reads it. Prove it's shared.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl apply -f yaml/shared.yaml
```

**Command-by-command explanation:**
- `kubectl apply -f yaml/shared.yaml` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.

**Verification / proof that it worked:**

```bash
kubectl exec shared-demo -c reader -- cat /data/msg
```

**Typed answer in English:**
The writer and reader containers mount the same emptyDir. The reader can see the file written by the writer.


---

## LO4.23 — List the StorageClasses, PVs and PVCs.

**Full question:** List the StorageClasses, PVs and PVCs.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl get storageclass
kubectl get pv
kubectl get pvc -A
```

**Command-by-command explanation:**
- `kubectl get storageclass` — lists storage classes that define how dynamic storage is provisioned.
- `kubectl get pv` — lists PersistentVolumes available or bound in the cluster.
- `kubectl get pvc -A` — lists PersistentVolumeClaims and whether they are bound to storage.

**Verification / proof that it worked:**

```bash
kubectl get sc
kubectl get pv
kubectl get pvc -A
```

**Typed answer in English:**
StorageClasses define storage types, PVs are actual volumes, and PVCs are claims made by workloads.


---

## LO4.24 — Mount a ConfigMap as a volume so each key becomes a file, and verify t...

**Full question:** Mount a ConfigMap as a volume so each key becomes a file, and verify the contents inside the pod.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl create configmap appcfg --from-literal=color=blue --from-literal=mode=dark --dry-run=client -o yaml | kubectl apply -f -
kubectl apply -f yaml/cm-vol.yaml
```

**Command-by-command explanation:**
- `kubectl create configmap appcfg --from-literal=color=blue --from-literal=mode=dark --dry-run=client -o yaml | kubectl apply -f -` — creates or updates a ConfigMap containing non-secret configuration values.
- `kubectl apply -f yaml/cm-vol.yaml` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.

**Verification / proof that it worked:**

```bash
kubectl exec cm-demo -- ls /etc/cfg
kubectl exec cm-demo -- cat /etc/cfg/color
```

**Typed answer in English:**
Each ConfigMap key becomes a file in the mounted directory. The file content is the key value.


---

## LO4.25 — Mount a single ConfigMap key to a specific path using subPath; explain...

**Full question:** Mount a single ConfigMap key to a specific path using subPath; explain when it's needed and its update caveat.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl create configmap appcfg --from-literal=color=blue --from-literal=mode=dark --dry-run=client -o yaml | kubectl apply -f -
kubectl apply -f yaml/cm-subpath.yaml
```

**Command-by-command explanation:**
- `kubectl create configmap appcfg --from-literal=color=blue --from-literal=mode=dark --dry-run=client -o yaml | kubectl apply -f -` — creates or updates a ConfigMap containing non-secret configuration values.
- `kubectl apply -f yaml/cm-subpath.yaml` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.

**Verification / proof that it worked:**

```bash
kubectl exec cm-subpath -- cat /etc/app/color.conf
```

**Typed answer in English:**
subPath mounts one key as one file without hiding the whole directory. The caveat is that subPath mounts do not receive live ConfigMap updates.


---

## LO4.26 — Mount a Secret as a volume and verify the files contain decoded values...

**Full question:** Mount a Secret as a volume and verify the files contain decoded values with restrictive permissions.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl create secret generic mysecret --from-literal=username=admin --from-literal=password=s3cret --dry-run=client -o yaml | kubectl apply -f -
kubectl apply -f yaml/secret-vol.yaml
```

**Command-by-command explanation:**
- `kubectl create secret generic mysecret --from-literal=username=admin --from-literal=password=s3cret --dry-run=client -o yaml | kubectl apply -f -` — creates or updates a generic Secret with literal values or files.
- `kubectl apply -f yaml/secret-vol.yaml` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.

**Verification / proof that it worked:**

```bash
kubectl exec secret-demo -- ls -l /etc/sec
kubectl exec secret-demo -- cat /etc/sec/password
```

**Typed answer in English:**
Secret values are mounted as decoded files. defaultMode can make the files restrictive, for example read-only for owner.


---

## LO4.27 — Show that scaling a StatefulSet creates one PVC per replica, then expl...

**Full question:** Show that scaling a StatefulSet creates one PVC per replica, then explain what happens to those PVCs when the StatefulSet is deleted.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl apply -f yaml/redis-pvc.yaml
```

**Command-by-command explanation:**
- `kubectl apply -f yaml/redis-pvc.yaml` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.

**Verification / proof that it worked:**

```bash
kubectl get pvc
kubectl delete statefulset redis-pvc
kubectl get pvc
```

**Typed answer in English:**
volumeClaimTemplates creates one PVC per StatefulSet replica. PVCs are retained after StatefulSet deletion by default to protect data.


---

## LO4.28 — Use an initContainer to pre-populate data into a shared volume before ...

**Full question:** Use an initContainer to pre-populate data into a shared volume before the main container reads it.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl apply -f yaml/init.yaml
```

**Command-by-command explanation:**
- `kubectl apply -f yaml/init.yaml` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.

**Verification / proof that it worked:**

```bash
kubectl logs init-demo -c app
kubectl exec init-demo -c app -- cat /data/file
```

**Typed answer in English:**
The initContainer runs first and writes data into the shared volume. The main container starts after it completes and can read the data.


---

## LO4.29 — Set readOnly: true on a volume mount and prove writes are rejected.

**Full question:** Set readOnly: true on a volume mount and prove writes are rejected.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl apply -f yaml/readonly.yaml
```

**Command-by-command explanation:**
- `kubectl apply -f yaml/readonly.yaml` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.

**Verification / proof that it worked:**

```bash
kubectl exec readonly-demo -- sh -c 'echo x > /data/file'
```

**Typed answer in English:**
readOnly true mounts the volume as read-only inside the container, so writing to that path fails.


---

## LO4.30 — Set a sizeLimit on an emptyDir and explain what happens if the contain...

**Full question:** Set a sizeLimit on an emptyDir and explain what happens if the container exceeds it.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl apply -f yaml/sizelimit.yaml
```

**Command-by-command explanation:**
- `kubectl apply -f yaml/sizelimit.yaml` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.

**Verification / proof that it worked:**

```bash
kubectl describe pod sizelimit-demo
# Optional risky proof:
# kubectl exec sizelimit-demo -- sh -c 'dd if=/dev/zero of=/scratch/big bs=1M count=200'
```

**Typed answer in English:**
emptyDir sizeLimit limits temporary Pod storage. If usage exceeds the limit, the kubelet can evict the Pod.


---

## LO4.31 — Create a generic Secret from literals for a DB username/password, then...

**Full question:** Create a generic Secret from literals for a DB username/password, then show the stored values are base64-encoded, not encrypted.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl create secret generic dbcreds --from-literal=username=admin --from-literal=password=s3cret --dry-run=client -o yaml | kubectl apply -f -
```

**Command-by-command explanation:**
- `kubectl create secret generic dbcreds --from-literal=username=admin --from-literal=password=s3cret --dry-run=client -o yaml | kubectl apply -f -` — creates or updates a generic Secret with literal values or files.

**Verification / proof that it worked:**

```bash
kubectl get secret dbcreds -o yaml
kubectl get secret dbcreds -o jsonpath='{.data.password}' | base64 -d; echo
```

**Typed answer in English:**
Kubernetes Secret data is base64 encoded in YAML. Base64 is encoding, not encryption.


---

## LO4.32 — Create a Secret from files (--from-file) holding a certificate and key...

**Full question:** Create a Secret from files (--from-file) holding a certificate and key, and identify the resulting keys.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
printf 'cert-bytes' > cert.pem
printf 'key-bytes' > key.pem
kubectl create secret generic tlsfiles --from-file=tls.crt=cert.pem --from-file=tls.key=key.pem --dry-run=client -o yaml | kubectl apply -f -
```

**Command-by-command explanation:**
- `printf 'cert-bytes' > cert.pem` — prepares a local file or test input needed for the task.
- `printf 'key-bytes' > key.pem` — prepares a local file or test input needed for the task.
- `kubectl create secret generic tlsfiles --from-file=tls.crt=cert.pem --from-file=tls.key=key.pem --dry-run=client -o yaml | kubectl apply -f -` — creates or updates a generic Secret with literal values or files.

**Verification / proof that it worked:**

```bash
kubectl get secret tlsfiles -o jsonpath='{.data}'; echo
```

**Typed answer in English:**
--from-file creates Secret keys from filenames or from the explicit key=path names. Here the keys are tls.crt and tls.key.


---

## LO4.33 — Create a kubernetes.io/tls typed Secret from a cert/key pair and expla...

**Full question:** Create a kubernetes.io/tls typed Secret from a cert/key pair and explain where it's consumed.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl create secret tls mytls --cert=cert.pem --key=key.pem --dry-run=client -o yaml | kubectl apply -f -
```

**Command-by-command explanation:**
- `kubectl create secret tls mytls --cert=cert.pem --key=key.pem --dry-run=client -o yaml | kubectl apply -f -` — creates a TLS Secret with certificate and key data.

**Verification / proof that it worked:**

```bash
kubectl get secret mytls -o jsonpath='{.type}'; echo
kubectl get secret mytls -o jsonpath='{.data}'; echo
```

**Typed answer in English:**
A kubernetes.io/tls Secret contains tls.crt and tls.key. It is commonly consumed by Ingress or workloads that need TLS certificates.


---

## LO4.34 — Mount a Secret as a volume and explain the security trade-offs versus ...

**Full question:** Mount a Secret as a volume and explain the security trade-offs versus env vars.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl apply -f yaml/secret-vol.yaml
```

**Command-by-command explanation:**
- `kubectl apply -f yaml/secret-vol.yaml` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.

**Verification / proof that it worked:**

```bash
kubectl exec secret-demo -- cat /etc/sec/password
```

**Typed answer in English:**
Secret volumes can have file permissions and can update automatically. Environment variables are simpler but can leak through process environment and require restart for updates.


---

## LO4.35 — Use envFrom with a secretRef to load every key of a Secret as env vars...

**Full question:** Use envFrom with a secretRef to load every key of a Secret as env vars at once.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl create secret generic dbcreds --from-literal=username=admin --from-literal=password=s3cret --dry-run=client -o yaml | kubectl apply -f -
kubectl apply -f yaml/envfrom.yaml
```

**Command-by-command explanation:**
- `kubectl create secret generic dbcreds --from-literal=username=admin --from-literal=password=s3cret --dry-run=client -o yaml | kubectl apply -f -` — creates or updates a generic Secret with literal values or files.
- `kubectl apply -f yaml/envfrom.yaml` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.

**Verification / proof that it worked:**

```bash
kubectl exec envfrom-demo -- env | grep -E 'username|password'
```

**Typed answer in English:**
envFrom.secretRef loads all Secret keys as environment variables. The environment variable names come from the Secret key names.


---

## LO4.36 — Create a docker-registry (image-pull) Secret and reference it via imag...

**Full question:** Create a docker-registry (image-pull) Secret and reference it via imagePullSecrets; explain when it's required.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl create secret docker-registry regcred --docker-server=docker.io --docker-username=USER --docker-password=PASS --dry-run=client -o yaml | kubectl apply -f -
```

**Command-by-command explanation:**
- `kubectl create secret docker-registry regcred --docker-server=docker.io --docker-username=USER --docker-password=PASS --dry-run=client -o yaml | kubectl apply -f -` — creates an image-pull Secret for authenticated registry access.

**Verification / proof that it worked:**

```bash
kubectl get secret regcred -o jsonpath='{.type}'; echo
```

**Typed answer in English:**
imagePullSecrets are required for private registries or authenticated pulls. Without them, Pods may fail with ImagePullBackOff.


---

## LO4.37 — Create a Secret with several keys and selectively mount only one of th...

**Full question:** Create a Secret with several keys and selectively mount only one of them into a pod.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl create secret generic multi --from-literal=a=1 --from-literal=b=2 --from-literal=c=3 --dry-run=client -o yaml | kubectl apply -f -
kubectl apply -f yaml/secret-one.yaml
```

**Command-by-command explanation:**
- `kubectl create secret generic multi --from-literal=a=1 --from-literal=b=2 --from-literal=c=3 --dry-run=client -o yaml | kubectl apply -f -` — creates or updates a generic Secret with literal values or files.
- `kubectl apply -f yaml/secret-one.yaml` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.

**Verification / proof that it worked:**

```bash
kubectl exec secret-one -- ls /etc/sec
kubectl exec secret-one -- cat /etc/sec/only-b.txt
```

**Typed answer in English:**
The items field selects only specific Secret keys to project into the volume. Other keys are not mounted.


---

## LO4.38 — Create a ClusterIP Service for a Deployment and resolve it by DNS from...

**Full question:** Create a ClusterIP Service for a Deployment and resolve it by DNS from another pod (nslookup <svc>.<ns>.svc.cluster.local).

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl delete svc web --ignore-not-found
kubectl expose deployment web --port=80 --target-port=80
```

**Command-by-command explanation:**
- `kubectl delete svc web --ignore-not-found` — removes the old object if it exists, so the next create/apply command does not fail because of a name conflict.
- `kubectl expose deployment web --port=80 --target-port=80` — creates a Service in front of Pods selected from the Deployment labels.

**Verification / proof that it worked:**

```bash
kubectl get svc web
kubectl run tmp --rm -it --image=busybox -- nslookup web.default.svc.cluster.local
```

**Typed answer in English:**
ClusterIP gives a stable internal Service IP. CoreDNS resolves the Service FQDN to that Service.


---

## LO4.39 — Create a NodePort Service and reach it via minikube service <svc> --ur...

**Full question:** Create a NodePort Service and reach it via minikube service <svc> --url and $(minikube ip):<nodePort>.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl delete svc web --ignore-not-found
kubectl expose deployment web --type=NodePort --port=80 --target-port=80
```

**Command-by-command explanation:**
- `kubectl delete svc web --ignore-not-found` — removes the old object if it exists, so the next create/apply command does not fail because of a name conflict.
- `kubectl expose deployment web --type=NodePort --port=80 --target-port=80` — creates a Service in front of Pods selected from the Deployment labels.

**Verification / proof that it worked:**

```bash
kubectl get svc web
minikube service web --url
curl $(minikube ip):$(kubectl get svc web -o jsonpath='{.spec.ports[0].nodePort}')
```

**Typed answer in English:**
NodePort opens a port on every node and forwards traffic to the Service.


---

## LO4.40 — Create a LoadBalancer Service, explain why EXTERNAL-IP is <pending> on...

**Full question:** Create a LoadBalancer Service, explain why EXTERNAL-IP is <pending> on minikube, and obtain access with minikube tunnel.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl delete svc web --ignore-not-found
kubectl expose deployment web --type=LoadBalancer --port=80 --target-port=80
```

**Command-by-command explanation:**
- `kubectl delete svc web --ignore-not-found` — removes the old object if it exists, so the next create/apply command does not fail because of a name conflict.
- `kubectl expose deployment web --type=LoadBalancer --port=80 --target-port=80` — creates a Service in front of Pods selected from the Deployment labels.

**Verification / proof that it worked:**

```bash
kubectl get svc web
# in another terminal: minikube tunnel
kubectl get svc web
```

**Typed answer in English:**
Minikube has no real cloud load balancer, so EXTERNAL-IP may stay pending until minikube tunnel provides access.


---

## LO4.41 — Create a headless Service (clusterIP: None) for a StatefulSet and show...

**Full question:** Create a headless Service (clusterIP: None) for a StatefulSet and show it returns per-pod A records instead of a single VIP.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl apply -f yaml/redis.yaml
```

**Command-by-command explanation:**
- `kubectl apply -f yaml/redis.yaml` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.

**Verification / proof that it worked:**

```bash
kubectl get svc redis
kubectl run tmp --rm -it --image=busybox -- nslookup redis.default.svc.cluster.local
```

**Typed answer in English:**
A headless Service has no ClusterIP. DNS returns Pod records instead of one virtual Service IP.


---

## LO4.42 — Inspect a Service's Endpoints/EndpointSlice and explain how they're po...

**Full question:** Inspect a Service's Endpoints/EndpointSlice and explain how they're populated from the pod selector.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl get endpoints web
kubectl get endpointslices -l kubernetes.io/service-name=web
```

**Command-by-command explanation:**
- `kubectl get endpoints web` — shows which Pod IPs the Service is actually routing to; empty endpoints usually mean label/readiness problems.
- `kubectl get endpointslices -l kubernetes.io/service-name=web` — shows which Pod IPs the Service is actually routing to; empty endpoints usually mean label/readiness problems.

**Verification / proof that it worked:**

```bash
kubectl describe endpoints web
kubectl get pods --show-labels
kubectl get svc web -o yaml
```

**Typed answer in English:**
Endpoints are populated from Ready Pods matching the Service selector. Empty endpoints usually mean label mismatch or not Ready Pods.


---

## LO4.43 — Demonstrate cross-namespace access using the FQDN, and show the short ...

**Full question:** Demonstrate cross-namespace access using the FQDN, and show the short name fails from another namespace.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl create namespace other --dry-run=client -o yaml | kubectl apply -f -
```

**Command-by-command explanation:**
- `kubectl create namespace other --dry-run=client -o yaml | kubectl apply -f -` — run this command as part of the task; then use the verification commands to confirm the result.

**Verification / proof that it worked:**

```bash
kubectl run tmp -n other --rm -it --image=busybox -- wget -qO- web.default.svc.cluster.local
kubectl run tmp -n other --rm -it --image=busybox -- nslookup web
```

**Typed answer in English:**
Short names work only in the same namespace. From another namespace, use service.namespace.svc.cluster.local.


---

## LO4.44 — Compare kubectl port-forward to a Service versus to a Pod — explain th...

**Full question:** Compare kubectl port-forward to a Service versus to a Pod — explain the difference and when each fits.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl port-forward svc/web 8080:80
# or for one specific Pod:
POD=$(kubectl get pod -l app=web -o jsonpath='{.items[0].metadata.name}')
kubectl port-forward pod/$POD 8080:80
```

**Command-by-command explanation:**
- `kubectl port-forward svc/web 8080:80` — opens a local tunnel from your machine to a Service or Pod for testing.
- `POD=$(kubectl get pod -l app=web -o jsonpath='{.items[0].metadata.name}')` — run this command as part of the task; then use the verification commands to confirm the result.
- `kubectl port-forward pod/$POD 8080:80` — opens a local tunnel from your machine to a Service or Pod for testing.

**Verification / proof that it worked:**

```bash
curl localhost:8080
```

**Typed answer in English:**
Port-forward to a Service uses the Service port and chooses a backend Pod. Port-forward to a Pod targets one specific Pod for debugging.


---

## LO4.45 — Explain the difference between a Service's port, targetPort, and nodeP...

**Full question:** Explain the difference between a Service's port, targetPort, and nodePort.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl get svc web -o yaml
```

**Command-by-command explanation:**
- `kubectl get svc web -o yaml` — lists or inspects Services, including type, cluster IP, ports and selectors.

**Verification / proof that it worked:**

```bash
kubectl get svc web -o yaml | grep -A10 ports
```

**Typed answer in English:**
port is the Service port, targetPort is the container port, and nodePort is the external node port for NodePort/LoadBalancer Services.


---

## LO4.46 — Verify connectivity to a ClusterIP Service from a throwaway debug pod ...

**Full question:** Verify connectivity to a ClusterIP Service from a throwaway debug pod (kubectl run tmp --rm -it --image=busybox -- sh) with wget/nc.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl run tmp --rm -it --image=busybox -- sh
```

**Command-by-command explanation:**
- `kubectl run tmp --rm -it --image=busybox -- sh` — starts a temporary BusyBox debug Pod and removes it after exit; useful for DNS and Service tests.

**Verification / proof that it worked:**

```bash
nslookup web
wget -qO- web:80
nc -zv web 80
```

**Typed answer in English:**
A temporary Pod tests connectivity from inside the cluster network, which is the correct place to test ClusterIP Services.


---

## LO4.47 — Create two Deployments and one Service that load-balances across both ...

**Full question:** Create two Deployments and one Service that load-balances across both by sharing a common label; prove requests hit both.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl apply -f yaml/shop.yaml
```

**Command-by-command explanation:**
- `kubectl apply -f yaml/shop.yaml` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.

**Verification / proof that it worked:**

```bash
kubectl get endpoints shop
kubectl run tmp --rm -it --image=busybox -- sh -c 'for i in $(seq 1 8); do wget -qO- shop; echo; done'
```

**Typed answer in English:**
The Service selector app=shop matches Pods from both Deployments, so endpoints include both versions and requests can hit both.


---

## LO4.48 — Systematically diagnose why a pod can't reach a Service: DNS → endpoin...

**Full question:** Systematically diagnose why a pod can't reach a Service: DNS → endpoints → selector labels → readiness → port.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl run tmp --rm -it --image=busybox -- nslookup web
kubectl get endpoints web
kubectl get svc web -o yaml
kubectl get pods --show-labels
```

**Command-by-command explanation:**
- `kubectl run tmp --rm -it --image=busybox -- nslookup web` — starts a temporary BusyBox debug Pod and removes it after exit; useful for DNS and Service tests.
- `kubectl get endpoints web` — shows which Pod IPs the Service is actually routing to; empty endpoints usually mean label/readiness problems.
- `kubectl get svc web -o yaml` — lists or inspects Services, including type, cluster IP, ports and selectors.
- `kubectl get pods --show-labels` — lists Pods and shows whether they are Running, Ready, Pending, failing, or recreated.

**Verification / proof that it worked:**

```bash
kubectl get pods -l app=web
kubectl describe pod <pod>
kubectl get svc web -o jsonpath='{.spec.ports}'; echo
```

**Typed answer in English:**
Check the path in order: DNS, endpoints, selector labels, Pod readiness, and correct targetPort.


---

## LO4.49 — Expose a service with a NodePort and verify it works.

**Full question:** Expose a service with a NodePort and verify it works.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl delete svc web-np --ignore-not-found
kubectl expose deployment web --name=web-np --type=NodePort --port=80 --target-port=80
```

**Command-by-command explanation:**
- `kubectl delete svc web-np --ignore-not-found` — removes the old object if it exists, so the next create/apply command does not fail because of a name conflict.
- `kubectl expose deployment web --name=web-np --type=NodePort --port=80 --target-port=80` — creates a Service in front of Pods selected from the Deployment labels.

**Verification / proof that it worked:**

```bash
kubectl get svc web-np
minikube service web-np --url
```

**Typed answer in English:**
NodePort exposes the Service on nodeIP:nodePort. A successful curl/minikube service proves it works.


---

## LO4.50 — Add an HTTP livenessProbe hitting / on port 80 to an nginx Deployment ...

**Full question:** Add an HTTP livenessProbe hitting / on port 80 to an nginx Deployment and verify via kubectl describe pod.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl apply -f yaml/web-probes.yaml
```

**Command-by-command explanation:**
- `kubectl apply -f yaml/web-probes.yaml` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.

**Verification / proof that it worked:**

```bash
kubectl describe pod -l app=web-probes | grep -A3 Liveness
```

**Typed answer in English:**
The liveness probe checks whether the container is still healthy. If it fails repeatedly, kubelet restarts the container.


---

## LO4.51 — Add a tcpSocket readiness probe to a redis container on port 6379 and ...

**Full question:** Add a tcpSocket readiness probe to a redis container on port 6379 and verify it.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl apply -f yaml/redis-readiness.yaml
```

**Command-by-command explanation:**
- `kubectl apply -f yaml/redis-readiness.yaml` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.

**Verification / proof that it worked:**

```bash
kubectl describe pod -l app=redis-readiness | grep -A3 Readiness
kubectl get pods -l app=redis-readiness
```

**Typed answer in English:**
A readiness probe controls whether the Pod is added to Service endpoints. It does not restart the container.


---

## LO4.52 — Configure a startup probe with a failureThreshold * periodSeconds budg...

**Full question:** Configure a startup probe with a failureThreshold * periodSeconds budget for a ~2-minute boot and justify the numbers.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
kubectl apply -f yaml/web-probes.yaml
```

**Command-by-command explanation:**
- `kubectl apply -f yaml/web-probes.yaml` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.

**Verification / proof that it worked:**

```bash
kubectl describe pod -l app=web-probes | grep -A5 Startup
```

**Typed answer in English:**
The startup probe gives slow applications time to start. For example failureThreshold 30 times periodSeconds 5 gives 150 seconds, which is more than 2 minutes.


---

## LO4.53 — Explain the default behavior when no probes are defined at all.

**Full question:** Explain the default behavior when no probes are defined at all.

**What the task wants:** See task and apply the provided manifest/command.

**Commands and what they do:**

```bash
# No command is required. This is an explanation question.
```

**Command-by-command explanation:**
- This section is mostly a manifest or theory snippet; apply/build it and use the verification commands to prove the result.

**Verification / proof that it worked:**

```bash
kubectl get pod <pod>
kubectl describe pod <pod>
```

**Typed answer in English:**
With no probes, Kubernetes considers the container ready after it starts. It restarts the container only if the process exits; it does not detect a hung but running app.
