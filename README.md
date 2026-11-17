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
