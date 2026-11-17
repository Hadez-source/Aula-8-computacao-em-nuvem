# Aula-8-computacao-em-nuvem
Passo 1:
root@controlplane:~$ kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   11d   v1.37.1
node01         Ready    <none>          11d   v1.37.1
root@controlplane:~$ kubectl create namespace aula8
namespace/aula8 created
root@controlplane:~$ kubectl get namespace aula8
NAME    STATUS   AGE
aula8   Active   8s

Passo 2:
root@controlplane:~$ cat > web-saude.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: web-saude
  namespace: aula8
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet: {path: /, port: 80}
      initialDelaySeconds: 5
      periodSeconds: 5
    readinessProbe:
      httpGet: {path: /health, port: 80}
      initialDelaySeconds: 2
      periodSeconds: 5
EOF 

Passo 3:
root@controlplane:~$ kubectl apply -f web-saude.yaml
pod/web-saude created
root@controlplane:~$ kubectl -n aula8 get pod web-saude
NAME        READY   STATUS    RESTARTS   AGE
web-saude   0/1     Running   0          14s
root@controlplane:~$ kubectl -n aula8 describe pod web-saude
Name:             web-saude
Namespace:        aula8
Priority:         0
Service Account:  default
Node:             node01/172.30.2.2
Start Time:       Tue, 17 Nov 2026 00:11:04 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               192.168.1.46
IPs:
  IP:  192.168.1.46
Containers:
  nginx:
    Container ID:   containerd://a6921602f31d55439e16d524a45c28e776b0eec6bdd4b5c474be5fe079a3b19b
    Image:          nginx:alpine
    Image ID:       docker.io/library/nginx@sha256:fed7e14be83cbd2661a75cdc5b872d936cf155d46c327084d30b8df19ae88d72
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 17 Nov 2026 00:11:12 +0000
    Ready:          False
    Restart Count:  0
    Liveness:       http-get http://:80/ delay=5s timeout=1s period=5s successThreshold=1 failureThreshold=3
    Readiness:      http-get http://:80/health delay=2s timeout=1s period=5s successThreshold=1 failureThreshold=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7kwc4 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       False 
  ContainersReady             False 
  PodScheduled                True 
Volumes:
  kube-api-access-7kwc4:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age               From               Message
  ----     ------     ----              ----               -------
  Normal   Scheduled  25s               default-scheduler  Successfully assigned aula8/web-saude to node01
  Normal   Pulling    20s               kubelet            spec.containers{nginx}: Pulling image "nginx:alpine"
  Normal   Pulled     17s               kubelet            spec.containers{nginx}: Successfully pulled image "nginx:alpine" in 2.932s (2.932s including waiting). Image size: 26368486 bytes.
  Normal   Created    17s               kubelet            spec.containers{nginx}: Container created
  Normal   Started    17s               kubelet            spec.containers{nginx}: Container started
  Warning  Unhealthy  5s (x3 over 15s)  kubelet            spec.containers{nginx}: Readiness probe failed: HTTP probe failed with statuscode: 404


  Passo 4
Fase Running: o processo Nginx está em execução.
READY 0/1: a verificação de prontidão falhou.
RESTARTS 0: a liveness em / continua bem-sucedida.
Anote o evento encontrado em kubectl describe pod.

Passo 5
root@controlplane:~$ kubectl -n aula8 exec web-saude -- sh -c \
  'echo OK > /usr/share/nginx/html/health'
root@controlplane:~$ kubectl -n aula8 get pod web-saude -w
NAME        READY   STATUS    RESTARTS   AGE
web-saude   1/1     Running   0          5m15s

Passo 6
root@controlplane:~$ kubectl -n aula8 exec web-saude -- \ \
  wget -qO- http://localhost/health
OK
root@controlplane:~$ kubectl -n aula8 get pod web-saude
NAME        READY   STATUS    RESTARTS   AGE
web-saude   1/1     Running   0          8m25s

Passo 7
root@controlplane:~$ kubectl -n aula8 exec web-saude -- \
  rm /usr/share/nginx/html/health
root@controlplane:~$ kubectl -n aula8 get pod web-saude -w
NAME        READY   STATUS    RESTARTS   AGE
web-saude   1/1     Running   0          9m4s
web-saude   0/1     Running   0          9m5s

Passo 8 
^Croot@controlplane:~kubectl -n aula8 exec web-saude -- \ \
  wget -qO- http://localhost/ >/dev/null
root@controlplane:~$ kubectl -n aula8 logs web-saude --tail=20
192.168.1.92 - - [17/Nov/2026:00:20:49 +0000] "GET / HTTP/1.1" 200 896 "-" "kube-probe/1.37" "-"
2026/11/17 00:20:54 [error] 30#30: *236 open() "/usr/share/nginx/html/health" failed (2: No such file or directory), client: 192.168.1.92, server: localhost, request: "GET /health HTTP/1.1", host: "192.168.1.46:80"
192.168.1.92 - - [17/Nov/2026:00:20:54 +0000] "GET /health HTTP/1.1" 404 153 "-" "kube-probe/1.37" "-"
192.168.1.92 - - [17/Nov/2026:00:20:54 +0000] "GET / HTTP/1.1" 200 896 "-" "kube-probe/1.37" "-"
2026/11/17 00:20:59 [error] 30#30: *238 open() "/usr/share/nginx/html/health" failed (2: No such file or directory), client: 192.168.1.92, server: localhost, request: "GET /health HTTP/1.1", host: "192.168.1.46:80"
192.168.1.92 - - [17/Nov/2026:00:20:59 +0000] "GET /health HTTP/1.1" 404 153 "-" "kube-probe/1.37" "-"
192.168.1.92 - - [17/Nov/2026:00:20:59 +0000] "GET / HTTP/1.1" 200 896 "-" "kube-probe/1.37" "-"
2026/11/17 00:21:04 [error] 30#30: *240 open() "/usr/share/nginx/html/health" failed (2: No such file or directory), client: 192.168.1.92, server: localhost, request: "GET /health HTTP/1.1", host: "192.168.1.46:80"
192.168.1.92 - - [17/Nov/2026:00:21:04 +0000] "GET /health HTTP/1.1" 404 153 "-" "kube-probe/1.37" "-"
192.168.1.92 - - [17/Nov/2026:00:21:04 +0000] "GET / HTTP/1.1" 200 896 "-" "kube-probe/1.37" "-"
::1 - - [17/Nov/2026:00:21:04 +0000] "GET / HTTP/1.1" 200 896 "-" "Wget" "-"
2026/11/17 00:21:09 [error] 30#30: *243 open() "/usr/share/nginx/html/health" failed (2: No such file or directory), client: 192.168.1.92, server: localhost, request: "GET /health HTTP/1.1", host: "192.168.1.46:80"
192.168.1.92 - - [17/Nov/2026:00:21:09 +0000] "GET /health HTTP/1.1" 404 153 "-" "kube-probe/1.37" "-"
192.168.1.92 - - [17/Nov/2026:00:21:09 +0000] "GET / HTTP/1.1" 200 896 "-" "kube-probe/1.37" "-"
2026/11/17 00:21:14 [error] 30#30: *245 open() "/usr/share/nginx/html/health" failed (2: No such file or directory), client: 192.168.1.92, server: localhost, request: "GET /health HTTP/1.1", host: "192.168.1.46:80"
192.168.1.92 - - [17/Nov/2026:00:21:14 +0000] "GET /health HTTP/1.1" 404 153 "-" "kube-probe/1.37" "-"
192.168.1.92 - - [17/Nov/2026:00:21:14 +0000] "GET / HTTP/1.1" 200 896 "-" "kube-probe/1.37" "-"
2026/11/17 00:21:19 [error] 30#30: *247 open() "/usr/share/nginx/html/health" failed (2: No such file or directory), client: 192.168.1.92, server: localhost, request: "GET /health HTTP/1.1", host: "192.168.1.46:80"
192.168.1.92 - - [17/Nov/2026:00:21:19 +0000] "GET /health HTTP/1.1" 404 153 "-" "kube-probe/1.37" "-"
192.168.1.92 - - [17/Nov/2026:00:21:19 +0000] "GET / HTTP/1.1" 200 896 "-" "kube-probe/1.37" "-"
root@controlplane:~$ 


