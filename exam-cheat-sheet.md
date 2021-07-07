# Exam Cheat Sheet

# Pod
k run NAME --image=IMAGE $dr
k run tmp --restart=Never --rm -it --image=nginx:alpine -- curl -m 5 IP:PORT
k run tmp --restart=Never --rm -it --image=busybox -- wget -O- IP:PORT
k run busybox --restart=Never --rm -it --image=busybox -- sh -c env
k set image po nginx nginx=nginx:1.91
## Arguments
--port=PORT 
--env=KEY=VALUE 
--labels=KEY=VALUE
--requests='cpu=100m,memory=256Mi' 
--limits='cpu=200m,memory=512Mi'
--serviceaccount=SA
--restart=Never 
--rm 
-it
-- sh -c 'echo hello;sleep 3600'
--expose # Create pod and service
> FILE.yaml

# Configmap
k create configmap config --from-literal=foo=lala --from-literal=foo2=lolo
echo -e "foo3=lili\nfoo4=lele" > config.txt && k create cm configmap2 --from-file=config.txt
echo -e "var1=val1\n# this is a comment\n\nvar2=val2\n#anothercomment" > config.env && k create cm configmap3 --from-env-file=config.env
echo -e "var3=val3\nvar4=val4" > config4.txt && k create cm configmap4 --from-file=special=config4.txt

# Secret
k create secret generic mysecret --from-literal=password=mypass
echo -n admin > username && k create secret generic mysecret2 --from-file=username
k get secret mysecret2 -o jsonpath='{.data.username}{"\n"}' | base64 -d

# Label
k label po nginx key=value
k label po nginx key=value --overwrite
k label po nginx1 nginx2 nginx3 app-
k get po -L app
k get po -l app=v2
k get po -l app=my-app,environment=production
k get po -l environment!=production
k get po -l 'environment in (production,development)'

# Annotation
k annotate po nginx1 nginx2 nginx3 description='my description'
k annotate po nginx{1..3} description-
k describe po nginx | grep -i 'annotations'

# Deployment
k create deploy NAME --image=IMAGE $dr
k create deploy NAME --image=IMAGE && k label deploy NAME KEY=VALUE
## Arguments
--replicas=N
--port=PORT
> FILE.yaml
## Manage deployments
k set image deploy nginx nginx=nginx:1.91
k rollout status deploy nginx
k rollout history deploy nginx --revision=4
k rollout undo deploy nginx # or
k rollout undo deploy nginx --to-revision=2
k rollout pause deploy nginx
k rollout resume deploy nginx
k scale deploy nginx --replicas=5
k autoscale deploy nginx --min=5 --max=10 --cpu-percent=80

# Job
k create job NAME --image=IMAGE $dr -- sh -c 'echo hello;sleep 3600' > FILE.yaml

# Cronjob
k create cronjob NAME --image=IMAGE --schedule="* * * * *" $dr -- sh -c 'echo hello;sleep 3600' > FILE.yaml

# Service
k create service clusterip NAME --tcp PORT:TARGET_PORT $dr > FILE.yaml
## Service from po/deploy
k expose po/deploy NAME --name=SVC_NAME --port=PORT --target-port=PORT (--type=ClusterIP,NodePort)
## Create a deployment and then a pod exposing a service to copy the content of the pod to the deployment file
## Copy the content of pod.spec into deploy.spec.template, delete the pod template and run
k create deploy NAME --image=IMAGE $dr > deployment.yaml
k run NAME --image=IMAGE --port=PORT --env=KEY=VALUE --labels=KEY=VALUE --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi' --expose $dr -- sh -c 'sleep 3600' >> deployment.yaml

# Quota
kubectl create quota myrq --hard=cpu=1,memory=1G,pods=2 $dr

# Taint
k taint nodes NODE KEY=VALUE:EFFECT # NoExecute, NoSchedule, PreferNoSchedule
k taint nodes node1 key1=value1:NoSchedule- # remove taint

# Tolerations and nodeSelector
tolerations:
- effect: NoSchedule
  key: node-role.kubernetes.io/master
nodeSelector:
  node-role.kubernetes.io/master: ""

# NodeAffinity
requiredDuringSchedulingIgnoredDuringExecution

# Nodes
k drain NODE # drain & cordon
k cordon NODE
k uncordon NODE

# Service Account + Role + RoleBinding + test auth
k create sa SA_NAME
k create role ROLE_NAME --verb=create --resource=secret --resource=configmap
k create rolebinding NAME --role ROLE_NAME --serviceaccount NAMESPACE:SA_NAME
k auth can-i create secret --as system:serviceaccount:NAMESPACE:SA_NAME

# DaemonSet
k -n NAMESPACE create deployment NAME --image=IMAGE # change kind to DaemonSet; remove spec.replicas, spec.strategy, status

# kubeadm
apt-get upgrade -y kubeadm=1.12.0-00
kubeadm upgrade apply v1.12.0
apt-get upgrade -y kubelet=1.12.0-00 # if master node has kubelet
systemctl restart kubelet # if master node has kubelet
k drain NODE
ssh NODE
apt-get upgrade -y kubeadm=1.12.0-00
apt-get upgrade -y kubelet=1.12.0-00
systemctl restart kubelet
k uncordon NODE
# join node to existing cluster
ssh MASTER
kubeadm token create --print-join-command
kubeadm token list
ssh WORKER
kubeadm join 192.168.100.31:6443 --token 9k3y1v.uptus84kbp38tamd --discovery-token-ca-cert-hash sha256:11bf0d439e7dfce0ee98d34d7333fc8933cde32e27c67b63a14dbd7c052d149b
k get node​
# kubeadm certs expir
kubeadm certs check-expiration | grep apiserver
kubeadm certs renew apiserver

# etcd
export ETCDCTL_API=3
kubectl get pods -n kube-system
kubectl exec etcd-master -n kube-system -- etcdctl version # etcd-minikube

etcdctl snapshot save 
etcdctl endpoint health
etcdctl get
etcdctl put

cat /etc/kubernetes/manifests/etcd.yaml
ETCDCTL_API=3 etcdctl --cacert="" --cert="" --key="" snapshot save PATH
ETCDCTL_API=3 etcdctl snapshot status PATH
ETCDCTL_API=3 etcdctl snapshot restore PATH --data-dir=NEW_PATH
vim /etc/kubernetes/manifests/etcd.yaml # change the volume path for the new snapshot (volumes[X].hostPath.path)

# Certs
openssl x509 -in file-path.crt -text -noout

# liveness and readiness
livenessProbe:
  exec:
    command:
    - 'true'
readinessProbe:
  exec:
    command:
    - sh
    - -c
    - 'wget -T2 -O- http://service-am-i-ready:80'

# kubectl basic help
k top node
k top pod # --containers=true for containers also
k get all
k get pods --selector KEY=VALUE
k explain po.spec --recursive
k exec nginx -- sh -c 'while true; do echo hello; sleep 2;done'
k exec secret-handler -- find /tmp/secret2
k logs nginx > logs.txt

# jsonpath
k config view -o jsonpath="{.contexts[*].name}"
# namespaced resources
k api-resources --namespaced -o name
# record action
k scale sts o3db --replicas 1 --record
# sorting
k get pod -A --sort-by=.metadata.creationTimestamp
# Count lines
k get RESOURCE --no-headers | wc -l
# Save output from ssh to main terminal
ssh NODE "docker logs 3dffb59b81ac" &> FILE
# iptables
ssh NODE iptables-save | grep SERVICE_NAME
# Check the machine type
cat /etc/*release*
# nodeName on pod.spec.nodeName if there is no scheduler

# check components (ssh to node)
ps aux | grep kubelet
find /etc/systemd/system/ | grep kube
find /etc/systemd/system/ | grep etcd
find /etc/kubernetes/manifests/
## What is the Service CIDR?
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep range 
## Which Networking (or CNI Plugin) is configured and where is its config file?
find /etc/cni/net.d/
cat /etc/cni/net.d/10-weave.conflist
## kubelet fix
ps aux | grep kubelet
service kubelet status
service kubelet start
/usr/local/bin/kubelet
whereis kubelet
vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf # change binary path
systemctl daemon-reload && systemctl restart kubelet
# kubelet client certificate
openssl x509  -noout -text -in /var/lib/kubelet/pki/kubelet-client-current.pem | grep Issuer
openssl x509  -noout -text -in /var/lib/kubelet/pki/kubelet.crt | grep "Extended Key Usage" -A1

# NetworkPolicy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-backend
  namespace: project-snake
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Egress                    
  egress:
    - to:                       
      - podSelector:
          matchLabels:
            app: db1
      ports:                    
      - protocol: TCP
        port: 1111
    - to:                       
      - podSelector:
          matchLabels:
            app: db2
      ports:                    
      - protocol: TCP
        port: 2222

# Set environment
alias k=kubectl
alias kubens='kubectl config set-context --current --namespace '
export dr="--dry-run=client -o yaml"
export gp="--grace-period=0 --force"
export sl="--show-labels"
source <(kubectl completion bash)
complete -F __start_kubectl k

# Set vim config
vim ~/.vimrc

set tabstop=2
set softtabstop=2
set expandtab
set shiftwidth=2
set number

# Vim basic help
Copy lines: y
Cut lines: d
Paste lines: p
:set nonumber + Enter
