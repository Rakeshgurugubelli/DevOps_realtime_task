![image](https://github.com/user-attachments/assets/66a93f53-fca7-4d7e-8365-bfe19ff71547)![image](https://github.com/user-attachments/assets/ef3745a8-bddb-420e-b9eb-9bfd79272d83)**Authentication:** In Kubernetes, authentication validates the identity of a user or service account making a request to the API server.

Example: When you run kubectl, Kubernetes checks your credentials (like tokens or certificates) to verify your identity.

**Authorization** After authentication, Kubernetes checks if the user is authorized to perform the requested operation (like reading a pod or deleting a service).

Example: Even if your identity is verified, Kubernetes will deny access if you don’t have permission to delete a deployment.

**📁 RBAC resources:**

**Role:** defines permissions within a namespace.

**ClusterRole:** defines permissions cluster-wide.

**RoleBinding:** applies a Role to a user/group in a namespace.

**ClusterRoleBinding:** applies a ClusterRole across the whole cluster.

**🤖 What is a Service Account in Kubernetes?**

A Service Account is an identity used by pods (applications) to interact with the Kubernetes API.

Just like users need permission to access resources, service accounts do too — and that's where RBAC roles and permissions come in.

**implementing the cluster and role binding in EKS**-------**User**

  **First create a user and provide admin access to it**

**Step-1:** create a user either by using cli or command

![image](https://github.com/user-attachments/assets/2b0e23b1-7b64-4532-97c0-70c4e6676302)

**Step-2:** Copy the access key and secret key of the user and add admin access to the user

**Step-3:** Edit configmap by using below command and the user in the mapusers

       kubectl get configmap aws-auth -n kube-system -o yaml > current-aws-auth.yaml

**current-aws-auth.yaml**

apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapRoles: |
    - groups:
      - system:bootstrappers
      - system:nodes
      rolearn: arn:aws:iam::725157737674:role/eksctl-chromosome-cluster-nodegrou-NodeInstanceRole-kx0Wqhc0iijp
      username: system:node:{{EC2PrivateDNSName}}
  mapUsers: |
    - userarn: arn:aws:iam::725157737674:user/rakesh
      username: rakesh
      groups:
        - system:masters

      kubectl apply -f current-aws-auth.yaml
      
      aws eks update-kubeconfig --region ap-south-1 --name chromosome-cluster

**Step-4:** Run the below command and provide creditinals verify able to run kubectl commands

         aws configure

         aws sts get-caller-identity

![image](https://github.com/user-attachments/assets/61b3918c-0d20-4950-8c8f-ce4dd4ba9b37)


**After completing the above 4 steps recommended to start below steps**

**Step-5:** Create a namespace  based on requirement and run a pod

           kubectl create ns kafka

           kubectl get pod -n kafka

![image](https://github.com/user-attachments/assets/dd0833b7-05c3-44d0-9549-fe49718743f3)

**Step-6:** Create a user  and create acesskey using the cli or console

           aws iam create-user --user-name kafka-user

           aws iam create-access-key --user-name kafka-user


![image](https://github.com/user-attachments/assets/14484ea8-dade-4fcc-ba31-ab20e0d8fef0)

![image](https://github.com/user-attachments/assets/478852d4-885a-4772-8cd4-5d928858b4be)
           

**step-7:** Create a user in the terminal,login to that user and add access & secret key to the user

![image](https://github.com/user-attachments/assets/213ba94c-1706-4bea-bd28-68d55040047d)

**step-8:** Now login back to the root user and map an iam user to k8s

**current_aws_auth.yml**

apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapRoles: |
    - groups:
      - system:bootstrappers
      - system:nodes
      rolearn: arn:aws:iam::725157737674:role/eksctl-chromosome-cluster-nodegrou-NodeInstanceRole-kx0Wqhc0iijp
      username: system:node:{{EC2PrivateDNSName}}
  mapUsers: |
    - userarn: arn:aws:iam::725157737674:user/rakesh
      username: rakesh
      groups:
        - system:masters
    - userarn: arn:aws:iam::725157737674:user/kafka-user
      username: kafka-user

![image](https://github.com/user-attachments/assets/03f1f38f-d3cd-4823-a116-35599d4cfc21)


**Step-9:** Now login back to the user and the run the kubectl commands it will throw error(Expected)

**why because user we mapped in config but not provide the permissions**

![image](https://github.com/user-attachments/assets/5dfa50a7-6e5b-423d-a7b4-4b82a24ace6c)

**Step-10:** create a role and role binding to the user from the admin access 

**rbaccluster-role.yml**

# kafka-read-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: kafka
  name: kafka-pod-deploy-reader
rules:
- apiGroups: [""] #"" core api group
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["extensions","apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch"]

![image](https://github.com/user-attachments/assets/7f218805-8c0b-4fcb-888d-83ab626b2f20)

**rbaccluster-role-binding.yml**

# kafka-rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: kafka-reader-binding
  namespace: kafka
subjects:
- kind: User
  name: kafka-user            # This must match the 'username' in aws-auth
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: kafka-pod-deploy-reader
  apiGroup: rbac.authorization.k8s.io

![image](https://github.com/user-attachments/assets/48a429e5-344d-4808-b213-ef27185d107f)

Step-11:

      aws sts get-caller-identity

      
![image](https://github.com/user-attachments/assets/8d1a3a07-b503-470d-ac01-fe28df265d7b)


Step-12:

export AWS_ACCESS_KEY_ID=AKIA2RVWE6TFDE7FCR6Ehhh
export AWS_SECRET_ACCESS_KEY=5QxUWGQj2dCYN5JRf1tLikuoNnDYD21BLiK4pfov
export AWS_DEFAULT_REGION=ap-south-1


aws eks update-kubeconfig \
  --region ap-south-1 \
  --name chromosome-cluster \
  --kubeconfig /home/kafka/.kube/config

# Make sure the directory exists and has correct ownership
sudo chown -R kafka:kafka /home/kafka/.kube
sudo chmod -R 700 /home/kafka/.kube

![image](https://github.com/user-attachments/assets/d58cb166-0781-4fac-a147-60d1d9fb131c)

**implementing the cluster and role binding in EKS**-------**group**

**Step-1:** create a users

![image](https://github.com/user-attachments/assets/b0a67c98-34eb-4625-974b-3e5e705a5158)

**Step-2:** Copy the access and sceret of the user

![image](https://github.com/user-attachments/assets/06b11e38-bc45-456d-b7ed-b4b9d1d2af7c)

**Step-3:** Map an Iam user 

**current-aws-auth.yaml**

apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapRoles: |
    - groups:
        - system:bootstrappers
        - system:nodes
      rolearn: arn:aws:iam::725157737674:role/eksctl-chromosome-cluster-nodegrou-NodeInstanceRole-kx0Wqhc0iijp
      username: system:node:{{EC2PrivateDNSName}}
  mapUsers: |
    - userarn: arn:aws:iam::725157737674:user/rakesh
      username: rakesh
      groups:
        - system:masters
    - userarn: arn:aws:iam::725157737674:user/kafka-user
      username: kafka-user
      groups:
        - dev-group
    - userarn: arn:aws:iam::725157737674:user/kafka-user2
      username: kafka-user2
      groups:
        - dev-group
    - userarn: arn:aws:iam::725157737674:user/kafka-user3
      username: kafka-user3
      groups:
        - dev-group
    - userarn: arn:aws:iam::725157737674:user/kafka-user4
      username: kafka-user4
      groups:
        - qa-group

![image](https://github.com/user-attachments/assets/7013d866-a986-41bb-a1db-e8af7bc8b397)

**Step-4:** Create ClusterRole / Role and RoleBindings

**dev-role.yml**
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: kafka
  name: kafka-pod-reader-dev
rules:
- apiGroups: [""] #"" core api group
  resources: ["pods"]
  verbs: ["get", "list", "watch"]

![image](https://github.com/user-attachments/assets/9c739999-3ea3-4f4f-af4b-967494a54338)

**dev-role-binding.yml**

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-group-binding
  namespace: kafka
subjects:
- kind: Group
  name: dev-group
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: kafka-pod-reader-dev
  apiGroup: rbac.authorization.k8s.io

![image](https://github.com/user-attachments/assets/3b28326b-e828-43fd-b79f-a87342d9397e)

**qa-role.yml**
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: kafka
  name: kafka-pod-reader-qa
rules:
- apiGroups: ["extensions","apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch"]

![image](https://github.com/user-attachments/assets/b409b9fa-8524-4c7b-93d5-f8ad68d1fc1d)

**qa-role-binding.yml**

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: qa-group-binding
  namespace: kafka
subjects:
- kind: Group
  name: qa-group
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: kafka-pod-reader-qa
  apiGroup: rbac.authorization.k8s.io

![image](https://github.com/user-attachments/assets/941f7154-ddcb-43db-8d98-a43fe852a7cb)

**Step-5:** Now verify logged into the users

kafka-user2

![image](https://github.com/user-attachments/assets/40df5b8a-a688-428b-af8c-a52412a14b8c)

kafka-user3

![image](https://github.com/user-attachments/assets/0a323383-abb4-49eb-b99f-6286850392b1)

kafka-user4

![image](https://github.com/user-attachments/assets/c382aafc-7312-473f-88b6-d381d0084bd2)

**multiple groups**

apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapUsers: |
    - userarn: arn:aws:iam::123456789012:user/multi-role-user
      username: multi-role-user
      groups:
        - dev-group
        - qa-group
        - read-only

**particular pod**

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: kafka
  name: kafka-specific-pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  resourceNames: ["kafka-broker-0"]
  verbs: ["get", "list"]

