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
- Env: export MASTER_DISK_SIZE=500GB MASTER_ROOT_DISK_SIZE=500GB KUBE_GCE_ZONE=us-central1-b MASTER_SIZE=n1-highmem-96 NODE_SIZE=n1-highmem-16 NUM_NODES=55 NODE_DISK_SIZE=500GB GOPATH=$HOME/go KUBE_GCE_ENABLE_IP_ALIASES=true KUBE_GCE_PRIVATE_CLUSTER=true CREATE_CUSTOM_NETWORK=true ETCD_QUOTA_BACKEND_BYTES=8589934592 TEST_CLUSTER_LOG_LEVEL=--v=2 ENABLE_KCM_LEADER_ELECT=false ENABLE_SCHEDULER_LEADER_ELECT=false SHARE_PARTITIONSERVER=true APISERVERS_EXTRA_NUM=0 WORKLOADCONTROLLER_EXTRA_NUM=0 ETCD_EXTRA_NUM=0 LOGROTATE_FILES_MAX_COUNT=50 LOGROTATE_MAX_SIZE=200M KUBEMARK_NUM_NODES=5000 KUBE_GCE_INSTANCE_PREFIX=daily1005-1a0w1e KUBE_GCE_NETWORK=daily1005-1a0w1e
- Cmd: 
GOPATH=$HOME/go nohup ./perf-tests/clusterloader2/run-e2e.sh --nodes=5000 --provider=kubemark --kubeconfig=/home/sonyali/go/src/k8s.io/arktos/test/kubemark/resources/kubeconfig.kubemark --report-dir=/home/sonyali/logs/perf-test/gce-5000/arktos/1005-daily1005-1a0w1e --testconfig=testing/load/config.yaml --testconfig=testing/density/config.yaml 


## 10/05/2020
### Changes
https://github.com/futurewei-cloud/arktos-perftest/commits/perf-20201005
- Bugfix in ETCD3 store
- Enable mutex/block profiling for kube-apiserver
- Add trace for ETCD create and delete
### Result
- Perf-tests load testing finished with bunch of object creation failure because “connection refused”
- Kube-apiserver crashed with error “killing connection/stream”
- Checked ETCD logs, there is no error log except warning “took too long to execute”
- logs can be found under GCP project: workload-controller-manager on sonyadev4: /home/sonyali/logs/perf-test/gce-5000/arktos/1005-daily1005-1a0w1e
 
