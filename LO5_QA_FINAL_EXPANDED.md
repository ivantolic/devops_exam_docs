# LO5 — FINAL EXPANDED TROUBLESHOOTING Q/A WITH COMPLETE QUESTIONS

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



## How to answer LO5 if professor asks you to type explanation

Use this order in English: `The mistake is ...` → `I fix it by ...` → `I verify it with ...` → `This works because ...`.


---

## LO5.1 — podman run -d --name web -p 80:8080 docker.io/library/nginx (nginx lis...

**Full question:** podman run -d --name web -p 80:8080 docker.io/library/nginx (nginx listens on 80; the site isn't reachable on the host port you expected — what's reversed?)

**What is wrong / what I am diagnosing:**
The port mapping is reversed/wrong for nginx. The format is -p host_port:container_port. nginx listens on container port 80, not 8080.

**Fix commands / commands to run:**

```bash
podman rm -f web 2>/dev/null || true
podman run -d --name web -p 80:80 docker.io/library/nginx
```

**What the commands mean:**
- `podman rm -f web 2>/dev/null || true` — removes old containers with the same name so the fixed command can run cleanly.
- `podman run -d --name web -p 80:80 docker.io/library/nginx` — runs the corrected container with the intended image, command, network, port, volume or environment settings.

**Verification / proof that the fix worked:**

```bash
curl localhost:80
podman ps
```

**Typed answer in English:**
The fix maps host port 80 to container port 80. The web page is reachable with curl, so the port mapping is correct.


---

## LO5.2 — podman run -d --name db -e MYSQL_ROOT_PASSWORD docker.io/library/mysql...

**Full question:** podman run -d --name db -e MYSQL_ROOT_PASSWORD docker.io/library/mysql (the container exits during init — inspect podman logs.)

**What is wrong / what I am diagnosing:**
MYSQL_ROOT_PASSWORD is passed without a value, so MySQL initialization fails.

**Fix commands / commands to run:**

```bash
podman rm -f db 2>/dev/null || true
podman run -d --name db -e MYSQL_ROOT_PASSWORD=secret docker.io/library/mysql
```

**What the commands mean:**
- `podman rm -f db 2>/dev/null || true` — removes old containers with the same name so the fixed command can run cleanly.
- `podman run -d --name db -e MYSQL_ROOT_PASSWORD=secret docker.io/library/mysql` — runs the corrected container with the intended image, command, network, port, volume or environment settings.

**Verification / proof that the fix worked:**

```bash
podman logs db
podman ps -a | grep db
```

**Typed answer in English:**
The variable must have a value. After setting MYSQL_ROOT_PASSWORD=secret, MySQL can continue initialization.


---

## LO5.3 — podman run -d --name app --network host -p 8080:80 docker.io/library/n...

**Full question:** podman run -d --name app --network host -p 8080:80 docker.io/library/nginx (why is the -p flag effectively ignored here?)

**What is wrong / what I am diagnosing:**
--network host makes the container use the host network namespace, so port publishing with -p is ignored/not useful.

**Fix commands / commands to run:**

```bash
podman rm -f app 2>/dev/null || true
podman run -d --name app -p 8080:80 docker.io/library/nginx
```

**What the commands mean:**
- `podman rm -f app 2>/dev/null || true` — removes old containers with the same name so the fixed command can run cleanly.
- `podman run -d --name app -p 8080:80 docker.io/library/nginx` — runs the corrected container with the intended image, command, network, port, volume or environment settings.

**Verification / proof that the fix worked:**

```bash
curl localhost:8080
podman port app
```

**Typed answer in English:**
The fix is to remove host networking if I want port mapping. With normal networking, -p 8080:80 forwards host port 8080 to container port 80.


---

## LO5.4 — podman run -d --name c1 docker.io/library/busybox (the container goes ...

**Full question:** podman run -d --name c1 docker.io/library/busybox (the container goes straight to Exited (0) — why, and how do you keep it running?)

**What is wrong / what I am diagnosing:**
BusyBox has no long-running default process here, so PID 1 exits and the container stops.

**Fix commands / commands to run:**

```bash
podman rm -f c1 2>/dev/null || true
podman run -d --name c1 docker.io/library/busybox sleep 1d
```

**What the commands mean:**
- `podman rm -f c1 2>/dev/null || true` — removes old containers with the same name so the fixed command can run cleanly.
- `podman run -d --name c1 docker.io/library/busybox sleep 1d` — runs the corrected container with the intended image, command, network, port, volume or environment settings.

**Verification / proof that the fix worked:**

```bash
podman ps | grep c1
podman logs c1
```

**Typed answer in English:**
A container stays alive only while its main process is running. sleep 1d keeps the container running.


---

## LO5.5 — podman run --rm -d --name job docker.io/library/alpine echo hello (the...

**Full question:** podman run --rm -d --name job docker.io/library/alpine echo hello (then podman logs job fails — explain the --rm + detached gotcha.)

**What is wrong / what I am diagnosing:**
--rm deletes the container immediately after echo finishes, so logs cannot be read later.

**Fix commands / commands to run:**

```bash
podman rm -f job 2>/dev/null || true
podman run -d --name job docker.io/library/alpine echo hello
podman logs job
```

**What the commands mean:**
- `podman rm -f job 2>/dev/null || true` — removes old containers with the same name so the fixed command can run cleanly.
- `podman run -d --name job docker.io/library/alpine echo hello` — runs the corrected container with the intended image, command, network, port, volume or environment settings.
- `podman logs job` — shows container logs, which usually contain the reason why it exited or failed.

**Verification / proof that the fix worked:**

```bash
podman ps -a | grep job
podman logs job
```

**Typed answer in English:**
The fix is to avoid --rm if I need to inspect logs after the container exits, or run it foreground without -d.


---

## LO5.6 — podman run -d -p 8080:80 --memory 8m docker.io/library/mysql (the data...

**Full question:** podman run -d -p 8080:80 --memory 8m docker.io/library/mysql (the database never becomes healthy — what limit is the problem?)

**What is wrong / what I am diagnosing:**
The memory limit is too low. MySQL needs much more than 8 MB. Also MySQL listens on 3306 and needs a root password.

**Fix commands / commands to run:**

```bash
podman rm -f mysql 2>/dev/null || true
podman run -d --name mysql -p 8080:3306 --memory 1g -e MYSQL_ROOT_PASSWORD=secret docker.io/library/mysql
```

**What the commands mean:**
- `podman rm -f mysql 2>/dev/null || true` — removes old containers with the same name so the fixed command can run cleanly.
- `podman run -d --name mysql -p 8080:3306 --memory 1g -e MYSQL_ROOT_PASSWORD=secret docker.io/library/mysql` — runs the corrected container with the intended image, command, network, port, volume or environment settings.

**Verification / proof that the fix worked:**

```bash
podman ps -a | grep mysql
podman logs mysql | tail -30
```

**Typed answer in English:**
The main fix is increasing memory. I also map to the correct MySQL container port 3306 and provide the required root password.


---

## LO5.7 — podman run -d --name a alpine sleep 1d; podman run -d --name b alpine ...

**Full question:** podman run -d --name a alpine sleep 1d; podman run -d --name b alpine sleep 1d; podman exec a ping b (name resolution fails — why, on the default network, and how do you fix it?)

**What is wrong / what I am diagnosing:**
The default Podman network may not provide container-name DNS resolution. A user-defined network provides name-based discovery.

**Fix commands / commands to run:**

```bash
podman rm -f a b 2>/dev/null || true
podman network create mynet 2>/dev/null || true
podman run -d --name a --network mynet docker.io/library/alpine sleep 1d
podman run -d --name b --network mynet docker.io/library/alpine sleep 1d
```

**What the commands mean:**
- `podman rm -f a b 2>/dev/null || true` — removes old containers with the same name so the fixed command can run cleanly.
- `podman network create mynet 2>/dev/null || true` — creates a user-defined network that supports name-based discovery between containers.
- `podman run -d --name a --network mynet docker.io/library/alpine sleep 1d` — runs the corrected container with the intended image, command, network, port, volume or environment settings.
- `podman run -d --name b --network mynet docker.io/library/alpine sleep 1d` — runs the corrected container with the intended image, command, network, port, volume or environment settings.

**Verification / proof that the fix worked:**

```bash
podman exec a ping -c1 b
podman network inspect mynet
```

**Typed answer in English:**
Both containers are on the same user-defined network, so container name b resolves from container a.


---

## LO5.8 — podman run -d --name web -v ./html:/usr/share/nginx/html docker.io/lib...

**Full question:** podman run -d --name web -v ./html:/usr/share/nginx/html docker.io/library/nginx (files aren't visible, or SELinux denies access — what mount flag is missing?)

**What is wrong / what I am diagnosing:**
On SELinux systems, the bind mount needs a relabel flag such as :Z or :z.

**Fix commands / commands to run:**

```bash
podman rm -f web 2>/dev/null || true
mkdir -p html && echo hello > html/index.html
podman run -d --name web -v $(pwd)/html:/usr/share/nginx/html:Z docker.io/library/nginx
```

**What the commands mean:**
- `podman rm -f web 2>/dev/null || true` — removes old containers with the same name so the fixed command can run cleanly.
- `mkdir -p html && echo hello > html/index.html` — prepares a local file or test input needed for the task.
- `podman run -d --name web -v $(pwd)/html:/usr/share/nginx/html:Z docker.io/library/nginx` — runs the corrected container with the intended image, command, network, port, volume or environment settings.

**Verification / proof that the fix worked:**

```bash
curl localhost || true
podman exec web cat /usr/share/nginx/html/index.html
```

**Typed answer in English:**
The :Z flag relabels the host directory for this container, so SELinux allows the container to read it.


---

## LO5.9 — For each Containerfile/Dockerfile: identify the mistake, fix it, and e...

**Full question:** For each Containerfile/Dockerfile: identify the mistake, fix it, and explain what was wrong.

**What is wrong / what I am diagnosing:**
This is a general instruction. For each Dockerfile, I check missing package index update, wrong CMD form, bad layer caching, overwritten PATH, EXPOSE mismatch, image size, file ownership, and multi-stage build.

**Fix commands / commands to run:**

```bash
# No single command. Read the Dockerfile and fix the specific mistake.
```

**What the commands mean:**
- This section is mostly a manifest or theory snippet; apply/build it and use the verification commands to prove the result.

**Verification / proof that the fix worked:**

```bash
podman build -t test .
podman run --rm test
```

**Typed answer in English:**
I identify the build/runtime mistake, change the Dockerfile, rebuild the image, and run the container to verify it.


---

## LO5.10 — FROM debian:12 / RUN apt-get install -y nginx (the build fails to find...

**Full question:** FROM debian:12 / RUN apt-get install -y nginx (the build fails to find the package — what's missing before install?)

**What is wrong / what I am diagnosing:**
apt-get update is missing before apt-get install.

**Fix commands / commands to run:**

```bash
cat > Containerfile <<'EOF'
FROM debian:12
RUN apt-get update && apt-get install -y nginx
EOF
podman build -t nginx-fixed -f Containerfile .
```

**What the commands mean:**
- `cat > Containerfile <<'EOF'` — prepares a local file or test input needed for the task.
- `EOF` — run this command as part of the task; then use the verification commands to confirm the result.
- `podman build -t nginx-fixed -f Containerfile .` — builds the image after the Dockerfile/Containerfile fix.

**Verification / proof that the fix worked:**

```bash
podman images | grep nginx-fixed
```

**Typed answer in English:**
Debian images do not include a fresh package index. apt-get update downloads package metadata so apt can find nginx.


---

## LO5.11 — FROM python:3.11 / COPY app.py /app/app.py / WORKDIR /app / CMD python...

**Full question:** FROM python:3.11 / COPY app.py /app/app.py / WORKDIR /app / CMD python app.py (the app starts but doesn't handle signals / can't be stopped cleanly — shell vs exec form?)

**What is wrong / what I am diagnosing:**
The CMD is in shell form. Exec form is better because the application receives signals directly.

**Fix commands / commands to run:**

```bash
# Use this CMD in the Containerfile:
CMD ["python", "app.py"]
```

**What the commands mean:**
- This section is mostly a manifest or theory snippet; apply/build it and use the verification commands to prove the result.

**Verification / proof that the fix worked:**

```bash
podman build -t python-fixed .
podman run --rm python-fixed
```

**Typed answer in English:**
Exec form avoids an unnecessary shell as PID 1 and handles stop signals more cleanly.


---

## LO5.12 — FROM node:20 / COPY . /app / WORKDIR /app / RUN npm install (every sou...

**Full question:** FROM node:20 / COPY . /app / WORKDIR /app / RUN npm install (every source change triggers a full npm install — reorder for layer caching.)

**What is wrong / what I am diagnosing:**
The whole source is copied before npm install, so every source change invalidates the npm install cache layer.

**Fix commands / commands to run:**

```bash
# Better order:
FROM node:20
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
```

**What the commands mean:**
- This section is mostly a manifest or theory snippet; apply/build it and use the verification commands to prove the result.

**Verification / proof that the fix worked:**

```bash
podman build -t node-fixed .
```

**Typed answer in English:**
Copying package files first allows npm install to be cached until dependencies change.


---

## LO5.13 — FROM alpine:3.20 / ENV PATH=/app/bin / RUN apk add --no-cache curl (af...

**Full question:** FROM alpine:3.20 / ENV PATH=/app/bin / RUN apk add --no-cache curl (after this, curl/sh aren't found — what did setting PATH break?)

**What is wrong / what I am diagnosing:**
ENV PATH=/app/bin overwrites the system PATH, so standard binaries are no longer found.

**Fix commands / commands to run:**

```bash
# Correct PATH line:
ENV PATH="/app/bin:${PATH}"
```

**What the commands mean:**
- This section is mostly a manifest or theory snippet; apply/build it and use the verification commands to prove the result.

**Verification / proof that the fix worked:**

```bash
podman build -t path-fixed .
podman run --rm path-fixed sh -c 'echo $PATH && which sh'
```

**Typed answer in English:**
The fix prepends /app/bin but keeps the existing system PATH.


---

## LO5.14 — FROM ubuntu:24.04 / EXPOSE 8080 / CMD ["python3", "-m", "http.server",...

**Full question:** FROM ubuntu:24.04 / EXPOSE 8080 / CMD ["python3", "-m", "http.server", "3000"]

**What is wrong / what I am diagnosing:**
The app listens on port 3000 but the Dockerfile exposes 8080. EXPOSE is only metadata and does not change the listening port.

**Fix commands / commands to run:**

```bash
# Either make the app listen on 8080:
CMD ["python3", "-m", "http.server", "8080"]
# or map host 8080 to container 3000:
podman run -p 8080:3000 image
```

**What the commands mean:**
- `podman run -p 8080:3000 image` — runs the corrected container with the intended image, command, network, port, volume or environment settings.

**Verification / proof that the fix worked:**

```bash
curl localhost:8080
```

**Typed answer in English:**
The container port in the port mapping must match the real port used by the process.


---

## LO5.15 — (you map -p 8080:8080 but nothing answers — what mismatch is there, an...

**Full question:** (you map -p 8080:8080 but nothing answers — what mismatch is there, and what does EXPOSE actually do?)

**What is wrong / what I am diagnosing:**
The mismatch is that the application is listening on another port, for example 3000, while the run command maps to 8080. EXPOSE only documents intended port; it does not publish or change the port.

**Fix commands / commands to run:**

```bash
podman run -p 8080:3000 image
```

**What the commands mean:**
- `podman run -p 8080:3000 image` — runs the corrected container with the intended image, command, network, port, volume or environment settings.

**Verification / proof that the fix worked:**

```bash
curl localhost:8080
```

**Typed answer in English:**
The correct mapping uses host port 8080 to the real container port. EXPOSE alone is not enough.


---

## LO5.16 — FROM debian:12 / RUN apt-get update && apt-get install -y build-essent...

**Full question:** FROM debian:12 / RUN apt-get update && apt-get install -y build-essential (the image is huge — how do you cut the size in the same layer?)

**What is wrong / what I am diagnosing:**
Package cache is left in the image. It should be removed in the same RUN layer.

**Fix commands / commands to run:**

```bash
RUN apt-get update && apt-get install -y --no-install-recommends build-essential \
  && rm -rf /var/lib/apt/lists/*
```

**What the commands mean:**
- `&& rm -rf /var/lib/apt/lists/*` — run this command as part of the task; then use the verification commands to confirm the result.

**Verification / proof that the fix worked:**

```bash
podman build -t smaller .
podman images | grep smaller
```

**Typed answer in English:**
Cleanup must be in the same layer; otherwise old layer data still remains in the image history.


---

## LO5.17 — FROM python:3.11 / USER appuser / COPY requirements.txt /app/requireme...

**Full question:** FROM python:3.11 / USER appuser / COPY requirements.txt /app/requirements.txt / RUN pip install -r /app/requirements.txt (permission denied errors — what's wrong with the USER/COPY ordering and ownership?)

**What is wrong / what I am diagnosing:**
The image switches to appuser before creating/owning the work directory and before installing dependencies. The user may not have permission.

**Fix commands / commands to run:**

```bash
FROM python:3.11
RUN useradd -m appuser
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY --chown=appuser:appuser . .
USER appuser
CMD ["python", "app.py"]
```

**What the commands mean:**
- This section is mostly a manifest or theory snippet; apply/build it and use the verification commands to prove the result.

**Verification / proof that the fix worked:**

```bash
podman build -t py-perm-fixed .
```

**Typed answer in English:**
Install dependencies and set ownership before switching to non-root user, or use COPY --chown for application files.


---

## LO5.18 — FROM golang:1.22 / COPY . /src / WORKDIR /src / RUN go build -o app . ...

**Full question:** FROM golang:1.22 / COPY . /src / WORKDIR /src / RUN go build -o app . / CMD ["/src/app"] (the final image is ~1 GB — how would a multi-stage build fix it?)

**What is wrong / what I am diagnosing:**
The final image contains the full Go build environment. Multi-stage build copies only the compiled binary into a smaller runtime image.

**Fix commands / commands to run:**

```bash
FROM golang:1.22 AS build
WORKDIR /src
COPY . .
RUN go build -o app .

FROM debian:12-slim
COPY --from=build /src/app /app
CMD ["/app"]
```

**What the commands mean:**
- This section is mostly a manifest or theory snippet; apply/build it and use the verification commands to prove the result.

**Verification / proof that the fix worked:**

```bash
podman build -t go-small .
podman images | grep go-small
```

**Typed answer in English:**
The build tools stay in the build stage, so the final image is much smaller.


---

## LO5.19 — A pod is stuck Pending. Walk through the diagnosis with kubectl descri...

**Full question:** A pod is stuck Pending. Walk through the diagnosis with kubectl describe pod and identify the reason from the events (scheduling / resources / PVC / nodeSelector).

**What is wrong / what I am diagnosing:**
Pending means the Pod is not scheduled yet or is waiting for a required resource such as PVC.

**Fix commands / commands to run:**

```bash
kubectl get pods
kubectl describe pod <pod>
kubectl get pvc
kubectl get nodes --show-labels
```

**What the commands mean:**
- `kubectl get pods` — lists Pods and shows whether they are Running, Ready, Pending, failing, or recreated.
- `kubectl describe pod <pod>` — shows detailed Pod information and Events; this is the main command for finding the root cause.
- `kubectl get pvc` — lists PersistentVolumeClaims and whether they are bound to storage.
- `kubectl get nodes --show-labels` — lists cluster nodes and can show node labels or readiness.

**Verification / proof that the fix worked:**

```bash
kubectl get pods
kubectl describe pod <pod> | tail -40
```

**Typed answer in English:**
I use describe and Events to find the exact reason: Insufficient cpu/memory, nodeSelector mismatch, taint, or unbound PVC. The fix depends on the event: reduce requests, label a node, add toleration, or create the PVC.


---

## LO5.20 — Find a deployment from older tasks, remove a couple of spaces and try ...

**Full question:** Find a deployment from older tasks, remove a couple of spaces and try to deploy it. Identify the cause in the error messages and fix it.

**What is wrong / what I am diagnosing:**
YAML is indentation-sensitive. Removing spaces can move fields to the wrong level or make the file invalid.

**Fix commands / commands to run:**

```bash
kubectl apply -f broken.yaml
kubectl apply -f broken.yaml --dry-run=server --validate=true
```

**What the commands mean:**
- `kubectl apply -f broken.yaml` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.
- `kubectl apply -f broken.yaml --dry-run=server --validate=true` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.

**Verification / proof that the fix worked:**

```bash
kubectl apply -f fixed.yaml --dry-run=server --validate=true
kubectl apply -f fixed.yaml
```

**Typed answer in English:**
The error message points to invalid YAML or an unknown/misplaced field. I fix indentation and validate before applying.


---

## LO5.21 — A pod is in ImagePullBackOff. Diagnose the cause (image typo, missing ...

**Full question:** A pod is in ImagePullBackOff. Diagnose the cause (image typo, missing tag, private registry without pull secret) and fix it.

**What is wrong / what I am diagnosing:**
ImagePullBackOff means Kubernetes cannot pull the container image.

**Fix commands / commands to run:**

```bash
kubectl describe pod <pod>
kubectl get pod <pod> -o yaml | grep -i image -A2
```

**What the commands mean:**
- `kubectl describe pod <pod>` — shows detailed Pod information and Events; this is the main command for finding the root cause.
- `kubectl get pod <pod> -o yaml | grep -i image -A2` — prints the Pod YAML/status and filters for the relevant error reason, for example OOMKilled.

**Verification / proof that the fix worked:**

```bash
kubectl get pods
kubectl describe pod <pod>
```

**Typed answer in English:**
I check Events for the exact pull error. I fix the image name/tag or add imagePullSecrets for a private registry. The Pod should then become Running.


---

## LO5.22 — A pod is in CrashLoopBackOff. Use kubectl logs --previous to read the ...

**Full question:** A pod is in CrashLoopBackOff. Use kubectl logs --previous to read the last crash and fix the root cause.

**What is wrong / what I am diagnosing:**
CrashLoopBackOff means the container starts, crashes, and Kubernetes restarts it repeatedly.

**Fix commands / commands to run:**

```bash
kubectl logs <pod> --previous
kubectl describe pod <pod>
```

**What the commands mean:**
- `kubectl logs <pod> --previous` — reads application/container logs; `--previous` is used for the last crashed container.
- `kubectl describe pod <pod>` — shows detailed Pod information and Events; this is the main command for finding the root cause.

**Verification / proof that the fix worked:**

```bash
kubectl get pod <pod>
kubectl get pod <pod> -o jsonpath='{.status.containerStatuses[*].restartCount}'; echo
```

**Typed answer in English:**
logs --previous shows logs from the last crashed container. I fix the application command/config, then verify restartCount stops increasing.


---

## LO5.23 — A pod's last state shows OOMKilled. Confirm it from kubectl get pod -o...

**Full question:** A pod's last state shows OOMKilled. Confirm it from kubectl get pod -o yaml / describe, then fix it (raise the memory limit or fix the app).

**What is wrong / what I am diagnosing:**
OOMKilled means the container exceeded its memory limit and the kernel/kubelet killed it.

**Fix commands / commands to run:**

```bash
kubectl describe pod <pod>
kubectl get pod <pod> -o yaml | grep -i OOM -A5
# If it is controlled by a Deployment, edit/set higher memory limit:
kubectl set resources deployment/<deploy-name> --limits=memory=512Mi --requests=memory=256Mi
```

**What the commands mean:**
- `kubectl describe pod <pod>` — shows detailed Pod information and Events; this is the main command for finding the root cause.
- `kubectl get pod <pod> -o yaml | grep -i OOM -A5` — prints the Pod YAML/status and filters for the relevant error reason, for example OOMKilled.
- `kubectl set resources deployment/<deploy-name> --limits=memory=512Mi --requests=memory=256Mi` — sets CPU/memory requests and limits on the workload container.

**Verification / proof that the fix worked:**

```bash
kubectl rollout status deployment/<deploy-name>
kubectl describe pod -l app=<label> | grep -A8 -i 'Limits\|Last State\|OOMKilled'
kubectl get pods
```

**Typed answer in English:**
First command shows Events and last state. Second command searches YAML status for OOMKilled. The fix is raising the memory limit or reducing app memory usage. After the fix, new Pods should run without OOMKilled and restartCount should stop increasing.


---

## LO5.24 — A Deployment's pods never become Ready. Trace it to a failing readines...

**Full question:** A Deployment's pods never become Ready. Trace it to a failing readiness probe and correct the probe or the app.

**What is wrong / what I am diagnosing:**
Running is not the same as Ready. A failing readiness probe keeps Pods out of Service endpoints.

**Fix commands / commands to run:**

```bash
kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>
kubectl get endpoints <service>
```

**What the commands mean:**
- `kubectl get pods` — lists Pods and shows whether they are Running, Ready, Pending, failing, or recreated.
- `kubectl describe pod <pod>` — shows detailed Pod information and Events; this is the main command for finding the root cause.
- `kubectl logs <pod>` — reads application/container logs; `--previous` is used for the last crashed container.
- `kubectl get endpoints <service>` — shows which Pod IPs the Service is actually routing to; empty endpoints usually mean label/readiness problems.

**Verification / proof that the fix worked:**

```bash
kubectl get pods
kubectl get endpoints <service>
kubectl describe pod <pod> | grep -A5 Readiness
```

**Typed answer in English:**
I check the readiness probe error in describe. I fix the probe path/port or the application. After the fix, Pods show READY and endpoints are populated.


---

## LO5.25 — A Service returns nothing. Diagnose a Service/pod label mismatch using...

**Full question:** A Service returns nothing. Diagnose a Service/pod label mismatch using kubectl get endpoints.

**What is wrong / what I am diagnosing:**
If Service endpoints are empty, the Service has no Ready matching Pods.

**Fix commands / commands to run:**

```bash
kubectl get svc
kubectl get endpoints <svc>
kubectl get pods --show-labels
kubectl get svc <svc> -o yaml
```

**What the commands mean:**
- `kubectl get svc` — lists or inspects Services, including type, cluster IP, ports and selectors.
- `kubectl get endpoints <svc>` — shows which Pod IPs the Service is actually routing to; empty endpoints usually mean label/readiness problems.
- `kubectl get pods --show-labels` — lists Pods and shows whether they are Running, Ready, Pending, failing, or recreated.
- `kubectl get svc <svc> -o yaml` — lists or inspects Services, including type, cluster IP, ports and selectors.

**Verification / proof that the fix worked:**

```bash
kubectl get endpoints <svc>
kubectl run tmp --rm -it --image=busybox -- wget -qO- <svc>:<port>
```

**Typed answer in English:**
I compare the Service selector with Pod labels. I fix either selector or labels. When endpoints contain Pod IPs, the Service can route traffic.


---

## LO5.26 — Given a manifest where spec.selector.matchLabels doesn't match spec.te...

**Full question:** Given a manifest where spec.selector.matchLabels doesn't match spec.template.metadata.labels, explain the error kubectl apply returns and fix it.

**What is wrong / what I am diagnosing:**
A Deployment selector must match the Pod template labels. Otherwise the Deployment cannot manage its Pods.

**Fix commands / commands to run:**

```bash
kubectl apply -f bad-deploy.yaml
# Fix labels so both sides match, for example app: web
kubectl apply -f fixed-deploy.yaml
```

**What the commands mean:**
- `kubectl apply -f bad-deploy.yaml` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.
- `kubectl apply -f fixed-deploy.yaml` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.

**Verification / proof that the fix worked:**

```bash
kubectl get deploy
kubectl get pods --show-labels
```

**Typed answer in English:**
kubectl apply rejects the manifest because selector does not match template labels. The fix is to make spec.selector.matchLabels equal to matching labels in spec.template.metadata.labels.


---

## LO5.27 — Given a manifest whose resources.requests exceed any node's capacity, ...

**Full question:** Given a manifest whose resources.requests exceed any node's capacity, explain why the pod is Pending (Insufficient cpu/memory) and fix it.

**What is wrong / what I am diagnosing:**
The scheduler cannot place a Pod if its requests are larger than available node capacity.

**Fix commands / commands to run:**

```bash
kubectl describe pod <pod>
kubectl describe node <node>
```

**What the commands mean:**
- `kubectl describe pod <pod>` — shows detailed Pod information and Events; this is the main command for finding the root cause.
- `kubectl describe node <node>` — shows node conditions and resource pressure such as DiskPressure, MemoryPressure, or NotReady reasons.

**Verification / proof that the fix worked:**

```bash
kubectl get pods
kubectl describe pod <pod> | tail -40
```

**Typed answer in English:**
Events show Insufficient cpu or memory. I fix it by lowering requests/limits or adding a bigger node. After fixing, the Pod should schedule.


---

## LO5.28 — Given a pod mounting a PVC that doesn't exist, diagnose the FailedMoun...

**Full question:** Given a pod mounting a PVC that doesn't exist, diagnose the FailedMount/Pending and create the PVC.

**What is wrong / what I am diagnosing:**
The Pod refers to a PVC name that does not exist in the same namespace.

**Fix commands / commands to run:**

```bash
kubectl describe pod <pod>
kubectl get pvc
# Create the PVC with the exact name expected by the Pod, or fix the Pod claimName.
```

**What the commands mean:**
- `kubectl describe pod <pod>` — shows detailed Pod information and Events; this is the main command for finding the root cause.
- `kubectl get pvc` — lists PersistentVolumeClaims and whether they are bound to storage.

**Verification / proof that the fix worked:**

```bash
kubectl get pvc
kubectl describe pod <pod>
kubectl get pods
```

**Typed answer in English:**
Events show FailedMount or claim not found. After creating the PVC or fixing claimName, the Pod can mount storage.


---

## LO5.29 — A container based on ubuntu with no long-running command shows Complet...

**Full question:** A container based on ubuntu with no long-running command shows Completed/CrashLoopBackOff. Explain why and add a proper command.

**What is wrong / what I am diagnosing:**
A container exits when its main process exits. Ubuntu without a long-running command can finish immediately.

**Fix commands / commands to run:**

```bash
# Add to Pod spec:
command: ["sleep", "3600"]
```

**What the commands mean:**
- This section is mostly a manifest or theory snippet; apply/build it and use the verification commands to prove the result.

**Verification / proof that the fix worked:**

```bash
kubectl get pods
kubectl logs <pod>
```

**Typed answer in English:**
The fix is to run a long-running process for test containers. Then the Pod stays Running instead of Completed.


---

## LO5.30 — Use kubectl get events --sort-by=.lastTimestamp (or kubectl events) to...

**Full question:** Use kubectl get events --sort-by=.lastTimestamp (or kubectl events) to triage everything that happened to a pod over time.

**What is wrong / what I am diagnosing:**
Events show the chronological Kubernetes actions and failures.

**Fix commands / commands to run:**

```bash
kubectl get events --sort-by=.lastTimestamp | tail -50
# if supported:
kubectl events
```

**What the commands mean:**
- `kubectl get events --sort-by=.lastTimestamp | tail -50` — shows cluster events in time order, which helps identify what failed first.
- `kubectl events` — shows cluster events in time order, which helps identify what failed first.

**Verification / proof that the fix worked:**

```bash
kubectl get events --sort-by=.lastTimestamp | tail -20
```

**Typed answer in English:**
I use events to see scheduling, image pulling, mount failures, probe failures, restarts, and other cluster actions in time order.


---

## LO5.31 — Use kubectl exec -it to get a shell in a pod and debug DNS with nslook...

**Full question:** Use kubectl exec -it to get a shell in a pod and debug DNS with nslookup and cat /etc/resolv.conf.

**What is wrong / what I am diagnosing:**
exec enters a running container so I can check DNS from inside the cluster network.

**Fix commands / commands to run:**

```bash
kubectl exec -it <pod> -- sh
cat /etc/resolv.conf
nslookup <service>
wget -qO- <service>:<port>
```

**What the commands mean:**
- `kubectl exec -it <pod> -- sh` — runs a command inside an existing Pod/container to prove what is visible or reachable from inside.
- `cat /etc/resolv.conf` — run this command as part of the task; then use the verification commands to confirm the result.
- `nslookup <service>` — checks DNS resolution from inside the debug Pod/container.
- `wget -qO- <service>:<port>` — tests HTTP access from inside the cluster/container.

**Verification / proof that the fix worked:**

```bash
nslookup <service>
cat /etc/resolv.conf
```

**Typed answer in English:**
This verifies the Pod DNS configuration and whether the Service name resolves inside the cluster.


---

## LO5.32 — Use an ephemeral debug container (kubectl debug -it <pod> --image=busy...

**Full question:** Use an ephemeral debug container (kubectl debug -it <pod> --image=busybox --target=<container>) to troubleshoot a no-shell/distroless container.

**What is wrong / what I am diagnosing:**
Distroless containers often have no shell or tools, so I attach a temporary debug container.

**Fix commands / commands to run:**

```bash
kubectl debug -it <pod> --image=busybox --target=<container> -- sh
```

**What the commands mean:**
- `kubectl debug -it <pod> --image=busybox --target=<container> -- sh` — adds an ephemeral debug container with tools to troubleshoot a minimal/no-shell container.

**Verification / proof that the fix worked:**

```bash
ip addr
nslookup <service>
wget -qO- <service>:<port>
```

**Typed answer in English:**
The debug container shares the Pod context and gives tools for troubleshooting without modifying the original image.


---

## LO5.33 — Spin up a throwaway kubectl run tmp --rm -it --image=busybox pod to te...

**Full question:** Spin up a throwaway kubectl run tmp --rm -it --image=busybox pod to test connectivity to a Service from inside the cluster.

**What is wrong / what I am diagnosing:**
A temporary BusyBox Pod is used to test cluster DNS and Service connectivity.

**Fix commands / commands to run:**

```bash
kubectl run tmp --rm -it --image=busybox -- sh
nslookup <service>
wget -qO- <service>:<port>
exit
```

**What the commands mean:**
- `kubectl run tmp --rm -it --image=busybox -- sh` — starts a temporary BusyBox debug Pod and removes it after exit; useful for DNS and Service tests.
- `nslookup <service>` — checks DNS resolution from inside the debug Pod/container.
- `wget -qO- <service>:<port>` — tests HTTP access from inside the cluster/container.
- `exit` — exits the temporary debug shell so the `--rm` Pod can be removed.

**Verification / proof that the fix worked:**

```bash
kubectl get pods
kubectl get endpoints <service>
```

**Typed answer in English:**
This proves whether the Service is reachable from inside the cluster, which is where ClusterIP Services are meant to work.


---

## LO5.34 — A node shows NotReady. List what you'd check (kubelet, disk pressure, ...

**Full question:** A node shows NotReady. List what you'd check (kubelet, disk pressure, CNI) and inspect it on minikube with kubectl describe node and sudo minikube logs.

**What is wrong / what I am diagnosing:**
Node NotReady means Kubernetes should not schedule normal Pods there.

**Fix commands / commands to run:**

```bash
kubectl get nodes
kubectl describe node <node>
minikube logs
```

**What the commands mean:**
- `kubectl get nodes` — lists cluster nodes and can show node labels or readiness.
- `kubectl describe node <node>` — shows node conditions and resource pressure such as DiskPressure, MemoryPressure, or NotReady reasons.
- `minikube logs` — shows Minikube logs for node/cluster troubleshooting.

**Verification / proof that the fix worked:**

```bash
kubectl get nodes
kubectl describe node <node> | grep -A10 Conditions
```

**Typed answer in English:**
I check kubelet, disk pressure, memory pressure, container runtime, and CNI/network problems. In Minikube I also check minikube logs.


---

## LO5.35 — A pod is stuck Terminating. Explain finalizers and the grace period, t...

**Full question:** A pod is stuck Terminating. Explain finalizers and the grace period, then force-delete it (--grace-period=0 --force) and state the risks.

**What is wrong / what I am diagnosing:**
Terminating can be stuck because of finalizers, volume detach, node issues, or long grace period.

**Fix commands / commands to run:**

```bash
kubectl describe pod <pod>
kubectl delete pod <pod> --grace-period=0 --force
```

**What the commands mean:**
- `kubectl describe pod <pod>` — shows detailed Pod information and Events; this is the main command for finding the root cause.
- `kubectl delete pod <pod> --grace-period=0 --force` — removes the old object if it exists, so the next create/apply command does not fail because of a name conflict.

**Verification / proof that the fix worked:**

```bash
kubectl get pods
```

**Typed answer in English:**
Force delete removes the API object immediately, but it is risky because the real process/storage cleanup may not be finished.


---

## LO5.36 — Given a manifest with the wrong apiVersion/kind pairing (e.g. apps/v1 ...

**Full question:** Given a manifest with the wrong apiVersion/kind pairing (e.g. apps/v1 + Pod), explain the validation error and correct the group/version.

**What is wrong / what I am diagnosing:**
apiVersion must match the resource kind.

**Fix commands / commands to run:**

```bash
# Pod uses:
apiVersion: v1
kind: Pod
# Deployment uses:
apiVersion: apps/v1
kind: Deployment
```

**What the commands mean:**
- This section is mostly a manifest or theory snippet; apply/build it and use the verification commands to prove the result.

**Verification / proof that the fix worked:**

```bash
kubectl apply --dry-run=server --validate=true -f file.yaml
```

**Typed answer in English:**
Kubernetes rejects wrong apiVersion/kind pairs because the resource is not valid in that API group/version.


---

## LO5.37 — A pod fails with CreateContainerConfigError because a configMapKeyRef ...

**Full question:** A pod fails with CreateContainerConfigError because a configMapKeyRef key is missing. Diagnose and fix.

**What is wrong / what I am diagnosing:**
The Pod references a ConfigMap key that does not exist.

**Fix commands / commands to run:**

```bash
kubectl describe pod <pod>
kubectl get configmap <name> -o yaml
# Add the missing key or fix the key name in the Pod manifest.
```

**What the commands mean:**
- `kubectl describe pod <pod>` — shows detailed Pod information and Events; this is the main command for finding the root cause.
- `kubectl get configmap <name> -o yaml` — run this command as part of the task; then use the verification commands to confirm the result.

**Verification / proof that the fix worked:**

```bash
kubectl get pods
kubectl describe pod <pod>
```

**Typed answer in English:**
CreateContainerConfigError happens before container start. The fix is to make the referenced ConfigMap key exist in the same namespace.


---

## LO5.38 — A pod can't mount a Secret that lives in a different namespace. Explai...

**Full question:** A pod can't mount a Secret that lives in a different namespace. Explain namespace scoping and fix it.

**What is wrong / what I am diagnosing:**
Secrets are namespace-scoped. A Pod can only mount Secrets from its own namespace.

**Fix commands / commands to run:**

```bash
kubectl get secret -n <secret-namespace>
kubectl get pod <pod> -n <pod-namespace>
kubectl create secret generic mysecret --from-literal=password=s3cret -n <pod-namespace>
```

**What the commands mean:**
- `kubectl get secret -n <secret-namespace>` — run this command as part of the task; then use the verification commands to confirm the result.
- `kubectl get pod <pod> -n <pod-namespace>` — checks the current Pod status.
- `kubectl create secret generic mysecret --from-literal=password=s3cret -n <pod-namespace>` — creates or updates a generic Secret with literal values or files.

**Verification / proof that the fix worked:**

```bash
kubectl get secret -n <pod-namespace>
kubectl describe pod <pod> -n <pod-namespace>
```

**Typed answer in English:**
The fix is to create/copy the Secret into the same namespace as the Pod or move the Pod to the namespace containing the Secret.


---

## LO5.39 — A rollout is stuck with ProgressDeadlineExceeded. Use kubectl describe...

**Full question:** A rollout is stuck with ProgressDeadlineExceeded. Use kubectl describe deployment / rollout status to find the cause and remediate.

**What is wrong / what I am diagnosing:**
ProgressDeadlineExceeded means the Deployment did not make progress before its deadline.

**Fix commands / commands to run:**

```bash
kubectl rollout status deployment/<name>
kubectl describe deployment <name>
kubectl get pods
kubectl describe pod <bad-pod>
```

**What the commands mean:**
- `kubectl rollout status deployment/<name>` — waits for the rollout to finish or shows that the rollout is stuck.
- `kubectl describe deployment <name>` — shows Deployment conditions, rollout events, image and readiness/progress problems.
- `kubectl get pods` — lists Pods and shows whether they are Running, Ready, Pending, failing, or recreated.
- `kubectl describe pod <bad-pod>` — shows detailed Pod information and Events; this is the main command for finding the root cause.

**Verification / proof that the fix worked:**

```bash
kubectl rollout undo deployment/<name>
kubectl rollout status deployment/<name>
```

**Typed answer in English:**
Common causes are bad image, failing readiness probe, insufficient resources, or CrashLoopBackOff. Fix root cause or rollback.


---

## LO5.40 — Catch indentation/field typos (e.g. imagePullpolicy, misnested ports) ...

**Full question:** Catch indentation/field typos (e.g. imagePullpolicy, misnested ports) before applying with kubectl apply --dry-run=server / --validate=true, then fix them.

**What is wrong / what I am diagnosing:**
Validation catches schema and indentation/field mistakes before real apply.

**Fix commands / commands to run:**

```bash
kubectl apply -f file.yaml --dry-run=server --validate=true
```

**What the commands mean:**
- `kubectl apply -f file.yaml --dry-run=server --validate=true` — applies a YAML manifest; Kubernetes creates or updates the objects defined in that file.

**Verification / proof that the fix worked:**

```bash
kubectl apply -f fixed.yaml --dry-run=server --validate=true
kubectl apply -f fixed.yaml
```

**Typed answer in English:**
I use server-side dry run to catch invalid fields like imagePullpolicy instead of imagePullPolicy and misnested YAML before changing the cluster.


---

## LO5.41 — An app's logs say it can't reach its database Service. Verify in order...

**Full question:** An app's logs say it can't reach its database Service. Verify in order: pod running → Service exists → endpoints populated → DNS resolves → correct port — and identify the broken link.

**What is wrong / what I am diagnosing:**
I must check the full connection chain in order.

**Fix commands / commands to run:**

```bash
kubectl get pods
kubectl get svc
kubectl get endpoints <db-service>
kubectl run tmp --rm -it --image=busybox -- nslookup <db-service>
kubectl run tmp --rm -it --image=busybox -- nc -zv <db-service> <port>
kubectl get svc <db-service> -o yaml
```

**What the commands mean:**
- `kubectl get pods` — lists Pods and shows whether they are Running, Ready, Pending, failing, or recreated.
- `kubectl get svc` — lists or inspects Services, including type, cluster IP, ports and selectors.
- `kubectl get endpoints <db-service>` — shows which Pod IPs the Service is actually routing to; empty endpoints usually mean label/readiness problems.
- `kubectl run tmp --rm -it --image=busybox -- nslookup <db-service>` — starts a temporary BusyBox debug Pod and removes it after exit; useful for DNS and Service tests.
- `kubectl run tmp --rm -it --image=busybox -- nc -zv <db-service> <port>` — starts a temporary BusyBox debug Pod and removes it after exit; useful for DNS and Service tests.
- `kubectl get svc <db-service> -o yaml` — lists or inspects Services, including type, cluster IP, ports and selectors.

**Verification / proof that the fix worked:**

```bash
kubectl get endpoints <db-service>
kubectl logs <app-pod>
```

**Typed answer in English:**
The first failed step is the root cause: app Pod not running, Service missing, endpoints empty, DNS failure, or wrong port/targetPort.


---

## LO5.42 — Read a container's state/lastState and restartCount with kubectl get p...

**Full question:** Read a container's state/lastState and restartCount with kubectl get pod -o jsonpath to determine the root cause of repeated restarts.

**What is wrong / what I am diagnosing:**
Container status fields show current state, previous state, and restart count.

**Fix commands / commands to run:**

```bash
kubectl get pod <pod> -o jsonpath='{.status.containerStatuses[*].state}'; echo
kubectl get pod <pod> -o jsonpath='{.status.containerStatuses[*].lastState}'; echo
kubectl get pod <pod> -o jsonpath='{.status.containerStatuses[*].restartCount}'; echo
```

**What the commands mean:**
- `kubectl get pod <pod> -o jsonpath='{.status.containerStatuses[*].state}'; echo` — reads a specific status field from the Pod using jsonpath.
- `kubectl get pod <pod> -o jsonpath='{.status.containerStatuses[*].lastState}'; echo` — reads a specific status field from the Pod using jsonpath.
- `kubectl get pod <pod> -o jsonpath='{.status.containerStatuses[*].restartCount}'; echo` — reads a specific status field from the Pod using jsonpath.

**Verification / proof that the fix worked:**

```bash
kubectl describe pod <pod>
kubectl logs <pod> --previous
```

**Typed answer in English:**
state shows current status, lastState shows the previous crash reason such as OOMKilled or Error, and restartCount shows how often it restarted.


---

## LO5.43 — Compare OpenShift Route to Ingress.

**Full question:** Compare OpenShift Route to Ingress.

**What is wrong / what I am diagnosing:**
Both expose HTTP/HTTPS applications externally, but Route is OpenShift-native and Ingress is the Kubernetes standard.

**Fix commands / commands to run:**

```bash
# No command is required if this is a theory comparison. On OpenShift, inspect routes with:
oc get routes
# On Kubernetes, inspect ingress with:
kubectl get ingress
```

**What the commands mean:**
- `oc get routes` — run this command as part of the task; then use the verification commands to confirm the result.
- `kubectl get ingress` — run this command as part of the task; then use the verification commands to confirm the result.

**Verification / proof that the fix worked:**

```bash
oc get routes
kubectl get ingress
```

**Typed answer in English:**
Ingress is more portable across Kubernetes distributions but needs an Ingress Controller. Route is integrated with OpenShift router and TLS workflow, but it is OpenShift-specific.
