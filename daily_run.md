## Common Test Setup (specify specific test change below )

- 5000 nodes
- 1 Apiserver
- 1 ETCD instance
- Load test only
- WCM disabled
- Perf-test tool: https://github.com/sonyafenge/perf-tests.git
- ETCD 3.4.3
- Leader-election disabled
- Coredump enabled
- Insecure-port enabled
- Prometheus enabled
- Env: export MASTER_DISK_SIZE=500GB MASTER_ROOT_DISK_SIZE=500GB KUBE_GCE_ZONE=us-west1-b MASTER_SIZE=n1-highmem-96 NODE_SIZE=n1-highmem-16 NUM_NODES=55 NODE_DISK_SIZE=500GB GOPATH=$HOME/go KUBE_GCE_ENABLE_IP_ALIASES=true KUBE_GCE_PRIVATE_CLUSTER=true CREATE_CUSTOM_NETWORK=true KUBE_GCE_INSTANCE_PREFIX=node5000-0625  KUBE_GCE_NETWORK=node5000-0625 ETCD_QUOTA_BACKEND_BYTES=8589934592 TEST_CLUSTER_LOG_LEVEL=--v=2  SHARE_PARTITIONSERVER=true APISERVERS_EXTRA_NUM=0 WORKLOADCONTROLLER_EXTRA_NUM=0 ETCD_EXTRA_NUM=0 LOGROTATE_FILES_MAX_COUNT=10 LOGROTATE_MAX_SIZE=200M KUBEMARK_NUM_NODES=5000
- Cmd: 
GOPATH=$HOME/go nohup ./clusterloader2/run-e2e.sh --nodes=5000 --provider=kubemark --kubeconfig=/home/sonyali/go/src/k8s.io/arktos/test/kubemark/resources/kubeconfig.kubemark --report-dir=/home/sonyali/logs/perf-test/gce-5000/arktos/0930-0625 --testconfig=testing/load/config.yaml 


## 10/05/2020
### Changes
TBD
### Result
- Perf-tests load testing finished with bunch of object creation failure because “connection refused”
- Kube-apiserver crashed with error “killing connection/stream”
- Checked ETCD logs, there is no error log except warning “took too long to execute”
- logs can be found under GCP project: workload-controller-manager on sonyadev4: /home/sonyali/logs/perf-test/gce-5000/arktos/1005-daily1005-1a0w1e
 
