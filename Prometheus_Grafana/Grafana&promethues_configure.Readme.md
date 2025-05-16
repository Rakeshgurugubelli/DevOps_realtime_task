**How to add datasources**

Step-1: Login into grafana

Step-2: Click on the Data sources under connections, and click on Add new data source and select prometheus

![image](https://github.com/user-attachments/assets/b57bfc4a-9022-46c3-852a-c0c53c463488)

![image](https://github.com/user-attachments/assets/d80f131f-a292-462a-8886-1bcd6f0e0818)

Step-3: click on the prometheus 1, change the name and copy the prometheus url

![image](https://github.com/user-attachments/assets/a1d1237a-e8f4-4f95-81dc-adf74f70f59c)

**how to the create the dashboard**

Step-1: Login into grafana

Step-2: Click on the dashboards, and click on New and select import

![image](https://github.com/user-attachments/assets/a7ffb414-9fa8-4748-b7cd-41ec31ae58b5)

Step-3: In search serach with 14513 and click on Load, select the datasource of the prometheus and click on Import

![image](https://github.com/user-attachments/assets/4019bc4f-56e5-4d26-b026-3674541d73d7)

**how to create custom metrices**

Step-1: login into the aws terminal

Step-2: Expose the application and verify it is access through browser

**our requirement we exposed our application using load balancer with port of 3001**

![image](https://github.com/user-attachments/assets/0167f1cb-921a-471f-9d3a-2057910321d9)

Step-3: verify in the browser when we search domain name/metrics metrics is coming or not(developer need to configure)

![image](https://github.com/user-attachments/assets/10dcf4c2-09f6-45ae-b9c0-ceeeff58a9f3)

Step-4: if its coming , the configure service discovery run the below manifest it will configure(service monitor)

**based on the requirement change app & path & namespaces fields**

apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: a-service-servicemonitor
  namespace: dev
  labels:
    release: prometheus
spec:
  selector:
    matchLabels:
      app: a-service
  endpoints:
    - port: http
      interval: 30s
      path: /metrics
  namespaceSelector:
    matchNames:
      - dev

  ![image](https://github.com/user-attachments/assets/e0b05b51-d2cd-45cf-ae7f-ad0b0ed10b0c)

Step-5: Go to promethues Server verify the customer metrics properly configure to prometheus

host or dns_name:9090/targets verify the service is present

![image](https://github.com/user-attachments/assets/f65cf0b0-906e-4055-9475-cb5b8624a7e1)

Step-6: After that go to promql and execute the commands

Ex: http_requests_total(this will get from developer, developer will configure in code)

![image](https://github.com/user-attachments/assets/1fa3579b-2f2e-4987-957e-9db5c3f21519)

Step-7: After that go to grafana and create custom dashboards

  1.) Go to grafana click on dashboards-> click on new dashboard

  ![image](https://github.com/user-attachments/assets/84e98cab-d2ee-4f5c-a51d-77744213e566)

  2.) Click on Add visualization, and select the data source

  ![image](https://github.com/user-attachments/assets/b6cec77a-4ba9-428a-b1f1-9d6e951c71b2)

  3.) search the promql query in the mertic and click on run queries

  query: http_requests_total

  ![image](https://github.com/user-attachments/assets/15d5a5db-f41d-47b1-9759-643527238a39)

   4.) After that we will see a graph and click on save dashboard our custom dashboard is created

  ![image](https://github.com/user-attachments/assets/66a0f679-ef98-4cfa-b4e0-0708e2e5986f)

**How to configure Alerts**

Step-1: login into the aws terminal

Step-2: Expose the application and verify it is access through browser

**our requirement we exposed our application using load balancer with port of 3001**

![image](https://github.com/user-attachments/assets/0167f1cb-921a-471f-9d3a-2057910321d9)

Step-3: verify in the browser when we search domain name/metrics metrics is coming or not(developer need to configure)

![image](https://github.com/user-attachments/assets/10dcf4c2-09f6-45ae-b9c0-ceeeff58a9f3)

Step-4: if its coming , the configure service discovery run the below manifest it will configure(Alert management to gmail)

video: Abhishek veeramalla(Learn obervability in 5 hours)2:45:00 

alertmanagement.yaml---config file

apiVersion: monitoring.coreos.com/v1alpha1
kind: AlertmanagerConfig
metadata:
  name: main-rules-alert-config
  namespace: dev
  labels:
    release: prometheus
spec:
  route:
    repeatInterval: 30m
    receiver: 'null'
    routes:
    - matchers:
      - name: alertname
        value: HighCpuUsage
      receiver: 'send-email'
    - matchers:
      - name: alertname
        value: PodRestart
      receiver: 'send-email'
      repeatInterval: 5m
  receivers:
  - name: 'send-email'
    emailConfigs:
    - to: YOUR_EMAIL_ID
      from: YOUR_EMAIL_ID
      sendResolved: false
      smarthost: smtp.gmail.com:587
      authUsername: YOUR_EMAIL_ID
      authIdentity: YOUR_EMAIL_ID
      authPassword:
        name: mail-pass
        key: gmail-pass
  - name: 'null'

**Logging with EFK Stack:**

Log means mesages written by the developer to understand the enduser 

1.) why EFk and not ELK?

Ans. EFK and ELK process is sameprocess , but most of organizations prefer EFK, why because in EFK logs directly forwaded to Elastic search , 
     in ELK more logs we can  aggreator and send to elastic serach most of organization won't prefer on that reason we preffered EFK



E = Elastic search
F = fleunt bit
k = kibana

  **EFk is also called as FEk**

flunet bit it will collect the logs from the services and sends to elastic search, elastic search will store the logs and those logs will visual by using kibana

📝 EFK Step-by-Step Setup

1) Create IAM Role for Service Account
eksctl create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --namespace kube-system \
    --cluster observability \
    --role-name AmazonEKS_EBS_CSI_DriverRole \
    --role-only \
    --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
    --approve
   
This command creates an IAM role for the EBS CSI controller.
IAM role allows EBS CSI controller to interact with AWS resources, specifically for managing EBS volumes in the Kubernetes cluster.
We will attach the Role with service account

3) Retrieve IAM Role ARN
ARN=$(aws iam get-role --role-name AmazonEKS_EBS_CSI_DriverRole --query 'Role.Arn' --output text)

Command retrieves the ARN of the IAM role created for the EBS CSI controller service account.

5) Deploy EBS CSI Driver
eksctl create addon --cluster observability --name aws-ebs-csi-driver --version latest \
    --service-account-role-arn $ARN --force

Above command deploys the AWS EBS CSI driver as an addon to your Kubernetes cluster.
It uses the previously created IAM service account role to allow the driver to manage EBS volumes securely.

6) Create Namespace for Logging
kubectl create namespace logging

7) Install Elasticsearch on K8s
helm repo add elastic https://helm.elastic.co

helm install elasticsearch \
 --set replicas=1 \
 --set volumeClaimTemplate.storageClassName=gp2 \
 --set persistence.labels.enabled=true elastic/elasticsearch -n logging
 
Installs Elasticsearch in the logging namespace.
It sets the number of replicas, specifies the storage class, and enables persistence labels to ensure data is stored on persistent volumes.

6) Retrieve Elasticsearch Username & Password
# for username
kubectl get secrets --namespace=logging elasticsearch-master-credentials -ojsonpath='{.data.username}' | base64 -d
# for password
kubectl get secrets --namespace=logging elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d
Retrieves the password for the Elasticsearch cluster's master credentials from the Kubernetes secret.
The password is base64 encoded, so it needs to be decoded before use.
👉 Note: Please write down the password for future reference

7) Install Kibana
helm install kibana --set service.type=LoadBalancer elastic/kibana -n logging
Kibana provides a user-friendly interface for exploring and visualizing data stored in Elasticsearch.
It is exposed as a LoadBalancer service, making it accessible from outside the cluster.

9) Install Fluentbit with Custom Values/Configurations
👉 Note: Please update the HTTP_Passwd field in the fluentbit-values.yml file with the password retrieved earlier in step 6: (i.e NJyO47UqeYBsoaEU)"
helm repo add fluent https://fluent.github.io/helm-charts
helm install fluent-bit fluent/fluent-bit -f fluentbit-values.yaml -n logging

if fluentbit-values.yaml not create manually 

fluentbit-values.yaml



✅ Conclusion
We have successfully installed the EFK stack in our Kubernetes cluster, which includes Elasticsearch for storing logs, Fluentbit for collecting and forwarding logs, and Kibana for visualizing logs.
To verify the setup, access the Kibana dashboard by entering the `LoadBalancer DNS name followed by :5601 in your browser.
http://LOAD_BALANCER_DNS_NAME:5601
Use the username and password retrieved in step 6 to log in.
Once logged in, create a new data view in Kibana and explore the logs collected from your Kubernetes cluster.

🧼 Clean Up
helm uninstall monitoring -n monitoring

helm uninstall fluent-bit -n logging

helm uninstall elasticsearch -n logging

helm uninstall kibana -n logging


kubectl delete -k kubernetes-manifest/

kubectl delete -k alerts-alertmanager-servicemonitor-manifest/


eksctl delete cluster --name observability














