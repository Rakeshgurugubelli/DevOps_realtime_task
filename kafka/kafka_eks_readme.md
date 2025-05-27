**Installing kafka on eks using strimzi**

it won't work on ALB will work on nlb

**Step-1:** create a namespace of kafka or strimzi based on requirement

![image](https://github.com/user-attachments/assets/c9d34afb-f009-42e4-863e-6f419bae6992)

**Step-2:** Install Strimzi Operator in the kafka namespace.

**purpose of installing strimzi operator**

. The Strimzi Operator makes it easy to run Apache Kafka on Kubernetes.

**Without Strimzi:**

Running Kafka on Kubernetes is hard:

You need to manually configure Kafka, Zookeeper, storage, networking, etc.

Updating or scaling Kafka is complex and error-prone.

**With Strimzi:**

✅ Strimzi automates all that:

Creates and manages Kafka and Zookeeper pods

Handles upgrades, scaling, and recovery

Manages users, topics, and TLS certs

Makes Kafka “Kubernetes native”

**command**: helm install strimzi-operator strimzi/strimzi-kafka-operator \
  --version 0.37.0 \
  --namespace kafka \
  --create-namespace

![image](https://github.com/user-attachments/assets/f2b416b8-2d2b-43fe-9fa9-c242c565330a)


**Step-3:** Create a custom resource

**why custom resource of kafka**

**Declarative Kafka cluster management:** Instead of manually installing and configuring kafka and zookeeper,you define your Kafka cluster’s desired state in a YAML file

**Operator automation:** Automates the lifecycle: creating, updating, scaling, and recovering Kafka clusters according to the YAML spec.

**Simplifies complex setups:** Kafka and Zookeeper clusters have many components and configs. Custom resources let you manage all of these as a single unit, instead of handling each piece individually.

**Consistent and repeatable:** You can version-control your Kafka CR manifests, making your Kafka infrastructure consistent and easy to reproduce across environments.

**strimizi-customresource.yml**

apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: strimzi-kafka-cluster
  namespace: kafka
spec:
  kafka:
    version: 3.5.1
    replicas: 1
    config:
      offsets.topic.replication.factor: 1
      transaction.state.log.replication.factor: 1
      transaction.state.log.min.isr: 1
      auto.create.topics.enable: "true"
    listeners:
      - name: internal
        port: 9092
        type: internal
        tls: false
        authentication:
          type: scram-sha-512
      - name: external
        port: 9093
        type: loadbalancer
        tls: true
        authentication:
          type: scram-sha-512
        configuration:
          bootstrap:
            annotations:
              service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
              service.beta.kubernetes.io/aws-load-balancer-backend-protocol: ssl
              service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "9093"
          brokers:
            - broker: 0
              annotations:
                service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
                service.beta.kubernetes.io/aws-load-balancer-backend-protocol: ssl
                service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "9093"
    authorization:
      type: simple
    storage:
      type: ephemeral
  zookeeper:
    replicas: 1
    storage:
      type: ephemeral
  entityOperator:
    topicOperator: {}
    userOperator: {}

**Step-4:** Once the custom resource is deployed verify logs of each pod with no errors and services are created

kubectl get pod -n kafka

![image](https://github.com/user-attachments/assets/8d19b060-c200-4ee3-bcf3-bdea63c894b7)

kubectl get svc -n kafka

![image](https://github.com/user-attachments/assets/0508465b-57c7-4630-8480-bcfac2a2f97e)

verify the logs of each pod, ensure no errors

**Step-5:** create a kafka user

Kafka has built-in authentication and authorization mechanisms (SASL, TLS, etc.).

Kafka clients (producers, consumers, admin tools) connect using credentials.

The KafkaUser CR lets you create Kafka users with certificates or passwords managed by Strimzi.

This user can then be referenced by your apps to authenticate with the Kafka cluster securely.

**Without creating users, your Kafka cluster might be inaccessible or only open without auth (which is unsafe).**

**kafka-user.yml**

apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaUser
metadata:
  name: my-user
  namespace: kafka
  labels:
    strimzi.io/cluster: strimzi-kafka-cluster
spec:
  authentication:
    type: scram-sha-512
  authorization:
    type: simple
    acls:
      - resource:
          type: topic
          name: '*'
          patternType: literal
        operation: Create
      - resource:
          type: topic
          name: '*'
          patternType: literal
        operation: Read
      - resource:
          type: topic
          name: '*'
          patternType: literal
        operation: Write
      - resource:
          type: group
          name: '*'
          patternType: literal
        operation: Read
      - resource:
          type: cluster
          name: kafka-cluster
        operation: Create

![image](https://github.com/user-attachments/assets/fb163b57-bd8f-4671-8864-222b57c4854f)

**Step-6:** Create a topic

Kafka topics are where your data streams go — producers send data to topics, consumers read from them.

With Strimzi, you can declare the topics you want in Kubernetes manifests as KafkaTopic CRs.

Strimzi will ensure those topics are created and managed (partitions, replicas, configs) on the Kafka cluster automatically.

This allows you to keep topics configuration as code, track changes via GitOps, and avoid manual kafka-topics.sh CLI commands.

**kafka-topic.yml**

apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata:
  name: my-topic
  namespace: kafka  # Must match the namespace of your Kafka cluster
  labels:
    strimzi.io/cluster: strimzi-kafka-cluster  # Must match your Kafka CR name
spec:
  partitions: 3
  replicas: 1
  config:
    retention.ms: 86400000   # 1 day
    segment.bytes: 10485760

![image](https://github.com/user-attachments/assets/5b148d6a-d0b9-41a6-9a99-93a0b5141f5e)

**Step-7:** Generate a username and password and copy it

username: my-user
password: GV8rbVEzm0BpUwZ29IUQke7xMTlSD9BD


kubectl get secret my-user -n kafka -o jsonpath='{.data.password}' | base64 -d; echo

![image](https://github.com/user-attachments/assets/1252ffdc-1906-46ac-8234-6676492416a3)

**Step-8:** Create a client.properties file and copy it to strimzi-kafka-cluster-kafka-0

cat > client.properties <<EOF

security.protocol=SASL_PLAINTEXT

sasl.mechanism=SCRAM-SHA-512

sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="my-user" password="GV8rbVEzm0BpUwZ29IUQke7xMTlSD9BD";

EOF

kubectl cp client.properties strimzi/strimzi-kafka-cluster-kafka-0:/tmp/client.properties

![image](https://github.com/user-attachments/assets/9381fb1d-1f28-4592-8d72-8fbc0d8ac718)

**Test the below steps once above steps are completed Successfully**

**Step-9:** Go inside the pod, verify the client.properties file copied or not and list the topics , create a topics

kubectl exec -n kafka -it strimzi-kafka-cluster-kafka-0 -- bash

cat /tmp/client.properties

![image](https://github.com/user-attachments/assets/c0fc8869-97cd-4654-be1c-edecc205b4a7)

**list the topics:**

kubectl exec -n kafka -it strimzi-kafka-cluster-kafka-0 -- bash

bin/kafka-topics.sh --list --bootstrap-server localhost:9092 --command-config /tmp/client.properties

![image](https://github.com/user-attachments/assets/74d361ef-9e9f-4d82-a578-ad7f517ed6c3)

**create a custom topic & verify**

kubectl exec -n kafka -it strimzi-kafka-cluster-kafka-0 -- bash

bin/kafka-topics.sh --create --topic test-topic123 --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1 --command-config /tmp/client.properties

![image](https://github.com/user-attachments/assets/d7a477dc-751b-4a76-8209-680bb26113e4)

![image](https://github.com/user-attachments/assets/5114ca02-ae3c-4dc0-8be1-5492101b9d3d)

Step-10: Test the message broker means producer and consumer

Copy the loadbalancer Dns of the strimzi-kafka-cluster-kafka-external-0

install kcat and verify the dns working or not using below command

telnet k8s-kafka-strimzik-34783085d2-9cd6c6a94c21c8b4.elb.ap-south-1.amazonaws.com -p 9093

--producer--

 kcat -b0 k8s-kafka-strimzik-34783085d2-9cd6c6a94c21c8b4.elb.ap-south-1.amazonaws.com:9093 \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=my-user \
  -X sasl.password='GV8rbVEzm0BpUwZ29IUQke7xMTlSD9BD' \
  -X ssl.endpoint.identification.algorithm=none \
  -t test-topic \
  -P

--consumer--

kcat -b k8s-kafka-strimzik-34783085d2-9cd6c6a94c21c8b4.elb.ap-south-1.amazonaws.com:9093   -X security.protocol=SASL_SSL   -X sasl.mechanisms=SCRAM-SHA-512   -X sasl.username=my-user   -X sasl.password='GV8rbVEzm0BpUwZ29IUQke7xMTlSD9BD'   -X ssl.endpoint.identification.algorithm=none   -t test-topic -C -o beginning -c 1

