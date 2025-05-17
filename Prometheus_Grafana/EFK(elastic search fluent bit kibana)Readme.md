**Logging with EFK Stack:**

Log means mesages written by the developer to understand the enduser

1.) why EFk and not ELK?

Ans. EFK and ELK process is sameprocess , but most of organizations prefer EFK, why because in EFK logs directly forwaded to Elastic search , in ELK more logs we can aggreator and send to elastic serach most of organization won't prefer on that reason we preffered EFK

E = Elastic search F = fleunt bit k = kibana

EFk is also called as FEk

flunet bit it will collect the logs from the services and sends to elastic search, elastic search will store the logs and those logs will visual by using kibana

📝 EFK Step-by-Step Setup

Create IAM Role for Service Account eksctl create iamserviceaccount
--name ebs-csi-controller-sa
--namespace kube-system
--cluster observability
--role-name AmazonEKS_EBS_CSI_DriverRole
--role-only
--attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy
--approve
This command creates an IAM role for the EBS CSI controller. IAM role allows EBS CSI controller to interact with AWS resources, specifically for managing EBS volumes in the Kubernetes cluster. We will attach the Role with service account

Retrieve IAM Role ARN ARN=$(aws iam get-role --role-name AmazonEKS_EBS_CSI_DriverRole --query 'Role.Arn' --output text)
Command retrieves the ARN of the IAM role created for the EBS CSI controller service account.

Deploy EBS CSI Driver eksctl create addon --cluster observability --name aws-ebs-csi-driver --version latest
--service-account-role-arn $ARN --force
Above command deploys the AWS EBS CSI driver as an addon to your Kubernetes cluster. It uses the previously created IAM service account role to allow the driver to manage EBS volumes securely.

Create Namespace for Logging kubectl create namespace logging

Install Elasticsearch on K8s helm repo add elastic https://helm.elastic.co

helm install elasticsearch
--set replicas=1
--set volumeClaimTemplate.storageClassName=gp2
--set persistence.labels.enabled=true elastic/elasticsearch -n logging

Installs Elasticsearch in the logging namespace. It sets the number of replicas, specifies the storage class, and enables persistence labels to ensure data is stored on persistent volumes.

Retrieve Elasticsearch Username & Password
for username
kubectl get secrets --namespace=logging elasticsearch-master-credentials -ojsonpath='{.data.username}' | base64 -d

for password
kubectl get secrets --namespace=logging elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d Retrieves the password for the Elasticsearch cluster's master credentials from the Kubernetes secret. The password is base64 encoded, so it needs to be decoded before use. 👉 Note: Please write down the password for future reference

Install Kibana helm install kibana --set service.type=LoadBalancer elastic/kibana -n logging Kibana provides a user-friendly interface for exploring and visualizing data stored in Elasticsearch. It is exposed as a LoadBalancer service, making it accessible from outside the cluster.

Install Fluentbit with Custom Values/Configurations 👉 Note: Please update the HTTP_Passwd field in the fluentbit-values.yml file with the password retrieved earlier in step 6: (i.e NJyO47UqeYBsoaEU)" helm repo add fluent https://fluent.github.io/helm-charts helm install fluent-bit fluent/fluent-bit -f fluentbit-values.yaml -n logging

if fluentbit-values.yaml not create manually

fluentbit-values.yaml

✅ Conclusion We have successfully installed the EFK stack in our Kubernetes cluster, which includes Elasticsearch for storing logs, Fluentbit for collecting and forwarding logs, and Kibana for visualizing logs. To verify the setup, access the Kibana dashboard by entering the `LoadBalancer DNS name followed by :5601 in your browser. http://LOAD_BALANCER_DNS_NAME:5601 Use the username and password retrieved in step 6 to log in. Once logged in, create a new data view in Kibana and explore the logs collected from your Kubernetes cluster.

🧼 Clean Up helm uninstall monitoring -n monitoring

helm uninstall fluent-bit -n logging

helm uninstall elasticsearch -n logging

helm uninstall kibana -n logging

kubectl delete -k kubernetes-manifest/

kubectl delete -k alerts-alertmanager-servicemonitor-manifest/

eksctl delete cluster --name observability
