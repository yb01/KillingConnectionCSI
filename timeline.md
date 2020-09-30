

Common Test Setup (specify change in the Backup column)

- 5000 nodes

- Load test only

- WCM disabled

- Perf-test tool: Arktos or community?

- ETCD 3.4.3

- Leader-election disabled

- Coredump enabled

- insecure-port enabled

  



| Build Date | Commit | Runner | Result                                                       | Analysis | Backup |
| ---------- | ------ | ------ | ------------------------------------------------------------ | -------- | ------ |
|            |        |        |                                                              |          |        |
|            |        |        |                                                              |          |        |
|            |        |        |                                                              |          |        |
|            | 9c050  | Joe    | Many  Kube-apiserver panics. “killing connection/stream”     |          |        |
| 09/24      | 25120  | Sonya  | Kube-apiserver Panic: killing connection/stream. Perf-tests tool panic: runtime error: invalid memory address or nil pointer dereference |          |        |

