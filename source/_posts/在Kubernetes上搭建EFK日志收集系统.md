---
title: 在 Kubernetes 上搭建 EFK 日志收集系统
date: 2019-11-06 18:54:02
categories: 持续集成
tags: [k8s, EFK]
---

Kubernetes 中比较流行的日志收集解决方案是 `Elasticsearch`、`Fluentd` 和 `Kibana`（EFK）技术栈，也是官方现在比较推荐的一种方案。

`Elasticsearch` 是一个实时的、分布式的可扩展的搜索引擎，允许进行全文、结构化搜索，它通常用于索引和搜索大量日志数据，也可用于搜索许多不同类型的文档。

Elasticsearch 通常与 `Kibana` 一起部署，Kibana 是 Elasticsearch 的一个功能强大的数据可视化 Dashboard，Kibana 允许你通过 web 界面来浏览 Elasticsearch 日志数据。

`Fluentd`是一个流行的开源数据收集器，我们将在 Kubernetes 集群节点上安装 Fluentd，通过获取容器日志文件、过滤和转换日志数据，然后将数据传递到 Elasticsearch 集群，在该集群中对其进行索引和存储。

我们先来配置启动一个可扩展的 Elasticsearch 集群，然后在 Kubernetes 集群中创建一个 Kibana 应用，最后通过 DaemonSet 来运行 Fluentd，以便它在每个 Kubernetes 工作节点上都可以运行一个 Pod。

![img](在Kubernetes上搭建EFK日志收集系统/20180511161547422)

## 部署nfs-client

### 创建 Provisioner

要使用 StorageClass，我们就得安装对应的自动配置程序，比如我们这里存储后端使用的是 nfs，那么我们就需要使用到一个 nfs-client 的自动配置程序，我们也叫它 Provisioner，这个程序使用我们已经配置好的 nfs 服务器，来自动创建持久卷，也就是自动帮我们创建 PV。

- 自动创建的 PV 以`${namespace}-${pvcName}-${pvName}`这样的命名格式创建在 NFS 服务器上的共享数据目录中
- 而当这个 PV 被回收后会以`archieved-${namespace}-${pvcName}-${pvName}`这样的命名格式存在 NFS 服务器上。

当然在部署`nfs-client`之前，我们需要先成功安装上 nfs 服务器，前面的课程中我们已经过了，服务地址是**10.151.30.57**，共享数据目录是**/data/k8s/**，然后接下来我们部署 nfs-client 即可，我们也可以直接参考[nfs-client 的文档](https://github.com/kubernetes-incubator/external-storage/tree/master/nfs-client)，进行安装即可。

**第一步**：配置 Deployment，将里面的对应的参数替换成我们自己的 nfs 配置（nfs-client.yaml）

```yaml
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: nfs-client-provisioner
spec:
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: quay.io/external_storage/nfs-client-provisioner:latest
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: fuseim.pri/ifs
            - name: NFS_SERVER
              value: 10.151.30.57
            - name: NFS_PATH
              value: /data/k8s
      volumes:
        - name: nfs-client-root
          nfs:
            server: 10.151.30.57
            path: /data/k8s
```

**第二步**：将环境变量 NFS_SERVER 和 NFS_PATH 替换，当然也包括下面的 nfs 配置，我们可以看到我们这里使用了一个名为 nfs-client-provisioner 的`serviceAccount`，所以我们也需要创建一个 sa，然后绑定上对应的权限：（nfs-client-sa.yaml）

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner

---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["list", "watch", "create", "update", "patch"]
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["create", "delete", "get", "list", "watch", "patch", "update"]

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    namespace: default
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
```

我们这里新建的一个名为 nfs-client-provisioner 的`ServiceAccount`，然后绑定了一个名为 nfs-client-provisioner-runner 的`ClusterRole`，而该`ClusterRole`声明了一些权限，其中就包括对`persistentvolumes`的增、删、改、查等权限，所以我们可以利用该`ServiceAccount`来自动创建 PV。

**第三步**：nfs-client 的 Deployment 声明完成后，我们就可以来创建一个`StorageClass`对象了：（nfs-client-class.yaml）

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: course-nfs-storage
provisioner: fuseim.pri/ifs # or choose another name, must match deployment's env PROVISIONER_NAME'
```

我们声明了一个名为 course-nfs-storage 的`StorageClass`对象，注意下面的`provisioner`对应的值一定要和上面的`Deployment`下面的 PROVISIONER_NAME 这个环境变量的值一样。

现在我们来创建这些资源对象吧：

```shell
$ kubectl create -f nfs-client.yaml
$ kubectl create -f nfs-client-sa.yaml
$ kubectl create -f nfs-client-class.yaml
```

创建完成后查看下资源状态：

```
$ kubectl get pods
NAME                                             READY     STATUS             RESTARTS   AGE
...
nfs-client-provisioner-7648b664bc-7f9pk          1/1       Running            0          7h
...
$ kubectl get storageclass
NAME                 PROVISIONER      AGE
course-nfs-storage   fuseim.pri/ifs   11s
```

### 新建 PVC

上面把`StorageClass`资源对象创建成功了，接下来我们来通过一个示例测试下动态 PV，首先创建一个 PVC 对象：(test-pvc.yaml)

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Mi
```

我们这里声明了一个`PVC`对象，采用 `ReadWriteMany` 的访问模式，请求 1Mi 的空间，但是我们可以看到上面的 PVC 文件我们没有标识出任何和 StorageClass 相关联的信息，那么如果我们现在直接创建这个 PVC 对象能够自动绑定上合适的 PV 对象吗？显然是不能的(前提是没有合适的 PV)，我们这里有两种方法可以来利用上面我们创建的 StorageClass 对象来自动帮我们创建一个合适的 PV:

- 第一种方法：在这个`PVC`对象中添加一个声明`StorageClass`对象的标识，这里我们可以利用一个`annotations`属性来标识，如下

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-pvc
  annotations:
    volume.beta.kubernetes.io/storage-class: "course-nfs-storage"
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Mi
```

- 第二种方法：我们可以设置这个 course-nfs-storage 的 StorageClass 为 Kubernetes 的默认存储后端，我们可以用`kubectl patch`命令来更新：

  ```yaml
  $ kubectl patch storageclass course-nfs-storage -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
  ```

  上面这两种方法都是可以的，当然为了不影响系统的默认行为，我们这里还是采用第一种方法，直接创建即可：

  ```yaml
  $ kubectl create -f test-pvc.yaml
  persistentvolumeclaim "test-pvc" created
  $ kubectl get pvc
  NAME         STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS          AGE
  ...
  test-pvc     Bound     pvc-73b5ffd2-8b4b-11e8-b585-525400db4df7   1Mi        RWX            course-nfs-storage    2m
  ...
  ```

我们可以看到一个名为 test-pvc 的 PVC 对象创建成功了，状态已经是`Bound`了，是不是也产生了一个对应的`VOLUME` 对象，最重要的一栏是`STORAGECLASS`，现在是不是也有值了，就是我们刚刚创建的`StorageClass`对象 course-nfs-storage。

然后查看下 PV 对象呢：

```shell
$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                STORAGECLASS          REASON    AGE
...
pvc-73b5ffd2-8b4b-11e8-b585-525400db4df7   1Mi        RWX            Delete           Bound       default/test-pvc     course-nfs-storage              8m
...
```

可以看到是不是自动生成了一个关联的 PV 对象，访问模式是`RWX`，回收策略是 `Delete`，这个 PV 对象并不是我们手动创建的吧，这是通过我们上面的 `StorageClass` 对象自动创建的。这就是 StorageClass 的创建方法。

### 测试

接下来我们还是用一个简单的示例来测试下我们上面用 StorageClass 方式声明的 PVC 对象吧：(test-pod.yaml)

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: test-pod
spec:
  containers:
  - name: test-pod
    image: busybox
    imagePullPolicy: IfNotPresent
    command:
    - "/bin/sh"
    args:
    - "-c"
    - "touch /mnt/SUCCESS && exit 0 || exit 1"
    volumeMounts:
    - name: nfs-pvc
      mountPath: "/mnt"
  restartPolicy: "Never"
  volumes:
  - name: nfs-pvc
    persistentVolumeClaim:
      claimName: test-pvc
```

上面这个 Pod 非常简单，就是用一个 **busybox** 容器，在 /mnt 目录下面新建一个 SUCCESS 的文件，然后把 /mnt 目录挂载到上面我们新建的 test-pvc 这个资源对象上面了，要验证很简单，只需要去查看下我们 nfs 服务器上面的共享数据目录下面是否有 SUCCESS 这个文件即可：

```shell
$ kubectl create -f test-pod.yaml
pod "test-pod" created
```

然后我们可以在 nfs 服务器的共享数据目录下面查看下数据：

```shell
$ ls /data/k8s/
default-test-pvc-pvc-73b5ffd2-8b4b-11e8-b585-525400db4df7
```

我们可以看到下面有名字很长的文件夹，这个文件夹的命名方式是不是和我们上面的规则：**${namespace}-${pvcName}-${pvName}**是一样的，再看下这个文件夹下面是否有其他文件：

```shell
$ ls /data/k8s/default-test-pvc-pvc-73b5ffd2-8b4b-11e8-b585-525400db4df7
SUCCESS
```

我们看到下面有一个 SUCCESS 的文件，是不是就证明我们上面的验证是成功的啊。

另外我们可以看到我们这里是手动创建的一个 PVC 对象，在实际工作中，使用 StorageClass 更多的是 StatefulSet 类型的服务，`StatefulSet`类型的服务我们也可以通过一个`volumeClaimTemplates`属性来直接使用 StorageClass，如下：(test-statefulset-nfs.yaml)

```yaml
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  name: nfs-web
spec:
  serviceName: "nginx"
  replicas: 3
  template:
    metadata:
      labels:
        app: nfs-web
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
      annotations:
        volume.beta.kubernetes.io/storage-class: course-nfs-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

实际上 volumeClaimTemplates 下面就是一个 PVC 对象的模板，就类似于我们这里 StatefulSet 下面的 template，实际上就是一个 Pod 的模板，我们不单独创建成 PVC 对象，而用这种模板就可以动态的去创建了对象了，这种方式在 StatefulSet 类型的服务下面使用得非常多。

直接创建上面的对象：

```shell
$ kubectl create -f test-statefulset-nfs.yaml
statefulset.apps "nfs-web" created
$ kubectl get pods
NAME                                             READY     STATUS              RESTARTS   AGE
...
nfs-web-0                                        1/1       Running             0          1m
nfs-web-1                                        1/1       Running             0          1m
nfs-web-2                                        1/1       Running             0          33s
...
```

创建完成后可以看到上面的3个 Pod 已经运行成功，然后查看下 PVC 对象：

```shell
$ kubectl get pvc
NAME            STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS          AGE
...
www-nfs-web-0   Bound     pvc-cc36b3ce-8b50-11e8-b585-525400db4df7   1Gi        RWO            course-nfs-storage    2m
www-nfs-web-1   Bound     pvc-d38285f9-8b50-11e8-b585-525400db4df7   1Gi        RWO            course-nfs-storage    2m
www-nfs-web-2   Bound     pvc-e348250b-8b50-11e8-b585-525400db4df7   1Gi        RWO            course-nfs-storage    1m
...
```

我们可以看到是不是也生成了3个 PVC 对象，名称由模板名称 name 加上 Pod 的名称组合而成，这3个 PVC 对象也都是 绑定状态了，很显然我们查看 PV 也可以看到对应的3个 PV 对象：

```shell
$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                   STORAGECLASS          REASON    AGE
...                                                        1d
pvc-cc36b3ce-8b50-11e8-b585-525400db4df7   1Gi        RWO            Delete           Bound       default/www-nfs-web-0   course-nfs-storage              4m
pvc-d38285f9-8b50-11e8-b585-525400db4df7   1Gi        RWO            Delete           Bound       default/www-nfs-web-1   course-nfs-storage              4m
pvc-e348250b-8b50-11e8-b585-525400db4df7   1Gi        RWO            Delete           Bound       default/www-nfs-web-2   course-nfs-storage              4m
...
```

查看 nfs 服务器上面的共享数据目录：

```shell
$ ls /data/k8s/
...
default-www-nfs-web-0-pvc-cc36b3ce-8b50-11e8-b585-525400db4df7
default-www-nfs-web-1-pvc-d38285f9-8b50-11e8-b585-525400db4df7
default-www-nfs-web-2-pvc-e348250b-8b50-11e8-b585-525400db4df7
...
```

是不是也有对应的3个数据目录，这就是我们的 StorageClass 的使用方法，对于 StorageClass 多用于 StatefulSet 类型的服务，在后面的课程中我们还学不断的接触到。

### 参考

<https://www.qikqiak.com/post/kubernetes-persistent-volume2/>



## 创建 Elasticsearch 集群

在创建 Elasticsearch 集群之前，我们先创建一个命名空间，我们将在其中安装所有日志相关的资源对象。

新建一个 kube-logging.yaml 文件：

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: logging
```

然后通过 kubectl 创建该资源清单，创建一个名为 logging 的 namespace：

```shell
$ kubectl create -f kube-logging.yaml
namespace/logging created
$ kubectl get ns
NAME           STATUS    AGE
default        Active    244d
istio-system   Active    100d
kube-ops       Active    179d
kube-public    Active    244d
kube-system    Active    244d
logging        Active    4h
monitoring     Active    35d
```

现在创建了一个命名空间来存放我们的日志相关资源，接下来可以部署 EFK 相关组件，首先开始部署一个3节点的 Elasticsearch 集群。

这里我们使用3个 Elasticsearch Pod 来避免高可用下多节点集群中出现的“脑裂”问题，当一个或多个节点无法与其他节点通信时会产生“脑裂”，可能会出现几个主节点。

> 了解更多 Elasticsearch 集群脑裂问题，可以查看文档<https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#split-brain>

一个关键点是您应该设置参数`discover.zen.minimum_master_nodes=N/2+1`，其中`N`是 Elasticsearch 集群中符合主节点的节点数，比如我们这里3个节点，意味着`N`应该设置为2。这样，如果一个节点暂时与集群断开连接，则另外两个节点可以选择一个新的主节点，并且集群可以在最后一个节点尝试重新加入时继续运行，在扩展 Elasticsearch 集群时，一定要记住这个参数。

首先创建一个名为 elasticsearch 的无头服务，新建文件 elasticsearch-svc.yaml，文件内容如下：

```yaml
kind: Service
apiVersion: v1
metadata:
  name: elasticsearch
  namespace: logging
  labels:
    app: elasticsearch
spec:
  selector:
    app: elasticsearch
  clusterIP: None
  ports:
    - port: 9200
      name: rest
    - port: 9300
      name: inter-node
```

定义了一个名为 elasticsearch 的 Service，指定标签`app=elasticsearch`，当我们将 Elasticsearch StatefulSet 与此服务关联时，服务将返回带有标签`app=elasticsearch`的 Elasticsearch Pods 的 DNS A 记录，然后设置`clusterIP=None`，将该服务设置成无头服务。最后，我们分别定义端口9200、9300，分别用于与 REST API 交互，以及用于节点间通信。

使用 kubectl 直接创建上面的服务资源对象：

```shell
$ kubectl create -f elasticsearch-svc.yaml
service/elasticsearch created
$ kubectl get services --namespace=logging
Output
NAME            TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)             AGE
elasticsearch   ClusterIP   None         <none>        9200/TCP,9300/TCP   26s
```

现在我们已经为 Pod 设置了无头服务和一个稳定的域名`.elasticsearch.logging.svc.cluster.local`，接下来我们通过 StatefulSet 来创建具体的 Elasticsearch 的 Pod 应用。

Kubernetes StatefulSet 允许我们为 Pod 分配一个稳定的标识和持久化存储，Elasticsearch 需要稳定的存储来保证 Pod 在重新调度或者重启后的数据依然不变，所以需要使用 StatefulSet 来管理 Pod。

> 要了解更多关于 StaefulSet 的信息，可以查看官网关于 StatefulSet 的相关文档：<https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/>。

新建名为 elasticsearch-statefulset.yaml 的资源清单文件，首先粘贴下面内容：

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: es-cluster
  namespace: logging
spec:
  serviceName: elasticsearch
  replicas: 3
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
```

该内容中，我们定义了一个名为 es-cluster 的 StatefulSet 对象，然后定义`serviceName=elasticsearch`和前面创建的 Service 相关联，这可以确保使用以下 DNS 地址访问 StatefulSet 中的每一个 Pod：`es-cluster-[0,1,2].elasticsearch.logging.svc.cluster.local`，其中[0,1,2]对应于已分配的 Pod 序号。

然后指定3个副本，将 matchLabels 设置为`app=elasticsearch`，所以 Pod 的模板部分`.spec.template.metadata.lables`也必须包含`app=elasticsearch`标签。

然后定义 Pod 模板部分内容：

```yaml
...
  spec:
    containers:
    - name: elasticsearch
      image: docker.elastic.co/elasticsearch/elasticsearch-oss:6.4.3
      resources:
        limits:
          cpu: 1000m
        requests:
          cpu: 100m
      ports:
      - containerPort: 9200
        name: rest
        protocol: TCP
      - containerPort: 9300
        name: inter-node
        protocol: TCP
      volumeMounts:
      - name: data
        mountPath: /usr/share/elasticsearch/data
      env:
      - name: cluster.name
        value: k8s-logs
      - name: node.name
        valueFrom:
          fieldRef:
            fieldPath: metadata.name
      - name: discovery.zen.ping.unicast.hosts
        value: "es-cluster-0.elasticsearch,es-cluster-1.elasticsearch,es-cluster-2.elasticsearch"
      - name: discovery.zen.minimum_master_nodes
        value: "2"
      - name: ES_JAVA_OPTS
        value: "-Xms512m -Xmx512m"
```

该部分是定义 StatefulSet 中的 Pod，我们这里使用一个`-oss`后缀的镜像，该镜像是 Elasticsearch 的开源版本，如果你想使用包含`X-Pack`之类的版本，可以去掉该后缀。然后暴露了9200和9300两个端口，注意名称要和上面定义的 Service 保持一致。然后通过 volumeMount 声明了数据持久化目录，下面我们再来定义 VolumeClaims。最后就是我们在容器中设置的一些环境变量了：

- cluster.name：Elasticsearch 集群的名称，我们这里命名成 k8s-logs。
- node.name：节点的名称，通过`metadata.name`来获取。这将解析为 es-cluster-[0,1,2]，取决于节点的指定顺序。
- discovery.zen.ping.unicast.hosts：此字段用于设置在 Elasticsearch 集群中节点相互连接的发现方法。我们使用 unicastdiscovery 方式，它为我们的集群指定了一个静态主机列表。由于我们之前配置的无头服务，我们的 Pod 具有唯一的 DNS 域`es-cluster-[0,1,2].elasticsearch.logging.svc.cluster.local`，因此我们相应地设置此变量。由于都在同一个 namespace 下面，所以我们可以将其缩短为`es-cluster-[0,1,2].elasticsearch`。要了解有关 Elasticsearch 发现的更多信息，请参阅 Elasticsearch 官方文档：<https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery.html>。
- discovery.zen.minimum_master_nodes：我们将其设置为`(N/2) + 1`，`N`是我们的群集中符合主节点的节点的数量。我们有3个 Elasticsearch 节点，因此我们将此值设置为2（向下舍入到最接近的整数）。要了解有关此参数的更多信息，请参阅官方 Elasticsearch 文档：<https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#split-brain>。
- ES_JAVA_OPTS：这里我们设置为`-Xms512m -Xmx512m`，告诉`JVM`使用`512 MB`的最小和最大堆。您应该根据群集的资源可用性和需求调整这些参数。要了解更多信息，请参阅设置堆大小的相关文档：<https://www.elastic.co/guide/en/elasticsearch/reference/current/heap-size.html>。

接下来添加关于 initContainer 的内容：

```yaml
...
    initContainers:
    - name: fix-permissions
      image: busybox
      command: ["sh", "-c", "chown -R 1000:1000 /usr/share/elasticsearch/data"]
      securityContext:
        privileged: true
      volumeMounts:
      - name: data
        mountPath: /usr/share/elasticsearch/data
    - name: increase-vm-max-map
      image: busybox
      command: ["sysctl", "-w", "vm.max_map_count=262144"]
      securityContext:
        privileged: true
    - name: increase-fd-ulimit
      image: busybox
      command: ["sh", "-c", "ulimit -n 65536"]
      securityContext:
        privileged: true
```

这里我们定义了几个在主应用程序之前运行的 Init 容器，这些初始容器按照定义的顺序依次执行，执行完成后才会启动主应用容器。

第一个名为 fix-permissions 的容器用来运行 chown 命令，将 Elasticsearch 数据目录的用户和组更改为`1000:1000`（Elasticsearch 用户的 UID）。因为默认情况下，Kubernetes 用 root 用户挂载数据目录，这会使得 Elasticsearch 无法方法该数据目录，可以参考 Elasticsearch 生产中的一些默认注意事项相关文档说明：<https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#_notes_for_production_use_and_defaults>。

第二个名为 increase-vm-max-map 的容器用来增加操作系统对`mmap`计数的限制，默认情况下该值可能太低，导致内存不足的错误，要了解更多关于该设置的信息，可以查看 Elasticsearch 官方文档说明：<https://www.elastic.co/guide/en/elasticsearch/reference/current/vm-max-map-count.html>。

最后一个初始化容器是用来执行`ulimit`命令增加打开文件描述符的最大数量的。

> 此外 [Elastisearch Notes for Production Use](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#_notes_for_production_use_and_defaults) 文档还提到了由于性能原因最好禁用 swap，当然对于 Kubernetes 集群而言，最好也是禁用 swap 分区的。

现在我们已经定义了主应用容器和它之前运行的 Init Containers 来调整一些必要的系统参数，接下来我们可以添加数据目录的持久化相关的配置，在 StatefulSet 中，使用 volumeClaimTemplates 来定义 volume 模板即可：

```yaml
...
  volumeClaimTemplates:
  - metadata:
      name: data
      labels:
        app: elasticsearch
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: es-data-db
      resources:
        requests:
          storage: 50Gi
```

我们这里使用 volumeClaimTemplates 来定义持久化模板，Kubernetes 会使用它为 Pod 创建 PersistentVolume，设置访问模式为`ReadWriteOnce`，这意味着它只能被 mount 到单个节点上进行读写，然后最重要的是使用了一个名为 es-data-db 的 StorageClass 对象，所以我们需要提前创建该对象，我们这里使用的 NFS 作为存储后端，所以需要安装一个对应的 provisioner 驱动，前面关于 StorageClass 的课程中已经和大家介绍过方法，新建一个 elasticsearch-storageclass.yaml 的文件，文件内容如下：

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: es-data-db
provisioner: fuseim.pri/ifs  # 该值需要和 provisioner 配置的保持一致
```

最后，我们指定了每个 PersistentVolume 的大小为 50GB，我们可以根据自己的实际需要进行调整该值。最后，完整的 Elasticsearch StatefulSet 资源清单文件内容如下：

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: es-cluster
  namespace: logging
spec:
  serviceName: elasticsearch
  replicas: 3
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      containers:
      - name: elasticsearch
        image: docker.elastic.co/elasticsearch/elasticsearch-oss:6.4.3
        resources:
            limits:
              cpu: 1000m
            requests:
              cpu: 100m
        ports:
        - containerPort: 9200
          name: rest
          protocol: TCP
        - containerPort: 9300
          name: inter-node
          protocol: TCP
        volumeMounts:
        - name: data
          mountPath: /usr/share/elasticsearch/data
        env:
          - name: cluster.name
            value: k8s-logs
          - name: node.name
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: discovery.zen.ping.unicast.hosts
            value: "es-cluster-0.elasticsearch,es-cluster-1.elasticsearch,es-cluster-2.elasticsearch"
          - name: discovery.zen.minimum_master_nodes
            value: "2"
          - name: ES_JAVA_OPTS
            value: "-Xms512m -Xmx512m"
      initContainers:
      - name: fix-permissions
        image: busybox
        command: ["sh", "-c", "chown -R 1000:1000 /usr/share/elasticsearch/data"]
        securityContext:
          privileged: true
        volumeMounts:
        - name: data
          mountPath: /usr/share/elasticsearch/data
      - name: increase-vm-max-map
        image: busybox
        command: ["sysctl", "-w", "vm.max_map_count=262144"]
        securityContext:
          privileged: true
      - name: increase-fd-ulimit
        image: busybox
        command: ["sh", "-c", "ulimit -n 65536"]
        securityContext:
          privileged: true
  volumeClaimTemplates:
  - metadata:
      name: data
      labels:
        app: elasticsearch
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: es-data-db
      resources:
        requests:
          storage: 100Gi
```

现在直接使用 kubectl 工具部署即可：

```shell
$ kubectl create -f elasticsearch-storageclass.yaml
storageclass.storage.k8s.io "es-data-db" created
$ kubectl create -f elasticsearch-statefulset.yaml
statefulset.apps/es-cluster created
```

添加成功后，可以看到 logging 命名空间下面的所有的资源对象：

```shell
$ kubectl get sts -n logging
NAME         DESIRED   CURRENT   AGE
es-cluster   3         3         20h
$ kubectl get pods -n logging
NAME                      READY     STATUS    RESTARTS   AGE
es-cluster-0              1/1       Running   0          20h
es-cluster-1              1/1       Running   0          20h
es-cluster-2              1/1       Running   0          20h
$ kubectl get svc -n logging
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
elasticsearch   ClusterIP   None             <none>        9200/TCP,9300/TCP   20h
```

Pods 部署完成后，我们可以通过请求一个 REST API 来检查 Elasticsearch 集群是否正常运行。使用下面的命令将本地端口9200转发到 Elasticsearch 节点（如es-cluster-0）对应的端口：

```shell
$ kubectl port-forward es-cluster-0 9200:9200 --namespace=logging
Forwarding from 127.0.0.1:9200 -> 9200
Forwarding from [::1]:9200 -> 9200
```

然后，在另外的终端窗口中，执行如下请求：

```shell
$ curl http://localhost:9200/_cluster/state?pretty
```

正常来说，应该会看到类似于如下的信息：

```shell
{
  "cluster_name" : "k8s-logs",
  "compressed_size_in_bytes" : 348,
  "cluster_uuid" : "QD06dK7CQgids-GQZooNVw",
  "version" : 3,
  "state_uuid" : "mjNIWXAzQVuxNNOQ7xR-qg",
  "master_node" : "IdM5B7cUQWqFgIHXBp0JDg",
  "blocks" : { },
  "nodes" : {
    "u7DoTpMmSCixOoictzHItA" : {
      "name" : "es-cluster-1",
      "ephemeral_id" : "ZlBflnXKRMC4RvEACHIVdg",
      "transport_address" : "10.244.4.191:9300",
      "attributes" : { }
    },
    "IdM5B7cUQWqFgIHXBp0JDg" : {
      "name" : "es-cluster-0",
      "ephemeral_id" : "JTk1FDdFQuWbSFAtBxdxAQ",
      "transport_address" : "10.244.2.215:9300",
      "attributes" : { }
    },
    "R8E7xcSUSbGbgrhAdyAKmQ" : {
      "name" : "es-cluster-2",
      "ephemeral_id" : "9wv6ke71Qqy9vk2LgJTqaA",
      "transport_address" : "10.244.40.4:9300",
      "attributes" : { }
    }
  },
...
```

看到上面的信息就表明我们名为 k8s-logs 的 Elasticsearch 集群成功创建了3个节点：es-cluster-0，es-cluster-1，和es-cluster-2，当前主节点是 es-cluster-0。

## 创建 Kibana 服务

Elasticsearch 集群启动成功了，接下来我们可以来部署 Kibana 服务，新建一个名为 kibana.yaml 的文件，对应的文件内容如下：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kibana
  namespace: logging
  labels:
    app: kibana
spec:
  ports:
  - port: 5601
  type: NodePort
  selector:
    app: kibana

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kibana
  namespace: logging
  labels:
    app: kibana
spec:
  selector:
    matchLabels:
      app: kibana
  template:
    metadata:
      labels:
        app: kibana
    spec:
      containers:
      - name: kibana
        image: docker.elastic.co/kibana/kibana-oss:6.4.3
        resources:
          limits:
            cpu: 1000m
          requests:
            cpu: 100m
        env:
          - name: ELASTICSEARCH_URL
            value: http://elasticsearch:9200
        ports:
        - containerPort: 5601
```

上面我们定义了两个资源对象，一个 Service 和 Deployment，为了测试方便，我们将 Service 设置为了 NodePort 类型，Kibana Pod 中配置都比较简单，唯一需要注意的是我们使用 ELASTICSEARCH_URL 这个环境变量来设置Elasticsearch 集群的端点和端口，直接使用 Kubernetes DNS 即可，此端点对应服务名称为 elasticsearch，由于是一个 headless service，所以该域将解析为3个 Elasticsearch Pod 的 IP 地址列表。

配置完成后，直接使用 kubectl 工具创建：

```shell
$ kubectl create -f kibana.yaml
service/kibana created
deployment.apps/kibana created
```

创建完成后，可以查看 Kibana Pod 的运行状态：

```shell
$ kubectl get pods --namespace=logging
NAME                      READY     STATUS    RESTARTS   AGE
es-cluster-0              1/1       Running   0          20h
es-cluster-1              1/1       Running   0          20h
es-cluster-2              1/1       Running   0          20h
kibana-7558d4dc4d-5mqdz   1/1       Running   0          20h
$ kubectl get svc --namespace=logging
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
elasticsearch   ClusterIP   None             <none>        9200/TCP,9300/TCP   20h
kibana          NodePort    10.105.208.253   <none>        5601:31816/TCP      20h
```

如果 Pod 已经是 Running 状态了，证明应用已经部署成功了，然后可以通过 NodePort 来访问 Kibana 这个服务，在浏览器中打开`http://<任意节点IP>:31816`即可，如果看到如下欢迎界面证明 Kibana 已经成功部署到了 Kubernetes集群之中。

![kibana welcome](在Kubernetes上搭建EFK日志收集系统/yBp0Hl.jpg)kibana welcome

## 部署 Fluentd

`Fluentd` 是一个高效的日志聚合器，是用 Ruby 编写的，并且可以很好地扩展。对于大部分企业来说，Fluentd 足够高效并且消耗的资源相对较少，另外一个工具`Fluent-bit`更轻量级，占用资源更少，但是插件相对 Fluentd 来说不够丰富，所以整体来说，Fluentd 更加成熟，使用更加广泛，所以我们这里也同样使用 Fluentd 来作为日志收集工具。

### 工作原理

Fluentd 通过一组给定的数据源抓取日志数据，处理后（转换成结构化的数据格式）将它们转发给其他服务，比如 Elasticsearch、对象存储等等。Fluentd 支持超过300个日志存储和分析服务，所以在这方面是非常灵活的。主要运行步骤如下：

- 首先 Fluentd 从多个日志源获取数据
- 结构化并且标记这些数据
- 然后根据匹配的标签将数据发送到多个目标服务去

![fluentd 架构](在Kubernetes上搭建EFK日志收集系统/lfDrcH.jpg)fluentd 架构

### 配置

一般来说我们是通过一个配置文件来告诉 Fluentd 如何采集、处理数据的，下面简单和大家介绍下 Fluentd 的配置方法。

#### 日志源配置

比如我们这里为了收集 Kubernetes 节点上的所有容器日志，就需要做如下的日志源配置：

```
<source>

@id fluentd-containers.log

@type tail

path /var/log/containers/*.log

pos_file /var/log/fluentd-containers.log.pos

time_format %Y-%m-%dT%H:%M:%S.%NZ

tag raw.kubernetes.*

format json

read_from_head true

</source>
```

上面配置部分参数说明如下：

- id：表示引用该日志源的唯一标识符，该标识可用于进一步过滤和路由结构化日志数据
- type：Fluentd 内置的指令，`tail`表示 Fluentd 从上次读取的位置通过 tail 不断获取数据，另外一个是`http`表示通过一个 GET 请求来收集数据。
- path：`tail`类型下的特定参数，告诉 Fluentd 采集`/var/log/containers`目录下的所有日志，这是 docker 在 Kubernetes 节点上用来存储运行容器 stdout 输出日志数据的目录。
- pos_file：检查点，如果 Fluentd 程序重新启动了，它将使用此文件中的位置来恢复日志数据收集。
- tag：用来将日志源与目标或者过滤器匹配的自定义字符串，Fluentd 匹配源/目标标签来路由日志数据。

#### 路由配置

上面是日志源的配置，接下来看看如何将日志数据发送到 Elasticsearch：

```
<match **>

@id elasticsearch

@type elasticsearch

@log_level info

include_tag_key true

type_name fluentd

host "#{ENV['OUTPUT_HOST']}"

port "#{ENV['OUTPUT_PORT']}"

logstash_format true

<buffer>

@type file

path /var/log/fluentd-buffers/kubernetes.system.buffer

flush_mode interval

retry_type exponential_backoff

flush_thread_count 2

flush_interval 5s

retry_forever

retry_max_interval 30

chunk_limit_size "#{ENV['OUTPUT_BUFFER_CHUNK_LIMIT']}"

queue_limit_length "#{ENV['OUTPUT_BUFFER_QUEUE_LIMIT']}"

overflow_action block

</buffer>
```

- match：标识一个目标标签，后面是一个匹配日志源的正则表达式，我们这里想要捕获所有的日志并将它们发送给 Elasticsearch，所以需要配置成`**`。
- id：目标的一个唯一标识符。
- type：支持的输出插件标识符，我们这里要输出到 Elasticsearch，所以配置成 elasticsearch，这是 Fluentd 的一个内置插件。
- log_level：指定要捕获的日志级别，我们这里配置成`info`，表示任何该级别或者该级别以上（INFO、WARNING、ERROR）的日志都将被路由到 Elsasticsearch。
- host/port：定义 Elasticsearch 的地址，也可以配置认证信息，我们的 Elasticsearch 不需要认证，所以这里直接指定 host 和 port 即可。
- logstash_format：Elasticsearch 服务对日志数据构建反向索引进行搜索，将 logstash_format 设置为`true`，Fluentd 将会以 logstash 格式来转发结构化的日志数据。
- Buffer： Fluentd 允许在目标不可用时进行缓存，比如，如果网络出现故障或者 Elasticsearch 不可用的时候。缓冲区配置也有助于降低磁盘的 IO。

### 安装

要收集 Kubernetes 集群的日志，直接用 DasemonSet 控制器来部署 Fluentd 应用，这样，它就可以从 Kubernetes 节点上采集日志，确保在集群中的每个节点上始终运行一个 Fluentd 容器。当然可以直接使用 Helm 来进行一键安装，为了能够了解更多实现细节，我们这里还是采用手动方法来进行安装。

首先，我们通过 ConfigMap 对象来指定 Fluentd 配置文件，新建 fluentd-configmap.yaml 文件，文件内容如下：

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: fluentd-config
  namespace: logging
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
data:
  system.conf: |-
    <system>
      root_dir /tmp/fluentd-buffers/
    </system>
  containers.input.conf: |-
    <source>
      @id fluentd-containers.log
      @type tail
      path /var/log/containers/*.log
      pos_file /var/log/es-containers.log.pos
      time_format %Y-%m-%dT%H:%M:%S.%NZ
      localtime
      tag raw.kubernetes.*
      format json
      read_from_head true
    </source>
    # Detect exceptions in the log output and forward them as one log entry.
    <match raw.kubernetes.**>
      @id raw.kubernetes
      @type detect_exceptions
      remove_tag_prefix raw
      message log
      stream stream
      multiline_flush_interval 5
      max_bytes 500000
      max_lines 1000
    </match>
  system.input.conf: |-
    # Logs from systemd-journal for interesting services.
    <source>
      @id journald-docker
      @type systemd
      filters [{ "_SYSTEMD_UNIT": "docker.service" }]
      <storage>
        @type local
        persistent true
      </storage>
      read_from_head true
      tag docker
    </source>
    <source>
      @id journald-kubelet
      @type systemd
      filters [{ "_SYSTEMD_UNIT": "kubelet.service" }]
      <storage>
        @type local
        persistent true
      </storage>
      read_from_head true
      tag kubelet
    </source>
  forward.input.conf: |-
    # Takes the messages sent over TCP
    <source>
      @type forward
    </source>
  output.conf: |-
    # Enriches records with Kubernetes metadata
    <filter kubernetes.**>
      @type kubernetes_metadata
    </filter>
    <match **>
      @id elasticsearch
      @type elasticsearch
      @log_level info
      include_tag_key true
      host elasticsearch
      port 9200
      logstash_format true
      request_timeout    30s
      <buffer>
        @type file
        path /var/log/fluentd-buffers/kubernetes.system.buffer
        flush_mode interval
        retry_type exponential_backoff
        flush_thread_count 2
        flush_interval 5s
        retry_forever
        retry_max_interval 30
        chunk_limit_size 2M
        queue_limit_length 8
        overflow_action block
      </buffer>
    </match>
```

上面配置文件中我们配置了 docker 容器日志目录以及 docker、kubelet 应用的日志的收集，收集到数据经过处理后发送到 elasticsearch:9200 服务。

然后新建一个 fluentd-daemonset.yaml 的文件，文件内容如下：

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: fluentd-es
  namespace: logging
  labels:
    k8s-app: fluentd-es
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: fluentd-es
  labels:
    k8s-app: fluentd-es
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
rules:
- apiGroups:
  - ""
  resources:
  - "namespaces"
  - "pods"
  verbs:
  - "get"
  - "watch"
  - "list"
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: fluentd-es
  labels:
    k8s-app: fluentd-es
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
subjects:
- kind: ServiceAccount
  name: fluentd-es
  namespace: logging
  apiGroup: ""
roleRef:
  kind: ClusterRole
  name: fluentd-es
  apiGroup: ""
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-es
  namespace: logging
  labels:
    k8s-app: fluentd-es
    version: v2.0.4
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  selector:
    matchLabels:
      k8s-app: fluentd-es
      version: v2.0.4
  template:
    metadata:
      labels:
        k8s-app: fluentd-es
        kubernetes.io/cluster-service: "true"
        version: v2.0.4
      # This annotation ensures that fluentd does not get evicted if the node
      # supports critical pod annotation based priority scheme.
      # Note that this does not guarantee admission on the nodes (#40573).
      annotations:
        scheduler.alpha.kubernetes.io/critical-pod: ''
    spec:
      serviceAccountName: fluentd-es
      containers:
      - name: fluentd-es
        image: cnych/fluentd-elasticsearch:v2.0.4
        env:
        - name: FLUENTD_ARGS
          value: --no-supervisor -q
        resources:
          limits:
            memory: 500Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /data/docker/containers
          readOnly: true
        - name: config-volume
          mountPath: /etc/fluent/config.d
      nodeSelector:
        beta.kubernetes.io/fluentd-ds-ready: "true"
      tolerations:
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /data/docker/containers
      - name: config-volume
        configMap:
          name: fluentd-config
```

我们将上面创建的 fluentd-config 这个 ConfigMap 对象通过 volumes 挂载到了 Fluentd 容器中，另外为了能够灵活控制哪些节点的日志可以被收集，所以我们这里还添加了一个 nodSelector 属性：

```yaml
nodeSelector:
  beta.kubernetes.io/fluentd-ds-ready: "true"
```

意思就是要想采集节点的日志，那么我们就需要给节点打上上面的标签，比如我们这里3个节点都打上了该标签：

```shell
$ kubectl get nodes --show-labels
NAME      STATUS    ROLES     AGE       VERSION   LABELS
master    Ready     master    245d      v1.10.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/fluentd-ds-ready=true,beta.kubernetes.io/os=linux,kubernetes.io/hostname=master,node-role.kubernetes.io/master=
node02    Ready     <none>    165d      v1.10.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/fluentd-ds-ready=true,beta.kubernetes.io/os=linux,com=youdianzhishi,course=k8s,kubernetes.io/hostname=node02
node03    Ready     <none>    225d      v1.10.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/fluentd-ds-ready=true,beta.kubernetes.io/os=linux,jnlp=haimaxy,kubernetes.io/hostname=node03
```

另外由于我们的集群使用的是 kubeadm 搭建的，默认情况下 master 节点有污点，所以要想也收集 master 节点的日志，则需要添加上容忍：

```yaml
tolerations:
- key: node-role.kubernetes.io/master
  operator: Exists
  effect: NoSchedule
```

另外需要注意的地方是，我这里的测试环境更改了 docker 的根目录：

```shell
$ docker info
...
Docker Root Dir: /data/docker
...
```

所以上面要获取 docker 的容器目录需要更改成`/data/docker/containers`，这个地方非常重要，当然如果你没有更改 docker 根目录则使用默认的`/var/lib/docker/containers`目录即可。

分别创建上面的 ConfigMap 对象和 DaemonSet：

```shell
$ kubectl create -f fluentd-configmap.yaml
configmap "fluentd-config" created
$ kubectl create -f fluentd-daemonset.yaml
serviceaccount "fluentd-es" created
clusterrole.rbac.authorization.k8s.io "fluentd-es" created
clusterrolebinding.rbac.authorization.k8s.io "fluentd-es" created
daemonset.apps "fluentd-es" created
```

创建完成后，查看对应的 Pods 列表，检查是否部署成功：

```shell
$ kubectl get pods -n logging
NAME                      READY     STATUS    RESTARTS   AGE
es-cluster-0              1/1       Running   0          1d
es-cluster-1              1/1       Running   0          1d
es-cluster-2              1/1       Running   0          1d
fluentd-es-2z9jg          1/1       Running   1          35s
fluentd-es-6dfdd          1/1       Running   0          35s
fluentd-es-bfkg7          1/1       Running   0          35s
kibana-7558d4dc4d-5mqdz   1/1       Running   0          1d
```

**注意**：如果当前node没有打上`fluentd-ds-ready=true` lable的话，对应的pod `fluentd`不会被启动。



Fluentd 启动成功后，我们可以前往 Kibana 的 Dashboard 页面中，点击左侧的`Discover`，可以看到如下配置页面：

![create index](在Kubernetes上搭建EFK日志收集系统/kSLsgE.jpg)create index

在这里可以配置我们需要的 Elasticsearch 索引，前面 Fluentd 配置文件中我们采集的日志使用的是 logstash 格式，这里只需要在文本框中输入`logstash-*`即可匹配到 Elasticsearch 集群中的所有日志数据，然后点击下一步，进入以下页面：

![index config](在Kubernetes上搭建EFK日志收集系统/bc7n1S.jpg)index config

在该页面中配置使用哪个字段按时间过滤日志数据，在下拉列表中，选择`@timestamp`字段，然后点击`Create index pattern`，创建完成后，点击左侧导航菜单中的`Discover`，然后就可以看到一些直方图和最近采集到的日志数据了：

![log data](在Kubernetes上搭建EFK日志收集系统/wHFvnu.jpg)log data

### 测试

现在我们来将上一节课的计数器应用部署到集群中，并在 Kibana 中来查找该日志数据。

新建 counter.yaml 文件，文件内容如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox
    args: [/bin/sh, -c,
            'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done']
```

该 Pod 只是简单将日志信息打印到 stdout，所以正常来说 Fluentd 会收集到这个日志数据，在 Kibana 中也就可以找到对应的日志数据了，使用 kubectl 工具创建该 Pod：

```shell
$ kubectl create -f counter.yaml
```

Pod 创建并运行后，回到 Kibana Dashboard 页面，在上面的`Discover`页面搜索栏中输入`kubernetes.pod_name:counter`，就可以过滤 Pod 名为 counter 的日志数据：

![counter log data](在Kubernetes上搭建EFK日志收集系统/HsYSF3.jpg)counter log data

我们也可以通过其他元数据来过滤日志数据，比如 您可以单击任何日志条目以查看其他元数据，如容器名称，Kubernetes 节点，命名空间等。

到这里，我们就在 Kubernetes 集群上成功部署了 EFK ，要了解如何使用 Kibana 进行日志数据分析，可以参考 Kibana 用户指南文档：<https://www.elastic.co/guide/en/kibana/current/index.html>

当然对于在生产环境上使用 Elaticsearch 或者 Fluentd，还需要结合实际的环境做一系列的优化工作，本文中涉及到的资源清单文件都可以在<https://github.com/cnych/kubernetes-learning/tree/master/efkdemo>找到。

> 参考文档: [How To Set Up an Elasticsearch, Fluentd and Kibana (EFK) Logging Stack on Kubernetes](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-elasticsearch-fluentd-and-kibana-efk-logging-stack-on-kubernetes)
>
> 参考链接：<https://www.jianshu.com/p/1000ae80a493>



## 部署logtrail

下载kibana对应版本的logtrail <https://github.com/sivasamyk/logtrail>

由于kibana镜像不带logtrail，安装插件后需要重启kibana，在容器下无法重启。因此需要基于kibana镜像重新生成一个带插件的新镜像。

以6.7.0版本为例，Dockerfile如下：

```
# elk https://hub.docker.com/r/sebp/elk
#FROM sebp/elk:670
FROM docker.elastic.co/kibana/kibana-oss:6.7.0
#ADD 02-beats-input.conf /etc/logstash/conf.d/02-beats-input.conf
#ADD 30-output.conf /etc/logstash/conf.d/30-output.conf
# kibanasivasamyk/logtrail 
ADD ./logtrail-6.7.0-0.1.31.zip /opt/kibana/plugin/logtrail-6.7.0-0.1.31.zip
#  kibana
# WORKDIR ${KIBANA_HOME}
# kibana
RUN bin/kibana-plugin install file:///opt/kibana/plugin/logtrail-6.7.0-0.1.31.zip
```

执行build命令

>docker build  --network=host -t docker.elastic.co/kibana/kibana-oss:kibana-logtrail .

或者

```dockerfile
FROM docker.elastic.co/kibana/kibana-oss:6.2.4
RUN kibana-plugin install https://github.com/sivasamyk/logtrail/releases/download/v0.1.27/logtrail-6.2.4-0.1.27.zip
WORKDIR /config
USER root
RUN mv /usr/share/kibana/plugins/logtrail/logtrail.json /config/logtrail.json && \
    ln -s /config/logtrail.json /usr/share/kibana/plugins/logtrail/logtrail.json
USER kibana
```

使用configmap

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: logtrail-config
data:
  logtrail.json: |
    {
        "version" : 1,
        "index_patterns" : [
        {      
            "es": {
                "default_index": "logstash-*"
            },
            "tail_interval_in_seconds": 10,
            "es_index_time_offset_in_seconds": 0,
            "display_timezone": "local",
            "display_timestamp_format": "MMM DD HH:mm:ss",
            "max_buckets": 500,
            "default_time_range_in_days" : 0,
            "max_hosts": 100,
            "max_events_to_keep_in_viewer": 5000,
            "fields" : {
                "mapping" : {
                    "timestamp" : "@timestamp",
                    "hostname" : "kubernetes.host",
                    "program": "kubernetes.pod_name",
                    "message": "log"
                },
                "message_format": "{{{log}}}"
            },
            "color_mapping" : {
            }
        }]
    } 
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: kibana
  labels:
    component: kibana
spec:
  replicas: 1
  selector:
    matchLabels:
     component: kibana
  template:
    metadata:
      labels:
        component: kibana
    spec:
      containers:
      - name: kibana
        image: sh4d1/kibana-logtrail:6.2.4 # or your image
        volumeMounts:
          - name: logtrail-config
            mountPath: /config
        env:
        - name: CLUSTER_NAME
          value: myesdb # the name of your ES cluster
        resources:
          limits:
            cpu: 1000m
          requests:
            cpu: 100m
        ports:
        - containerPort: 5601
          name: http
      volumes:
        - name: logtrail-config
          configMap:
            name: logtrail-config
```

Then the important part is the `fields` section. It will display like:

```
timestamp hostname program:message
```

