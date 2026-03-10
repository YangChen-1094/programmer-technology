
### **1、StatefulSets使用场景**

在具有以下特点时使用StatefulSets：
* 稳定性，唯一的网络标识符。
* 稳定性，持久化存储。
* 有序的部署和扩展。
* 有序的删除和终止。
* 有序的自动滚动更新。

### **2、部署和扩展**

* 对于具有N个副本的StatefulSet，当部署Pod时，将会顺序从{0..N-1}开始创建。
* Pods被删除时，会从{N-1..0}的相反顺序终止。
* 在将缩放操作应用于Pod之前，它的所有前辈必须运行和就绪。
* 对Pod执行扩展操作时，前面的Pod必须都处于Running和Ready状态。
* 在Pod终止之前，所有successors都须完全关闭。
```mysql
##部署示例
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: gcr.io/google_containers/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: my-storage-class
      resources:
        requests:
          storage: 1Gi
```
<span style='color:red'>在上面示例中，会按顺序部署三个pod（name: web-0、web-1、web-2）。web-0在Running and Ready状态后开始部署web-1，web-1在Running and Ready状态后部署web-2，期间如果web-0运行失败，web-2是不会被运行，直到web-0重新运行，web-1、web-2才会按顺序进行运行。</span>
如果用户通过StatefulSet来扩展修改部署pod副本数，比如修改replicas=1，那么web-2首先被终止。在web-2完全关闭和删除之前，web-1是不会被终止。如果在web-2被终止和完全关闭后，但web-1还没有被终止之前，此时web-0运行出错了，那么直到web-0再次变为Running and Ready状态之后，web-1才会被终止。
