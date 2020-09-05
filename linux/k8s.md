# Goal

1. 深入理解Linux操作系统, 掌握主流linux服务器系统安装配置维护能力 
2. 熟悉常见的服务器基础设施（Nginx、Rabbitmq、Kafka、Mysql、Redis 、ELK、Jenkins、Gitlab、Memcache、Tomcat等）的安装、维护和优化; 
3. 熟悉CI/CD技术和流程，有项目持续集成经验，使用过Jenkins+Maven+Nexus+Git等工具 
4. 熟悉微服务，网关、熟悉网关配置，能合理的实施负载均衡 
5. 熟悉docker以及任务编排，熟悉k8s架构，容器化经验2年以上； 
6. 熟悉主流云计算产品，对云计算产品的配置管理有一定经验。 
7. 熟练掌握开源监控软件zabbix prometheus等 
8. 熟悉Python语言，熟悉Shell脚本编写，有一定编程能力； 
9. 熟悉shell脚本编写, 具备python和golang语言开发能力 
10. 熟悉常用NAS设备，精通分布式存储系统ceph 
11. 至少熟悉一种配置管理工具, 如salt, ansible等 
12. 掌握RAID，ipmi等服务器技术原理和应用 

------
# K8S


## HA
__TTL Mechanism__ and __Lease__ 

`bidirectional gRPC stream`

## 自签名SSL证书

```
$ mkdir secrets
$ openssl req -newkey rsa:4096 -nodes -sha256 -keyout secrets/domain.key -x509 -day 365 -out secrets/domain.crt
$ ls secrets
$ openssl rand -hex -out secrets/http.secret 8
$ docker run -i httpd /bin/bash -c 'echo my-super-secure-password | /usr/local/apache2/bin/htpasswd -nBi user01' > secrets/registry_passwd
$ kubectl create secret generic registry-secrets --from-file secrets/

```


## 集群管理高级

### kubeconfig 高阶配置

__kubeconfig__ 管理集群、上下文、认证；可以在不同集群不同上下文进行环境的切换

`kubectl config`

![kubeconfig contains three parameters:user, cluster, and context](./e705eb01-9990-424f-8967-f63d4de49904.png)

```
# 查看 kubernetes 配置
$ kubectl config view
......

# 备份
$ cp ~/.kube/config ~/original-kubeconfig

# 查看配置具体选项
$ kubectl config set-credentials -h

# 复制到新目录
$ cp ~/original-kubeconfig ~/new-kubeconfig

# 添加用户
$ kubectl config set-credentials local@local.com --username=local-local --password=123qwe --kubeconfig="new-kubeconfig"
```

#### Context
`Context` 包含了cluster, namespaces, user;

```
# 设置Context
S kubectl config set-context <context_name> --user=<credential_name> --namespaces=<namespace_name> --cluster=<cluster_name>
$ kubectl config set-context defalut/local/local --user=local@local.com --namespaces=defalut --cluster=cluster.local

# 查看配置
$ kubectl config view 

# 当前上下文
$ kubectl config current-context

# 切换上下文
$ kubectl config use-context defalut/local/local

# 删除
$ kubectl config delete-cluster local-cluster

$ kubectl unset local@local.com

$ cp ./original-kubeconfig ~/.kube/config

```


### node 资源配置

### WebUI

### RESTful API

### Kubernetes DNS

### Authentication and Authorization

------- 
#  minikube
mac 更新后可能会出现 ` zsh: command not found: minikube `;  需要 ` brew link minikube `


# CKA


custom certificates : ` --cert-dir/certificatedsDir `

external CA : ` --controllers=csrsigner `

check certification expiration : ` kubeadm alpha certs check-expiration `

turn off automatic certificate renewal : `  kubeadm upgrade apply/kubeadm upgrade node  ----certificate-renewal=false `

 Manual certificate renewal : ` kubeadm alpha certs renew ` renew ` /etc/kubernetes/pki `

 ### Renew certificates with the Kubernetes certificates API

1. active built-in signer : ` --cluster-signing-cert-file ` and ` --cluster-signing-key-file `
2. Create certificate signing requests (CSR) : ` kubeadm alpha certs renew --use-api `
3. Approve certificate signing requests : `  kubectl certificate approve kubeadm-cert-kube-apiserver-ld526 `

### Renew certificates with external CA

 1. Create certificate signing requests : `  kubeadm alpha certs renew --csr-only `
 
### config resource

#### ` LimitRange ` config CPU and Memory

``` yaml
  limits:
  - max:
      memory: 1Gi
      cpu: 800m
    min:
      memory: 500Mi
      cpu: 200m
    default:
      memory: 1Gi
      cpu: 800m
    defaultRequest:
      memory: 1Gi      
      cpu: 800m
```

#### ` ResourceQuota ` set ResourceQuota

ResourceQuota constraine of all Container

``` yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: mem-cpu-demo
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    pods: "2"
    persistentvolumeclaims: "1"
    services.loadbalancers: "2"
    services.nodeports: "0"

```

### Access Clusters Using the Kubernetes API

``` bash
# Check all possible clusters, as your .KUBECONFIG may have multiple contexts:
kubectl config view -o jsonpath='{"Cluster name\tServer\n"}{range .clusters[*]}{.name}{"\t"}{.cluster.server}{"\n"}{end}'

# Select name of cluster you want to interact with from above output:
export CLUSTER_NAME="some_server_name"

# Point to the API server referring the cluster name
APISERVER=$(kubectl config view -o jsonpath="{.clusters[?(@.name==\"$CLUSTER_NAME\")].cluster.server}")

# Gets the token value
TOKEN=$(kubectl get secrets -o jsonpath="{.items[?(@.metadata.annotations['kubernetes\.io/service-account\.name']=='default')].data.token}"|base64 --decode)

# Explore the API with TOKEN
curl -X GET $APISERVER/api --header "Authorization: Bearer $TOKEN" --insecure
```

### Accessing the API from within a Pod


pod crt file tree : ` /var/run/secrets/kubernetes.io/serviceaccount/ca.crt `

default namespace API : ` /var/run/secrets/kubernetes.io/serviceaccount/namespace `


#### Without using a proxy

``` yaml
# Point to the internal API server hostname
APISERVER=https://kubernetes.default.svc

# Path to ServiceAccount token
SERVICEACCOUNT=/var/run/secrets/kubernetes.io/serviceaccount

# Read this Pod's namespace
NAMESPACE=$(cat ${SERVICEACCOUNT}/namespace)

# Read the ServiceAccount bearer token
TOKEN=$(cat ${SERVICEACCOUNT}/token)

# Reference the internal certificate authority (CA)
CACERT=${SERVICEACCOUNT}/ca.crt

# Explore the API with TOKEN
curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api
```

## Access Services Running on Clusters

### Accessing services running on the cluster

1. ` services ` through public ips
    - `NodePort` and `LoadBalancer`



### Change the default StorageClass

1. ` kubectl patch storageclass <your-class-name> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}' `

#### 

``` yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-scheduler
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: my-scheduler-as-kube-scheduler
subjects:
- kind: ServiceAccount
  name: my-scheduler
  namespace: kube-system
roleRef:
  kind: ClusterRole
  name: system:kube-scheduler
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    component: scheduler
    tier: control-plane
  name: my-scheduler
  namespace: kube-system
spec:
  selector:
    matchLabels:
      component: scheduler
      tier: control-plane
  replicas: 1
  template:
    metadata:
      labels:
        component: scheduler
        tier: control-plane
        version: second
    spec:
      serviceAccountName: my-scheduler
      containers:
      - command:
        - /usr/local/bin/kube-scheduler
        - --address=0.0.0.0
        - --leader-elect=false
        - --scheduler-name=my-scheduler
        image: gcr.io/my-gcp-project/my-kube-scheduler:1.0
        livenessProbe:
          httpGet:
            path: /healthz
            port: 10251
          initialDelaySeconds: 15
        name: kube-second-scheduler
        readinessProbe:
          httpGet:
            path: /healthz
            port: 10251
        resources:
          requests:
            cpu: '0.1'
        securityContext:
          privileged: false
        volumeMounts: []
      hostNetwork: false
      hostPID: false
      volumes: []
```

# Kubernetes in Action

## Service:enabling clients to discover and talk to pod

#### TLS traffic

HTTPS client and Ingress Controller is encrypted, but controller and backend is not.

Create a TLS certificate for Ingress

``` shell

# generate cet file and key
openssl genrsa -out tls.key 2048
openssl req -new -x509 -key tls.key -out tls.cert -days 360 -subj /CN=kubia.example.com

# create secret in k8s
kubectl create secreate tls tls-secret --cert=tls.cert --key=tls.key

```

Ingress yaml

``` yaml
......
metadata:
  name: kubia
spec:
  tls:
    - hosts:
      - kubia.example.com
      secretName: tls-secret
......
```

### Signaling when a pod is ready to accept connections

Three Type of Readiness probes:
1. `exec`
2. `HTTP GET` 
3. `TCP Socket`

### Headless Service

Set service spec `clusterIP: None`. Headless Service No proxy, traffice directly transfered to Pod

Headless service:

``` yaml
......
spec:
  clusterIP: None
  ports:
    ......
......
```

## Volumes: attaching disk storage to container

### `emptyDir` Volume

create emptyDir in memory:

``` yaml
volumes:
  - name: html
    emptyDir:
      medium: Memory
```