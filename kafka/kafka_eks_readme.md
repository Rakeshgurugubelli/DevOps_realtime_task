**Installing kafka on eks using strimzi**

Step-1: create a namespace of kafka or strimzi based on requirement

![image](https://github.com/user-attachments/assets/c9d34afb-f009-42e4-863e-6f419bae6992)

Step-2: Install Strimzi Operator in the kafka namespace.

command: kubectl apply -f https://strimzi.io/install/latest?namespace=kafka -n kafka

![image](https://github.com/user-attachments/assets/b6aa8e0f-78a1-4801-a021-1ee50f0db14f)
