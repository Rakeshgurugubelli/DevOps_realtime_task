
**Authentication:** In Kubernetes, authentication validates the identity of a user or service account making a request to the API server.

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

**implementing the cluster and role binding for kubeadm and kops**

**Step-1:** Create a user & setup  password

command:  
        
         useradd rakesh 
         
         passwd rakesh

![image](https://github.com/user-attachments/assets/4e9c0875-6191-4b85-9120-293f33929dd2)

**Step2:** Add admin access to the user(by using below command we can do admin access)

command:

        usermod -aG sudo rakesh

**Step-3:** Create a directory in the tmp folder or any folder based on requirement and go inside the folder

command:

        mkdir /tmp/rbac

![image](https://github.com/user-attachments/assets/a1afe566-a263-454d-a981-e40bbdb2de2c)


**step-4:** Create a key for the user 

command:

       openssl genrsa -out rakesh.key 2048

![image](https://github.com/user-attachments/assets/54437390-e651-415d-a6e2-1a273f9abeb2)

**Step-5:** After create key need to create the certificate signing key
       
command:

          openssl req -new -key rakesh.key -out rakesh.csr -subj "/CN=rakesh/O=devops"

![image](https://github.com/user-attachments/assets/123ef10a-9967-4284-acb6-823be7d8c480)

**Step-6:** Create the certificate key using the user key and certificate signing key

command:

        openssl x509 -req -in rakesh.csr - CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out rakesh.crt -days 365

**implementing the cluster and role binding in EKS**

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

