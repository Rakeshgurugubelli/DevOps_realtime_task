**Installing kafka on eks using strimzi**

it won't on ALB will work on nlb

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

**command**: kubectl apply -f https://strimzi.io/install/latest?namespace=kafka -n kafka

![image](https://github.com/user-attachments/assets/b6aa8e0f-78a1-4801-a021-1ee50f0db14f)

**Step-3:** Create a custom resource

**why custom resource of kafka**

**Declarative Kafka cluster management:** Instead of manually installing and configuring kafka and zookeeper,you define your Kafka cluster’s desired state in a YAML file

**Operator automation:** Automates the lifecycle: creating, updating, scaling, and recovering Kafka clusters according to the YAML spec.

**Simplifies complex setups:** Kafka and Zookeeper clusters have many components and configs. Custom resources let you manage all of these as a single unit, instead of handling each piece individually.

**Consistent and repeatable:** You can version-control your Kafka CR manifests, making your Kafka infrastructure consistent and easy to reproduce across environments.

strimizi-customresource.yml

apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: strimzi-kafka-cluster
  namespace: strimzi
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
          #  authentication:
          # type: scram-sha-512
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
              service.beta.kubernetes.io/aws-load-balancer-ssl-cert: arn:aws:acm:ap-south-1:730335348812:certificate/16966c84-1872-45f8-8dac-66ceacf58cf2
              service.beta.kubernetes.io/aws-load-balancer-backend-protocol: ssl
              service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "9093"
          brokers:
            - broker: 0
              annotations:
                service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
                service.beta.kubernetes.io/aws-load-balancer-ssl-cert: arn:aws:acm:ap-south-1:730335348812:certificate/16966c84-1872-45f8-8dac-66ceacf58cf2
                service.beta.kubernetes.io/aws-load-balancer-backend-protocol: ssl
                service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "9093"

    authorization:
      type: simple  # <-- Add this block to enable ACLs
    storage:
      type: persistent-claim
      size: 100Gi
      class: efs-sc
      deleteClaim: false

  zookeeper:
    replicas: 1
    storage:
      type: persistent-claim
      size: 10Gi
      class: efs-sc
      deleteClaim: false

  entityOperator:
    topicOperator: {}
    userOperator: {}
