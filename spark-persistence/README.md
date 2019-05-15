# Spark persistence with Portworx on Kubernetes
This example illustrates how to achieve persistence for spark applications using Portworx on Kubernetes.


## Concept
We'll be using GoogleCloudPlatform Spark Operator on K8S to illustrate spark persistence. To deploy the spark operator, refer to: https://github.com/GoogleCloudPlatform/spark-on-k8s-operator


The idea behind this solution is to use spark checkpointing and write ahead logs (WAL). To enable this, we need to deploy spark operator with mutating admission webhook enabled. Refer to https://github.com/GoogleCloudPlatform/spark-on-k8s-operator/blob/master/docs/quick-start-guide.md#about-the-mutating-admission-webhook


Now, create a storageClass that can be used to create persistentVolumeClaim for the persistent spark apps.
```
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
  name: portworx-spark-sc
provisioner: kubernetes.io/portworx-volume
parameters:
  repl: "1"
  shared: "true"
```

Let's create a persistentVolumeClaim to be used in the persistent spark app
```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: spark-data
  annotations:
    volume.beta.kubernetes.io/storage-class: portworx-spark-sc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

Now, that we have the storageClass and persistentVolumeClaim in place, we can use the persistent volume for the driver and executor pods.
In the example below, notice the `volumes` and `volumeMounts` specification, where we're using the persistent volume
```
apiVersion: "sparkoperator.k8s.io/v1beta1"
kind: SparkApplication
metadata:
  name: spark-pi
  namespace: default
spec:
  type: Scala
  mode: cluster
  image: "gcr.io/spark-operator/spark:v2.4.0"
  imagePullPolicy: Always
  mainClass: org.apache.spark.examples.SparkPi
  mainApplicationFile: "local:///opt/spark/examples/jars/spark-examples_2.11-2.4.0.jar"
  sparkVersion: "2.4.0"
  restartPolicy:
    type: Never
  volumes:
    - name: "spark-data"
      persistentVolumeClaim:
        claimName: spark-data  driver:
    cores: 0.1
    coreLimit: "200m"
    memory: "512m"
    labels:
      version: 2.4.0
    serviceAccount: spark
    volumeMounts:
      - name: "spark-data"
        mountPath: "/tmp"
  executor:
    cores: 1
    instances: 1
    memory: "512m"
    labels:
      version: 2.4.0
    volumeMounts:
      - name: "spark-data"
        mountPath: "/tmp"
```

Note that the example above does not use an application that needs persistence. The above example is merely to demonstrate how to use a persistent volume in the driver and executor pods.
The application should have checkpointing enabled. Refer to: https://spark.apache.org/docs/latest/streaming-programming-guide.html#how-to-configure-checkpointing

## @TODO: Add a sample application that uses checkpointing, and illustrates persistence using Portworx
