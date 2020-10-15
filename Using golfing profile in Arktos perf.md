
### Overall good readings about golang profiling:
https://github.com/google/pprof/blob/master/doc/README.md

https://golang.org/pkg/runtime/pprof

https://blog.golang.org/pprof


### Profile collection in Arktos perf test
The CPU, Memory, Mutex, Block, goroutine profiles for api server is enabled as default and profile data 
are collected every 30 minutes during the test.
All profiles are copied over to the test server machine, e.g. //sonyadev4 after the test run along with
all the other logs for the test. for example, the profiles were copied to the below folder for the 1001 
test run:
```
    /home/sonyali/logs/perf-test/gce-5000/arktos/1001-pprof-1a0w1e/logs/crashed/kubemarkmaster/log/pprof
```
### Analyze the profiles
A lot info can be found in the golang profiling links above. for Arktos on the server machine:

1. set up the path needed:
```
    export PATH=$PATH:/usr/local/go/bin
    export GOPATH=/home/sonyali/go
    export PPROF_BINARY_PATH=/home/sonyali/go/src/k8s.io/arktos/_output/dockerized/bin/linux/amd64
```
   the source files are copied to the folder already for you for the latest test run, if you see any code
   mismatch with the "list" command, you might need to update the source code in this path:
```
    /go/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/vendor/k8s.io/
```    
2. go to the profile folder
```bash
    cd /home/sonyali/logs/perf-test/gce-5000/arktos/1001-pprof-1a0w1e/logs/crashed/kubemarkmaster/log/pprof/
```    
3. Examples pprof commands
Below are few useful commands to analyze the profile data, i.e. check top resource consumptions, review the annotated code for
details of functions and code related to the resources, comparsion of multiple profiles.

//sonyadev4 is setup to generated the graph of the profile as well, just use "pdf" command to generate the pdf file. then you can copy it over
and review the full picture of it.

For the completed references, please refer to the golang profile doc.   
```bash
root@sonya-uswest2:/home/sonyali/logs/perf-test/gce-5000/arktos/1001-pprof-1a0w1e/logs/crashed/kubemarkmaster/log/pprof/2020-10-01-22:08:40# go tool pprof kube-apiserver-profile-2020-10-01-22:08:40.pprof
File: kube-apiserver
Type: cpu
Time: Oct 1, 2020 at 10:08pm (UTC)
Duration: 30.11s, Total samples = 4.09mins (814.03%)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof) top
Showing nodes accounting for 116.61s, 47.57% of 245.12s total
Dropped 1880 nodes (cum <= 1.23s)
Showing top 10 nodes out of 316
      flat  flat%   sum%        cum   cum%
    37.69s 15.38% 15.38%     37.77s 15.41%  runtime.(*lfstack).pop
    15.82s  6.45% 21.83%     16.39s  6.69%  runtime.mapaccess1_faststr
     9.89s  4.03% 25.86%      9.89s  4.03%  runtime.(*lfstack).push
     9.45s  3.86% 29.72%      9.45s  3.86%  memeqbody
     8.26s  3.37% 33.09%      8.54s  3.48%  syscall.Syscall
     7.35s  3.00% 36.09%      7.35s  3.00%  runtime.epollwait
     7.32s  2.99% 39.07%      7.55s  3.08%  runtime.findObject
     7.25s  2.96% 42.03%      7.25s  2.96%  runtime.futex
     6.92s  2.82% 44.86%      6.92s  2.82%  runtime.memmove
     6.66s  2.72% 47.57%     34.44s 14.05%  k8s.io/kubernetes/vendor/k8s.io/apiserver/pkg/storage/cacher.(*cacheWatcher).convertToWatchEvent

(pprof) list convertToWatchEvent
Total: 4.09mins
ROUTINE ======================== k8s.io/kubernetes/vendor/k8s.io/apiserver/pkg/storage/cacher.(*cacheWatcher).convertToWatchEvent in /go/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/vendor/k8s
.io/apiserver/pkg/storage/cacher/cacher.go
     6.66s     34.44s (flat, cum) 14.05% of Total
         .          .   1138:           return c.deadline, false
         .          .   1139:   }
         .          .   1140:   return c.deadline.Add(-2 * time.Second), true
         .          .   1141:}
         .          .   1142:
      30ms       30ms   1143:func (c *cacheWatcher) convertToWatchEvent(event *watchCacheEvent) *watch.Event {
     5.67s      5.67s   1144:   if event.Type == watch.Bookmark {
         .          .   1145:           return &watch.Event{Type: watch.Bookmark, Object: event.Object.DeepCopyObject()}
         .          .   1146:   }
         .          .   1147:
     870ms     23.64s   1148:   curObjPasses := event.Type != watch.Deleted && c.filter(event.Key, event.ObjLabels, event.ObjFields)
         .          .   1149:   oldObjPasses := false
      10ms       10ms   1150:   if event.PrevObject != nil {
      60ms      3.04s   1151:           oldObjPasses = c.filter(event.Key, event.PrevObjLabels, event.PrevObjFields)
         .          .   1152:   }
         .          .   1153:   if !curObjPasses && !oldObjPasses {
         .          .   1154:           // Watcher is not interested in that object.
      10ms       10ms   1155:           return nil
         .          .   1156:   }
         .          .   1157:
         .          .   1158:   switch {
         .          .   1159:   case curObjPasses && !oldObjPasses:
         .       10ms   1160:           return &watch.Event{Type: watch.Added, Object: event.Object.DeepCopyObject()}
         .          .   1161:   case curObjPasses && oldObjPasses:
         .      2.02s   1162:           return &watch.Event{Type: watch.Modified, Object: event.Object.DeepCopyObject()}
      10ms       10ms   1163:   case !curObjPasses && oldObjPasses:
         .          .   1164:           // return a delete event with the previous object content, but with the event's resource version
         .          .   1165:           oldObj := event.PrevObject.DeepCopyObject()
         .          .   1166:           if err := c.versioner.UpdateObject(oldObj, event.ResourceVersion); err != nil {
         .          .   1167:                   utilruntime.HandleError(fmt.Errorf("failure to version api object (%d) %#v: %v", event.ResourceVersion, oldObj, err))
         .          .   1168:           }


root@sonya-uswest2:/home/sonyali/logs/perf-test/gce-5000/arktos/1001-pprof-1a0w1e/logs/crashed/kubemarkmaster/log/pprof/2020-10-01-22:08:40# go tool pprof --base ../2020-10-01-21:06:33/kube-apiserver-heap-2020
-10-01-21:06:33.pprof kube-apiserver-heap-2020-10-01-22:08:40.pprof
File: kube-apiserver
Type: inuse_space
Time: Oct 1, 2020 at 9:07pm (UTC)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof) top
Showing nodes accounting for 2671.17MB, 44.78% of 5964.48MB total
Dropped 906 nodes (cum <= 29.82MB)
Showing top 10 nodes out of 255
      flat  flat%   sum%        cum   cum%
  817.98MB 13.71% 13.71%   817.98MB 13.71%  k8s.io/kubernetes/vendor/k8s.io/apiserver/pkg/storage/cacher.newCacheWatcher
  788.79MB 13.22% 26.94%   788.79MB 13.22%  runtime.malg
  187.52MB  3.14% 30.08%   187.52MB  3.14%  k8s.io/kubernetes/vendor/github.com/grafov/bcast.NewGroup
  171.14MB  2.87% 32.95%   171.14MB  2.87%  reflect.New
  157.09MB  2.63% 35.59%   159.09MB  2.67%  k8s.io/kubernetes/vendor/k8s.io/api/core/v1.(*ResourceRequirements).Unmarshal
  152.57MB  2.56% 38.14%   152.57MB  2.56%  math/big.basicSqr
  109.02MB  1.83% 39.97%   118.02MB  1.98%  k8s.io/kubernetes/vendor/k8s.io/apimachinery/pkg/apis/meta/v1.(*ObjectMeta).Unmarshal
  102.39MB  1.72% 41.69%   102.39MB  1.72%  bufio.NewWriterSize
   93.16MB  1.56% 43.25%    93.16MB  1.56%  regexp/syntax.(*compiler).inst
   91.52MB  1.53% 44.78%   306.11MB  5.13%  k8s.io/kubernetes/vendor/k8s.io/api/core/v1.(*PodSpec).Unmarshal
(pprof) list ListResource.func1
Total: 5.82GB
ROUTINE ======================== k8s.io/kubernetes/vendor/k8s.io/apiserver/pkg/endpoints.restfulListResource.func1 in /go/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/vendor/k8s.io/apiserv
er/pkg/endpoints/installer.go
         0     1.39GB (flat, cum) 23.80% of Total
         .          .   1250:   return false
         .          .   1251:}
         .          .   1252:
         .          .   1253:func restfulListResource(r rest.Lister, rw rest.Watcher, scope handlers.RequestScope, forceWatch bool, minRequestTimeout time.Duration) restful.RouteFunction {
         .          .   1254:   return func(req *restful.Request, res *restful.Response) {
         .     1.39GB   1255:           handlers.ListResource(r, rw, &scope, forceWatch, minRequestTimeout)(res.ResponseWriter, req.Request)
         .          .   1256:   }
         .          .   1257:}
         .          .   1258:
         .          .   1259:func restfulCreateNamedResource(r rest.NamedCreater, scope handlers.RequestScope, admit admission.Interface) restful.RouteFunction {
         .          .   1260:   return func(req *restful.Request, res *restful.Response) {
```
