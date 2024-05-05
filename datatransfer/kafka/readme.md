#https://ambidextrous-dev.medium.com/kafka-on-kubernetes-ac245694f853

Kafka on Kubernetes

Ambidextrous
Ambidextrous
·
Follow
10 min read
·
# source repo

https://github.com/ambidextrous-dev/k8s-kafka


Dec 6, 2023

Which Helm Chart to Choose?
Due to the time constraints, we decided to go ahead with the open-source helm charts rather than going ahead with our custom manifests. This way we traded off flexibility for speed of execution. Two of the most popular helm charts for Kafka on Kubernetes are — Confluent and Bitnami. I first explored the Confluent chart expecting it to be better 3. Customize Helm Values — The helm chart usually comes with a default set of values. We can override the values with our values.yaml at the runtime. Below is a sample values.yaml file for a Kafka Cluster comprising of 3 brokers and 3 zookeepers in TLS mode, and HA configuration. We have added podAntiAffinityPreset: hard to make sure that no two broker/zookeeper pods are on the same node. These Kafka brokers will be accessed via Nodeport on the specified ports.


This next section goes through the step-by-step procedure to install a Kafka Cluster on a Kubernetes Cluster:

1. Namespace: Create a dedicated namespace for this Kafka cluster. A dedicated namespace allows us to isolate this cluster from other apps and is a good organizational tool for organizing resources
kubectl create namespace kafka

2. Persistent Volumes (PV) — Before installing the Kafka broker and Zookeeper, we need to finalize our persistent volume. For this use case, we went with a dedicated NFS driver which will be shared between all the Kafka brokers and zookeepers. We will set up a 3 broker and 3 zookeeper node setup, therefore we will need 6 persistent volumes in total. The persistent volume claims will be generated automatically by the helm chart at the runtime.


3. Customize Helm Values — The helm chart usually comes with a default set of values. We can override the values with our values.yaml at the runtime. Below is a sample values.yaml file for a Kafka Cluster comprising of 3 brokers and 3 zookeepers in TLS mode, and HA configuration. We have added podAntiAffinityPreset: hard to make sure that no two broker/zookeeper pods are on the same node. These Kafka brokers will be accessed via Nodeport on the specified ports.

image:
  tag: 3.5.1-debian-11-r25 #update as per latest chart version
  pullPolicy: Always

existingLog4jConfigMap: "kafka-log4j-config"

kraft:
  enabled: false

auth:
  interBrokerProtocol: tls

broker:
  replicaCount: 3
  podAntiAffinityPreset: hard

controller:
  replicaCount: 0

zookeeper:
  enabled: true
  replicaCount: 3
  podAntiAffinityPreset: hard

listeners:
  client:
    protocol: SSL

  interbroker:
    protocol: SSL

  external:
    protocol: SSL

tls:
  type: jks
  existingSecret: kafka-tls
  passwordsSecret: kafka-tls-passwords
  passwordsSecretKeystoreKey: keystore-password
  passwordsSecretTruststoreKey: truststore-password

externalAccess:
  enabled: true
  broker:
    service:
      type: NodePort
      nodePorts:
        - 30066
        - 30067
        - 30068
      externalIPs:
        - <<Node1IP>>
        - <<Node2IP>>
        - <<Node3IP>>

  controller:
    service:
      type: ClusterIP
      domain: kafka-dev.example.com # not being used currently
4. Create secrets — As this is a TLS-secured cluster, we need to create a keystore and truststore and then create two separate Kubernetes secrets to store the secrets and the credentials. The name of the secrets and the key should match the ones specified in the values.yaml file
# Create TLS secret
kubectl create secret generic kafka-tls -n kafka \
    --from-file=truststore/kafka.truststore.jks \
    --from-file=keystore/kafka.keystore.jks

# Create a Secret named kafka-tls-passwords with the passwords
kubectl create secret generic kafka-tls-passwords \
  --from-literal=keystore-pwd='your_keystore_password_here' \
  --from-literal=truststore-pwd='your_truststore_password_here'
5. Customize Logging — We can also customize the logging of our Kafka cluster by passing a custom log4j config file in a config map.
kubectl create configmap kafka-log4j-config -n kafka \
    --from-file=./config/log4j.properties 
6. Install Helm Chart — We are now ready to install the Helm Chart in our cluster. This will install two StatefulSets — kafka-broker and kafka-zookeeper which will further manage the pods.
helm3 repo add bitnami https://charts.bitnami.com/bitnami
helm3 repo update

helm3 upgrade --install kafka -f values-dev.yaml -n kafka bitnami/kafka



Schema Registry: Kafka Schema Registry is a centralized service in the Apache Kafka ecosystem for managing AVRO or JSON schemas used in message serialization. It ensures data consistency by enforcing a schema for producers and consumers, allowing seamless evolution of data formats. Here are the steps to set up Kafka Schema Registry on Kubernetes:
Create a Kubernetes secretkafka-schema-registry-ssl that contains the truststore of the Kafka cluster and a keystore for the schema registry. This secret is then later mapped as a volume in step 2.
kubectl create secret generic kafka-schema-registry-ssl -n kafka \
    --from-file=truststore/kafka.truststore.jks \
    --from-file=client_certs/schema-registry/schema-registry.keystore.jks
2. Then, create a manifest file for deployment. The Schema Registry(SR) will run in SSL mode. We pass the TLS truststore and keystore secrets, and their passwords as environment variables to the container. SCHEMA_REGISTRY_LISTENERS environment variable is used to specify the listeners or network interfaces on which the Schema Registry should bind and listen for incoming requests. In this case, the SR will be exposed internally on port 8081 and externally on port 30092
apiVersion: apps/v1
kind: Deployment
metadata:
  name: schema-registry
  namespace: kafka
spec:
  replicas: 2
  selector:
    matchLabels:
      app: schema-registry
  template:
    metadata:
      labels:
        app: schema-registry
    spec:
      containers:
        - name: schema-registry
          image: confluentinc/cp-schema-registry:7.4.1
          imagePullPolicy: Always
          ports:
            - containerPort: 8081
          env:
            - name: SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS
              value: "kafka-broker-0.kafka-broker-headless.kafka.svc.cluster.local:9094,kafka-broker-1.kafka-broker-headless.kafka.svc.cluster.local:9094,kafka-broker-2.kafka-broker-headless.kafka.svc.cluster.local:9094"
            - name: SCHEMA_REGISTRY_KAFKASTORE_SECURITY_PROTOCOL
              value: "SSL"
            - name: SCHEMA_REGISTRY_LISTENERS
              value: "http://0.0.0.0:8081,http://0.0.0.0:30092"
            - name: SCHEMA_REGISTRY_KAFKASTORE_SSL_TRUSTSTORE_LOCATION
              value: /etc/ssl/certs/kafka.truststore.jks
            - name: SCHEMA_REGISTRY_KAFKASTORE_SSL_KEYSTORE_LOCATION
              value: /etc/ssl/certs/schema-registry.keystore.jks
            - name: SCHEMA_REGISTRY_KAFKASTORE_SSL_TRUSTSTORE_PASSWORD
              value: <<SuperStrongTrustStorePassword>>
            - name: SCHEMA_REGISTRY_KAFKASTORE_SSL_KEYSTORE_PASSWORD
              value: <<SuperStrongKeyStorePassword>>
            - name: SCHEMA_REGISTRY_HOST_NAME
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
          volumeMounts:
            - name: ssl-certs
              mountPath: /etc/ssl/certs
              readOnly: true
      volumes:
        - name: ssl-certs
          secret:
            secretName: kafka-schema-registry-ssl
3. Next, we create a manifest file for a service for Schema Registry which exposes the pods via NodePort
apiVersion: v1
kind: Service
metadata:
  name: schema-registry-svc
  namespace: kafka
spec:
  selector:
    app: schema-registry
  type: NodePort
  ports:
    - name: http
      port: 8081
      targetPort: 8081
      nodePort: 30092
    - name: https
      port: 8443
      targetPort: 8443
      nodePort: 30443
4. Now, that we have all the required components for the Schema Registry, we just need to apply them via kubectl
# Deploy Schema Registry
kubectl apply -f k8s-manifests/$environment/schema-registry/service.yaml \
  --namespace kafka
kubectl apply -f k8s-manifests/$environment/schema-registry/deployment.yaml \
  --namespace kafka 
Kafka Connect: Kafka Connect is a distributed data integration framework within the Apache Kafka ecosystem. It facilitates the seamless movement of data between Kafka and external systems through connectors. Source connectors ingest data into Kafka, while sink connectors export data from Kafka to external systems. Here are the steps to set up Kafka Connect on Kubernetes:
Create a Kubernetes secretkafka-connect-ssl that contains the truststore of the Kafka cluster and a keystore for the Kafka Connect app. This secret is then later mapped as a volume in step 2.
# Create Kafka Connect Secret
kubectl create secret generic kafka-connect-ssl -n kafka \
    --from-file=truststore/kafka.truststore.jks \
    --from-file=client_certs/kafka-connect/kafka-connect.keystore.jks
2. Create a manifest file for PV and PVC. Similar to how we created an NFS-based PV for Kafka broker, we will create a PV for Kafka Connect. But, this time, we will create a Persistent Volume Claim (PVC), which the app will use to request storage from the PV.
#kafka-connect-pv.yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: kafka-connect-dev-pv
  labels:
    app: kafka-connect
spec:
  storageClassName: local-storage
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 10Gi
  nfs:
    server: 10.2.2.10 #TODO: update with correct NFS server
    path: /mnt/kafka/kafka-connect1
  volumeMode: Filesystem
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: kafka-connect-dev-config-pvc
  namespace: kafka
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
  storageClassName: local-storage
3. Create a Kubernetes deployment manifest that deploys kafka-connect. Here is a sample YAML that deploys Kafka Connect, provides TLS information for it to connect with the Kafka Brokers, mounts PV as a volume, and mounts another volume that maps the secrets created in step 1 with the container. The Kafka Connect deployment also references the Kafka Schema Registry deployed in the previous step.
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kafka-connect
  namespace: kafka
spec:
  replicas: 2
  selector:
    matchLabels:
      app: kafka-connect
  template:
    metadata:
      labels:
        app: kafka-connect
    spec:
      containers:
        - name: kafka-connect
          image: confluentinc/cp-kafka-connect:7.4.1
          env:
            - name: CONNECT_BOOTSTRAP_SERVERS
              value: "kafka-broker-0.kafka-broker-headless.kafka.svc.cluster.local:9094,kafka-broker-1.kafka-broker-headless.kafka.svc.cluster.local:9094,kafka-broker-2.kafka-broker-headless.kafka.svc.cluster.local:9094"
            - name: CONNECT_CONFIG_STORAGE_TOPIC
              value: connect-configs
            - name: CONNECT_GROUP_ID
              value: kafka-connect-dev
            - name: CONNECT_KEY_CONVERTER
              value: io.confluent.connect.avro.AvroConverter
            - name: CONNECT_VALUE_CONVERTER
              value: io.confluent.connect.avro.AvroConverter
            - name: CONNECT_OFFSET_STORAGE_TOPIC
              value: "kafka-connect-offsets"
            - name: CONNECT_STATUS_STORAGE_TOPIC
              value: "kafka-connect-status"
            - name: CONNECT_SECURITY_PROTOCOL
              value: "SSL"
            - name: CONNECT_SSL_TRUSTSTORE_LOCATION
              value: /etc/kafka/secrets/kafka.truststore.jks
            - name: CONNECT_SSL_TRUSTSTORE_PASSWORD
              value: <<SuperStrongTrustStorePassword>>
            - name: CONNECT_SSL_KEYSTORE_LOCATION
              value: /etc/kafka/secrets/kafka-connect.keystore.jks
            - name: CONNECT_SSL_KEYSTORE_PASSWORD
              value: <<SuperStrongKeyStorePassword>>
            - name: CONNECT_PLUGIN_PATH
              value: "/usr/share/java"
            - name: CONNECT_REST_ADVERTISED_HOST_NAME
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
            - name: CONNECT_KEY_CONVERTER_SCHEMA_REGISTRY_URL
              value: http://<<schemaregistryserver>>:30092
            - name: CONNECT_VALUE_CONVERTER_SCHEMA_REGISTRY_URL
              value: http://<<schemaregistryserver>>:30092
          volumeMounts:
            - name: secrets-volume
              mountPath: /etc/kafka/secrets
            - name: config-volume
              mountPath: /etc/kafka/connect
      volumes:
        - name: secrets-volume
          secret:
            secretName: kafka-connect-ssl
        - name: config-volume
          persistentVolumeClaim:
            claimName: kafka-connect-dev-config-pvc #link pvc created in previous step
4. The last manifest file needed is a service file that will expose Kafka Connect outside of the cluster via NodePort. Here, the app is exposed externally on port 30099 and internally on port 8083.
apiVersion: v1
kind: Service
metadata:
  name: kafka-connect-svc
  namespace: kafka
spec:
  selector:
    app: kafka-connect
  type: NodePort
  ports:
    - port: 8083
      targetPort: 8083
      nodePort: 30099
5. Now, all our manifest files and dependent artifacts are created. We can apply these manifests via kubectl:
# Deploy Kafka Connect
kubectl apply -f k8s-manifests/$environment/kafka-connect/pv.yaml \
   --namespace kafka
kubectl apply -f k8s-manifests/$environment/kafka-connect/service.yaml \
   --namespace kafka
kubectl apply -f k8s-manifests/$environment/kafka-connect/deployment.yaml \
   --namespace kafka
That’s it, that finishes our setup process and we now have a Kafka Cluster on Kubernetes comprising Kafka Brokers, Zookeepers, Schema Registry, and Kafka Connect.


Hopefully, this article is helpful for you. All of the code samples for this exercise are available on GitHub for reference.


