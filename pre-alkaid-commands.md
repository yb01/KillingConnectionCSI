- apiserver
```
/bin/sh -c /usr/local/bin/kube-apiserver --insecure-bind-address=0.0.0.0 --etcd-servers=http://127.0.0.1:2379 --etcd-servers-overrides=/events#http://127.0.0.1:4002 --profiling=true --contention-profiling=true --etcd-cafile=/etc/srv/kubernetes/etcd-apiserver-ca.crt --etcd-certfile=/etc/srv/kubernetes/etcd-apiserver-client.crt --etcd-keyfile=/etc/srv/kubernetes/etcd-apiserver-client.key --tls-cert-file=/etc/srv/kubernetes/server.cert --tls-private-key-file=/etc/srv/kubernetes/server.key --requestheader-client-ca-file=/etc/srv/kubernetes/aggr_ca.crt --requestheader-allowed-names=aggregator --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --proxy-client-cert-file=/etc/srv/kubernetes/proxy_client.crt --proxy-client-key-file=/etc/srv/kubernetes/proxy_client.key --enable-aggregator-routing=true --client-ca-file=/etc/srv/kubernetes/ca.crt --token-auth-file=/etc/srv/kubernetes/known_tokens.csv --secure-port=443 --basic-auth-file=/etc/srv/kubernetes/basic_auth.csv --target-ram-mb=300000 --service-cluster-ip-range=10.0.0.0/16 --admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,Priority,StorageObjectInUseProtection,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota --authorization-mode=Node,RBAC --allow-privileged=true --storage-backend=etcd3 --min-request-timeout=300 --max-requests-inflight=3000 --max-mutating-requests-inflight=1000 --feature-gates=ExperimentalCriticalPodAnnotation=true  --runtime-config=extensions/v1beta1,scheduling.k8s.io/v1alpha1 --v=2  --delete-collection-workers=16 1>>/var/log/kube-apiserver.log 2>&1
```

- controller-manager
```
/bin/sh -c /usr/local/bin/kube-controller-manager --v=2    --use-service-account-credentials --kubeconfig=/etc/srv/kubernetes/kube-controller-manager/kubeconfig --service-account-private-key-file=/etc/srv/kubernetes/server.key --root-ca-file=/etc/srv/kubernetes/ca.crt --allocate-node-cidrs=true --cluster-cidr=10.64.0.0/11 --service-cluster-ip-range=10.0.0.0/16 --terminated-pod-gc-threshold=100 1>>/var/log/kube-controller-manager.log 2>&1
```

- scheduler
```
/bin/sh -c /usr/local/bin/kube-scheduler --v=2   --kubeconfig=/etc/srv/kubernetes/kube-scheduler/kubeconfig 1>>/var/log/kube-scheduler.log 2>&1
```

- kubelet
```
missing
```

- etcd
```
/bin/sh -c /usr/local/bin/etcd --name=etcd-prealkaid1012-1a0w1e-kubemark-master --listen-peer-urls=http://127.0.0.1:2380 --advertise-client-urls=http://127.0.0.1:2379 --listen-client-urls=http://0.0.0.0:2379 --client-cert-auth --trusted-ca-file /etc/srv/kubernetes/etcd-apiserver-ca.crt --cert-file /etc/srv/kubernetes/etcd-apiserver-server.crt --key-file /etc/srv/kubernetes/etcd-apiserver-server.key --data-dir=/var/etcd/data 1>>/var/log/etcd.log 2>&1

/bin/sh -c /usr/local/bin/etcd --name=etcd-prealkaid1012-1a0w1e-kubemark-master --listen-peer-urls=http://127.0.0.1:2380 --advertise-client-urls=http://127.0.0.1:2379 --listen-client-urls=http://0.0.0.0:2379 --client-cert-auth --trusted-ca-file /etc/srv/kubernetes/etcd-apiserver-ca.crt --cert-file /etc/srv/kubernetes/etcd-apiserver-server.crt --key-file /etc/srv/kubernetes/etcd-apiserver-server.key --data-dir=/var/etcd/data 1>>/var/log/etcd.log 2>&1

```
