**Grafana:**

-> it is open source tool for performing data analytics retrieving metrics that make of large amount of data and monitoring our apps using nice configurable dashboard.

-> so grafana can integrate with a wide range of data sources including graphite,prometheus influx TB elastic search mysql postgres and others.

-> so when connected to supported data sources it will provide web based chart graphs and alert

**promethus**

-> it is an open-source monitoring and alerting toolkit designed for reliability and scalability in model and dynamic environment.

-> which prometheus is excels at collecting and storing time series data allowing users to gain valuable insight into the performance and health of the application and infrastruture 

-> so its powerful query language and support for multi-dimensional data prometheus has become a popular choice for monitoring system with cloud native

**prequsite:**

Install and setup kubectl,eksctl on ubuntu server

Install and setup helm chart on ubuntu server


Step-1: Onces the kubectl, eksctl, helm and eks cluster created 

Step-2: Add helm stable charts for your local client

command: helm repo add stable https://charts.helm.sh/stable

![image](https://github.com/user-attachments/assets/437fbad8-b2af-4be2-bfba-aa9b3d0474d5)

Step-3: Add prometheus helm repository 

command: helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

![image](https://github.com/user-attachments/assets/3bfa4142-defb-4772-acf1-f4d34a601d31)

Step-4: create a namespace of prometheus

kubectl create ns prometheus

![image](https://github.com/user-attachments/assets/a10bd0ca-ba57-4727-97b9-0f078236dd0d)

Step-5: install prometheus using helm 

helm install prometheus prometheus-community/kube-prometheus-stack -n prometheus



