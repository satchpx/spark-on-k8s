# Spark On Kubernetes
This is my experience running spark applications on kubernetes via the operator

## Begin:
	Using the helm chart below to deploy the operator:
  ```
  https://github.com/GoogleCloudPlatform/spark-on-k8s-operator
  ```
	The operator spins up in spark-operator namespace.

## Next Steps:
  Deploy an example spark application in the examples directory. Upon following the logs from the operator, there are errors reported
  ```
  +0000 UTC { 0    } {SUBMISSION_FAILED failed to run spark-submit for SparkApplication default/pyspark-pi: Exception in thread "main" io.fabric8.kubernetes.client.KubernetesClientException: Failure executing: POST at: https://10.233.0.1/api/v1/namespaces/default/pods. Message: Forbidden!Configured service account doesn't have access. Service account may have been revoked. pods "pyspark-pi-driver" is forbidden: error looking up service account default/spark: serviceaccount "spark" not found.
 at io.fabric8.kubernetes.client.dsl.base.OperationSupport.requestFailure(OperationSupport.java:470)
 at io.fabric8.kubernetes.client.dsl.base.OperationSupport.assertResponseCode(OperationSupport.java:407)
 at io.fabric8.kubernetes.client.dsl.base.OperationSupport.handleResponse(OperationSupport.java:379)
 at io.fabric8.kubernetes.client.dsl.base.OperationSupport.handleResponse(OperationSupport.java:343)
 at io.fabric8.kubernetes.client.dsl.base.OperationSupport.handleCreate(OperationSupport.java:226)
 at io.fabric8.kubernetes.client.dsl.base.BaseOperation.handleCreate(BaseOperation.java:769)
 at io.fabric8.kubernetes.client.dsl.base.BaseOperation.create(BaseOperation.java:356)
 at org.apache.spark.deploy.k8s.submit.Client$$anonfun$run$2.apply(KubernetesClientApplication.scala:141)
 at org.apache.spark.deploy.k8s.submit.Client$$anonfun$run$2.apply(KubernetesClientApplication.scala:140)
 at org.apache.spark.util.Utils$.tryWithResource(Utils.scala:2543)
 at org.apache.spark.deploy.k8s.submit.Client.run(KubernetesClientApplication.scala:140)
 at org.apache.spark.deploy.k8s.submit.KubernetesClientApplication$$anonfun$run$5.apply(KubernetesClientApplication.scala:250)
 at org.apache.spark.deploy.k8s.submit.KubernetesClientApplication$$anonfun$run$5.apply(KubernetesClientApplication.scala:241)
 at org.apache.spark.util.Utils$.tryWithResource(Utils.scala:2543)
 at org.apache.spark.deploy.k8s.submit.KubernetesClientApplication.run(KubernetesClientApplication.scala:241)
 at org.apache.spark.deploy.k8s.submit.KubernetesClientApplication.start(KubernetesClientApplication.scala:204)
 at org.apache.spark.deploy.SparkSubmit.org$apache$spark$deploy$SparkSubmit$$runMain(SparkSubmit.scala:849)
 at org.apache.spark.deploy.SparkSubmit.doRunMain$1(SparkSubmit.scala:167)
 at org.apache.spark.deploy.SparkSubmit.submit(SparkSubmit.scala:195)
 at org.apache.spark.deploy.SparkSubmit.doSubmit(SparkSubmit.scala:86)
 at org.apache.spark.deploy.SparkSubmit$$anon$2.doSubmit(SparkSubmit.scala:924)
 at org.apache.spark.deploy.SparkSubmit$.main(SparkSubmit.scala:933)
 at org.apache.spark.deploy.SparkSubmit.main(SparkSubmit.scala)
 ```

 Create a serviceaccount
 ```
 kubectl create serviceaccount spark
 ```

 To grant a service account a Role or ClusterRole, a RoleBinding or ClusterRoleBinding is needed. To create a RoleBinding or ClusterRoleBinding, a user can use the kubectl create rolebinding (or clusterrolebinding for ClusterRoleBinding) command. For example, the following command creates an edit ClusterRole in the default namespace and grants it to the spark service account created above:
 ```
 kubectl create clusterrolebinding spark-role --clusterrole=edit --serviceaccount=default:spark --namespace=default
 ```

 Now run an example spark application again
 ```
 kubectl apply -f examples/spark-py-pi.yaml
 kubectl apply -f examples/spark-pi-prometheus.yaml
 ```

 At this point, you should see driver and executor pods corresponding to the above applications.

 Delete the CRD's above to verify if the pods get cleaned up
 ```
 kubectl delete -f examples/spark-py-pi.yaml
 kubectl delete -f examples/spark-pi-prometheus.yaml
 ``` 
