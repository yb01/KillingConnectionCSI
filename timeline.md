

### Common Test Setup (specify specific test change below )

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

### Definition of identification of target issue
- **kubelet** kills apiserver, log has the following:
```
killContainer "kube-apiserver"(id={"unknown" "2cdeaae2b172d97c85ecedaff11a2a2f0d4baced5c28138138efb4bde8741b84"}) for pod "kube-apiserver-june11-1a0w1e-kubemark-master_kube-system_system(87bfba049d75e1ed76c1162a027a2db7)" failed: rpc error: code = DeadlineExceeded desc = context deadline exceeded
```
Make a note of the first date and time this happened.

- **apiserver** has the following "killing connection"
```
Observed a panic: &errors.errorString{s:"killing connection/stream because serving request timed out and response had been started"}
```

And 
```
apiserver received an error that is not an metav1.Status: &errors.errorString{s:"context canceled"}
```

Usually it started from the time apiserver was killed by kubelet.

All build shall be labelled as the following:

- ![#c5f015](https://via.placeholder.com/15/c5f015/000000?text=+) means issue is not found
- ![#c5f015](https://via.placeholder.com/15/000000/000000?text=+) means this test does not fit the binary search test and can be ignored
- ![#f03c15](https://via.placeholder.com/15/f03c15/000000?text=+) means issue is found

### Investigative Metrics:
- The step that the test reached.
  Search for log message as follows in nohup.out to find it.
  ```
  I1002 06:17:49.991954    5672 simple_test_executor.go:135] Step "Starting measurements" started
  ```

  FYI, the steps in load test are as follows:
  1. Starting measurements
  2. Creating SVCs
  3. Creating PriorityClass for DaemonSets
  4. Starting measurement for waiting for pods
  5. Creating objects
  6. Waiting for pods to be running
  7. Scaling and updating objects
  8. Waiting for objects to become scaled
  9. Deleting objects
  10. Waiting for pods to be deleted
  11. Deleting PriorityClass for DaemonSets
  12. Deleting SVCs
  13. Collecting measurements


- The size of etcd. Collect the data on:
  1. size of db file ([saved_etcd_dir]/data/member/snap/db). 
  2. Number of KV pairs. 
     - start etcd using the saved etcd data. Command: etcd --data-dir [saved_etcd_dir]/data
     - get the following data
        - the number of all KV pairs. Command : etcdctl get "" --prefix=true --keys-only  | awk NF | wc -l
        - the number of deployments pairs. Command : etcdctl get "/registry/deployments" --prefix=true --keys-only  | awk NF | wc -l
        - the number of endpoints pairs. Command : etcdctl get "/registry/services/endpoints" --prefix=true --keys-only  | awk NF | wc -l     
        

### Build Date: Pre-Alkaid - Baseline: density test passed, load test failed but completed with timeouts
- Verdict: ![#c5f015](https://via.placeholder.com/15/c5f015/000000?text=+)
- Commit: 75e94764
- Runner: Vinay
- Date of run: Sep 22
- Test: density + load
- Result:
  - Density test passed. Load test failed but completed with object timeouts.
  - Logs don't show "killing connection" or panics or apiserver kills.
  - sonyafenge@sonyadev:/home/sonyafenge/vinaykul/logs/perf-test/sep22_pre_alkaid_etcd343_5k_1api

### Build Date: 04/05 - density and load tests failed but completed with timeouts
- Verdict: ![#c5f015](https://via.placeholder.com/15/c5f015/000000?text=+)
- Commit: 4fa2fd3d1385cc71a11a4e5216ea928c7a1e4ad8
- Runner: Vinay
- Date of run: Sep 23
- Test: density + load
- Result:
  - Density test passed. Load test completed with object timeouts.
  - Logs don't show "killing connection" or panics or apiserver kills.
  - sonyafenge@sonyadev:/home/sonyafenge/vinaykul/logs/perf-test/sep23_ark_apr_etcd343_5k_1api
  
### Build Date: 05/21 - load test failed but completed with timeouts
- Verdict: ![#c5f015](https://via.placeholder.com/15/c5f015/000000?text=+)
- Commit: 82b1405efe8475a8ee26fffce06ec39454aca9b4
- Runner: Vinay
- Date of run: Sep 26
- Test: load
- Result:
  - Load test completed with object timeouts.
  - Logs don't show "killing connection" or panics or apiserver kills.
  - sonyafenge@sonyadev:/home/sonyafenge/vinaykul/logs/perf-test/sep26_ark_may21_etcd343_5k_1api
  
### Build Date: 05/21 - density + load test failed but completed with timeouts
- Verdict: ![#c5f015](https://via.placeholder.com/15/c5f015/000000?text=+)
- Commit: 82b1405efe8475a8ee26fffce06ec39454aca9b4
- Runner: Vinay
- Date of run: Sep 30
- Test: density + load
- Result:
  - Load test completed with object timeouts.
  - Logs don't show "killing connection" or panics or apiserver kills.
  - sonyafenge@sonyadev:/home/sonyafenge/vinaykul/logs/perf-test/sep30_ark_may21_etcd343_5k_1api

### Build Date: 05/26 - load test failed but completed with timeouts
- Verdict: ![#c5f015](https://via.placeholder.com/15/c5f015/000000?text=+)
- Commit: 5f9c12ff6fda588abee41a107a7637895024f6f4
- Runner: Vinay
- Date of run: Sep 28
- Test: load
- Result:
  - Load test completed with object timeouts.
  - Logs don't show "killing connection" or panics or apiserver kills.
  - sonyafenge@sonyadev:/home/sonyafenge/vinaykul/logs/perf-test/sep28_ark_may26_etcd343_5k_1api

### Build Date: 06/10 No fail

### Build Date: 06/12 with WCM - load test failed but completed with lots of timeout objects, lots of 'killing connection' logs
- Verdict: ![#c5f015](https://via.placeholder.com/15/000000/000000?text=+)
- Commit: 6804ff1aa718858619b399ecb921a2450710582a
- Runner: Vinay
- Date of run: Sep 26
- Test: load
- Result:
  - Load test completed with lot of object timeouts.
  - Amount of apiserver logging indicates it was very busy
  - Logs show "killing connection", panics.
  - root@vinaydev2:/root/logs/perf-test/sep26_ark_jun12_etcd343_5k_1api
  
### Build Date: 06/12 re-run - load test failed but completed with timeouts
- Verdict: ![#c5f015](https://via.placeholder.com/15/c5f015/000000?text=+)
- Commit: 6804ff1aa718858619b399ecb921a2450710582a
- Runner: Vinay
- Date of run: Sep 28
- Test: load
- Result:
  - Load test completed with object timeouts.
  - Logs don't show "killing connection" or panics or apiserver kills.
  - root@vinaydev2:/root/logs/perf-test/sep28_ark_jun12_etcd343_5k_1api

### Build Date: 06/12 - density + load test failed but completed with timeouts
- Verdict: ![#c5f015](https://via.placeholder.com/15/c5f015/000000?text=+)
- Commit: 6804ff1aa718858619b399ecb921a2450710582a
- Runner: Vinay
- Date of run: Sep 30
- Test: density + load
- Result:
  - Load test completed with object timeouts.
  - Logs don't show "killing connection" or panics or apiserver kills.
  - root@vinaydev2:/root/logs/perf-test/sep30_ark_jun12_etcd343_5k_1api

### Build Date: 06/18

- Verdict: ![#f03c15](https://via.placeholder.com/15/f03c15/000000?text=+)

- Commit: 91f3f2bb1d89038223264eb246552e856559c050

- Runner: Joe

- Date of run:

- Result: 
  - Many  Kube-apiserver panics. “killing connection/stream”
  -  The load test completed with 1 failure that many objects got timed out as usual
  - Not many “took too long” warning of “registry/leases/kube-node-lease/hollow-node” as usual. There are only 2712 out of the 38294 “took too long” warnings. It used to be more than 80%.
  - There are many context canceled errors in the etcd.log
  - Log: jundev2:/home/shaoj9/logs/perf-test/gce-5000/arktos/0929-0618-91f3


### Build Date: 06/25 with WCM - load test failed but completed with lots of timeout objects, lots of 'killing connection' logs
- Verdict: ![#c5f015](https://via.placeholder.com/15/000000/000000?text=+)
- Commit: 6b17cef2ea78353b8981a0a896d468e97f287a84
- Runner: Vinay
- Date of run: Sep 24
- Test: load
- Result:
  - Load test completed with object timeouts.
  - Amount of apiserver logging indicates it was very busy
  - Logs show "killing connection", panics.
  - root@vinaydev2:/root/logs/perf-test/sep24_ark_jun25_etcd343_5k_1api
  
### Build Date: 06/25 re-run - load test failed but completed with timeouts
- Verdict: ![#c5f015](https://via.placeholder.com/15/c5f015/000000?text=+)
- Commit: 6b17cef2ea78353b8981a0a896d468e97f287a84
- Runner: Vinay
- Date of run: Sep 29
- Test: load
- Result:
  - Load test completed with object timeouts.
  - Logs don't show "killing connection" or panics or apiserver kills.
  - sonyafenge@sonyadev:/home/sonyafenge/vinaykul/logs/perf-test/sep29_ark_jun25_etcd343_5k_1api
  
 ### Build Date: 06/25 re-run 
- Commit: 6b17cef2ea78353b8981a0a896d468e97f287a84
- Runner: Jun
- Date of run: Oct 1
- Test: load
- Env: export MASTER_DISK_SIZE=500GB MASTER_ROOT_DISK_SIZE=500GB KUBE_GCE_ZONE=us-west1-b MASTER_SIZE=n1-highmem-96 NODE_SIZE=n1-highmem-16 NUM_NODES=55 NODE_DISK_SIZE=1000GB GOPATH=$HOME/go KUBE_GCE_ENABLE_IP_ALIASES=true KUBE_GCE_PRIVATE_CLUSTER=true CREATE_CUSTOM_NETWORK=true KUBE_GCE_INSTANCE_PREFIX=node5000-0625  KUBE_GCE_NETWORK=node5000-0625 ETCD_QUOTA_BACKEND_BYTES=8589934592 TEST_CLUSTER_LOG_LEVEL=--v=2  SHARE_PARTITIONSERVER=true APISERVERS_EXTRA_NUM=0 WORKLOADCONTROLLER_EXTRA_NUM=0 ETCD_EXTRA_NUM=0 LOGROTATE_FILES_MAX_COUNT=10 LOGROTATE_MAX_SIZE=200M KUBEMARK_NUM_NODES=5000
- Cmd: 
GOPATH=$HOME/go nohup ./clusterloader2/run-e2e.sh --nodes=5000 --provider=kubemark --kubeconfig=/home/sonyali/go/src/k8s.io/arktos/test/kubemark/resources/kubeconfig.kubemark --report-dir=/home/sonyali/logs/perf-test/gce-5000/arktos/0930-0625 --testconfig=testing/load/config.yaml 
- Result:
  - No kube-apiserver panics. There are kubelet panics invalid memory address or nil pointer dereference panics as usual.
  - There are 256 connection refused errors in apiserver.
  - There are 524587 “took too long” warnings. 404064 of them are for the key “registry/leases/kube-node-lease/hollow-node-***”
  - There are two context canceled errors in the etcd.log due to “took too long to execute”
  - The logs can be found under GCP project: Scalability-testing at  jundev: /home/shaoj9/logs/perf-test/gce-5000/arktos/0930-0625-1f3
  - Prometheus report can be found at http://34.83.153.175:9090/

  
### Build Date: 06/30 - load test failed but completed with timeouts
- Verdict: ![#c5f015](https://via.placeholder.com/15/c5f015/000000?text=+)
- Commit: 3f91cac7dcbaed9b5cb1981de329dc61e93c7376
- Runner: Vinay
- Date of run: Sep 30
- Test: load
- Result:
  - Load test completed with object timeouts.
  - Logs don't show "killing connection" or panics or apiserver kills.
  - sonyafenge@sonyadev:/home/sonyafenge/vinaykul/logs/perf-test/sep30_ark_jun30_3f91_etcd343_5k_1api

### Build Date: 07/23 - load test failed but completed with timeouts
- Commit: b82af68a72d74bd86f652cf5bb7318beee547601
- Runner: Vinay
- Date of run: Sep 30
- Test: load
- Result:
  - Load test completed with object timeouts.
  - Logs don't show "killing connection" or panics or apiserver kills.
  - root@vinaydev2:/root/logs/perf-test/sep29_ark_jul23_etcd343_5k_1api

### Build Date: 08/11 - load test in progress
- Commit: 5069e78665708f7949af52a77aa8581998d573b0.   +   Commit: 79d954c790c63f6f8af0ecec665243d1d2af95f6
- Runner: Sonya
- Date of run: Oct 1
- Test: load
- Result:
  
### Build Date: 08/13 - load test crashed (seg fault)
- Commit: 50e561d3b3e5ca125bf422560748e106a335d94c
- Runner: Vinay
- Date of run: Sep 29
- Test: load
- Result:
  - Load test completed with object timeouts.
  - Amount of apiserver logging indicates it was very busy
  - Logs show "killing connection", panics.
  - root@vinaydev2:/root/logs/perf-test/sep29_ark_aug13_etcd343_5k_1api
  
### Build Date: 09/24

- Verdict: ![#f03c15](https://via.placeholder.com/15/f03c15/000000?text=+)

- Commit: 40fd8c82fbd0446c2bf32a558004849335025120

- Runner: Sonya

- Date of run:

- Result: 
  - Kube-apiserver Panic: killing connection/stream
  - Perf-tests tool panic: runtime error: invalid memory address or nil pointer dereference
  - logs can be found under GCP project: workload-controller-manager on sonya-useast1: /home/sonyali/logs/perf-test/gce-5000/arktos/0929-communityperf-1a0w1e
  - Prometheus report can be found at [http://35.231.121.60:9090/](https://nam11.safelinks.protection.outlook.com/?url=http%3A%2F%2F35.231.121.60%3A9090%2F&data=02|01|peng.du%40futurewei.com|50f8201d0df9413f5dbf08d865752edb|0fee8ff2a3b240189c753a1d5591fedc|1|0|637370901289650492&sdata=uINL%2B1heyhSHkiGabpRKXAXtUwaE6Nstz0qmVzd24Sc%3D&reserved=0) or snapshot can be found on sonya-useast1: /home/sonyali/logs/perf-test/gce-5000/arktos/0929-communityperf-1a0w1e/logs/crashed/kubemarkmaster/prometheus/snapshots


### Build Date: 09/24

- Verdict: ![#f03c15](https://via.placeholder.com/15/f03c15/000000?text=+)

- Commit: 196d649b2c33eec55d70a96b7cc8b302c9ede698

- Runner: Jun

- Date of run: Oct 1

- Result: 
  - The load test completed with 1 failure: Object creation failed due to connection refused,  deletion error: etcdserver: too many requests, and  objects timed out.
  - ApiServer restarted several times, with many killing connection panics
  - The logs can be found under GCP project: Scalability-testing at  jundev2: /home/shaoj9/logs/perf-test/gce-5000/arktos/1001-perflog/logs
  - The prometheus report can be found at http://35.233.179.148:9090/graph and its snapshots are under GCP project: Scalability-testing at  jundev2: /home/shaoj9/logs/perf-test/gce-5000/arktos/1001-perflog/snapshots
  - Pprof are collected every 30 minutes and can be found under under GCP project: Scalability-testing at  jundev2: /home/shaoj9/logs/perf-test/gce-5000/arktos/1001-perflog/pprof

 



