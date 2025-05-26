**Installing kafka on eks using strimzi**

Step-1: create a namespace of kafka or strimzi based on requirement

![image](https://github.com/user-attachments/assets/c9d34afb-f009-42e4-863e-6f419bae6992)

Step-2: Install Strimzi Operator in the kafka namespace.

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

