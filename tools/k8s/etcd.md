etcd
===

## etcd节点损坏
etcd某节点宕机后，如果此节点上的数据没有损坏，则此节点重新启动可以继续加入集群。如果节点上的数据损坏了，则加入集群会是失败。失败日志如下：
```
etcdmain: member c525f8002b25504d has already been bootstrapped
```

1. 解决此问题，需要先在集群中将当前节点删除
```shell
ETCDCTL_API=3 /tmp/etcd-download-test/etcdctl --endpoints=http://10.103.221.0:2379 --user=test:test123 member remove c525f8002b25504d
Member c525f8002b25504d removed from cluster 8ec2a50af9708b3c
```

2. 如果是k8s中部署的，则先将pods和deployment删除
```shell
kubectl delete deployment etcd-02
kubectl delete pods etcd-02-7cc8bc9776-t4v2g
```

3. 然后添加节点
```shell
ETCDCTL_API=3 /tmp/etcd-download-test/etcdctl --endpoints=http://10.103.221.0:2379 --user=test:test123 member add etcd02 --peer-urls=http://svc-etcd-02:2380
Member b3c536ef77577469 added to cluster 8ec2a50af9708b3c

ETCD_NAME="etcd02"
ETCD_INITIAL_CLUSTER="etcd03=http://svc-etcd-03:2380,etcd02=http://svc-etcd-02:2380,etcd01=http://svc-etcd-01:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://svc-etcd-02:2380"
ETCD_INITIAL_CLUSTER_STATE="existing"
```

4. 重新启动etcd02集群，在重新启动集群时需要设置ETCD_INITIAL_CLUSTER_STATE="existing"