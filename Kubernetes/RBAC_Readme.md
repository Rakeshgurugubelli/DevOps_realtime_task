**RBAC: Role based access control**

**🧠 Simple Explanation:**

RBAC controls who can do what in your Kubernetes cluster.

**📦 Think of it like this:**

**User:** a person or service (like a CI/CD pipeline).


**Role:** a set of permissions (e.g., "can read pods", "can create services").


**Binding:** connects a user to a role.

**🔐 Example:**
Let’s say you have:

A developer who should only view pods.

An admin who can do everything.

You create:

**A Role:** "pod-reader" → allows get, list, watch on pods.

**A RoleBinding:** links the developer to the "pod-reader" Role.

Now the developer can only view pods — not delete or create them.

**📁 RBAC resources:**

**Role:** defines permissions within a namespace.

**ClusterRole:** defines permissions cluster-wide.

**RoleBinding:** applies a Role to a user/group in a namespace.

**ClusterRoleBinding:** applies a ClusterRole across the whole cluster.


**🤖 What is a Service Account in Kubernetes?**

A Service Account is an identity used by pods (applications) to interact with the Kubernetes API.

Just like users need permission to access resources, service accounts do too — and that's where RBAC roles and permissions come in.

**🔧 Step-by-Step: RBAC with Certificate Authentication**

Step-1: Generate Private Key and CSR

openssl genrsa -out dev-user1.key 2048

openssl req -new -key dev-user1.key -out dev-user.csr -subj "/CN=dev-user/O=dev-group"

**CN=dev-user will be the username, and O=dev-group is the group.**

Step-2: for the base64 encoded ecr 

cat dev-user1.csr | base64 | tr -d '\n'

![image](https://github.com/user-attachments/assets/5c3bf54b-1bf7-4d85-811c-752e8a99de51)

Step-3:  Create a Kubernetes CSR Object

# dev-user-csr.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: dev-user1
spec:
  request: <base64-encoded-csr>
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth

Step-4: Apply the manifest file dev-user-csr.yaml 

kubectl apply -f dev-user-csr.yaml

![image](https://github.com/user-attachments/assets/14416046-dcf3-4acf-a527-7bb2014a293e)

Step-5: Approve the CSR

kubectl certificate approve dev-user1

![image](https://github.com/user-attachments/assets/f2d20cbd-a905-4003-8802-9da47b7d1e10)

Step-6: Downloaded the signed certificate

kubectl get csr dev-user1 -o jsonpath='{.status.certificate}' | base64 -d > dev-user1.crt

Step-7: To view kubeconfig 

cat ~./kube/config

Step-8: configure kubectl for the user

kubectl config set-credentials dev-user1 \
  --client-certificate=dev-user1.crt \
  --client-key=dev-user1.key

![image](https://github.com/user-attachments/assets/559b3729-81b1-4fdc-8eea-79e070f6146f)

![image](https://github.com/user-attachments/assets/8fa5a94c-94e2-4ce0-97b8-81b8ce797846)

kubectl config set-context dev-user1-context \
  --cluster=chromosome-cluster \
  --namespace=default \
  --user=dev-user1

![image](https://github.com/user-attachments/assets/6096b4a5-1f8f-46d3-b33e-e9bbc56b8439)

Step-9: check the available cluster 

kubectl config get-clusters

kubectl config get-contexts

kubectl config use-context dev-user1-context

Step-10: check able to Switch to the dev user, again to admin user

kubectl config use-context dev-user1-context
![image](https://github.com/user-attachments/assets/10f81c77-7d53-4b73-a4f2-cfa6816bcec8)

Step-11: Create a role 

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: dev-role
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]

Step-12: creat a role binding

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-group-viewer
  namespace: default
roleRef:
  kind: Role
  name: dev-role
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: Group
  name: dev-group  # matches the O field in the cert
  apiGroup: rbac.authorization.k8s.io

Step-13: check the authentication

kubectl config use-context arn:aws:eks:ap-south-1:725157737674:cluster/chromosome-cluster


**cluster role and custer role binding**

Step-1: Generate Private Key and CSR

openssl genrsa -out dev.key 2048

openssl req -new -key dev.key -out dev.csr -subj "/CN=dev"

**CN=dev-user will be the username, and O=dev-group is the group.**

Step-2: for the base64 encoded ecr 

cat dev.csr | base64 | tr -d '\n'

![image](https://github.com/user-attachments/assets/ecfb3125-3881-41cd-b6ba-fe610db3b5b1)

Step-3:  Create a Kubernetes CSR Object

# dev-csr.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: dev-user1
spec:
  request: <base64-encoded-csr>
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth

Step-4: Apply the manifest file dev-user-csr.yaml 

kubectl apply -f dev-csr.yaml

![image](https://github.com/user-attachments/assets/14416046-dcf3-4acf-a527-7bb2014a293e)

Step-5: Approve the CSR

kubectl certificate approve dev-user1

![image](https://github.com/user-attachments/assets/f2d20cbd-a905-4003-8802-9da47b7d1e10)

Step-6: Downloaded the signed certificate

kubectl get csr dev-user1 -o jsonpath='{.status.certificate}' | base64 -d > dev-user1.crt


  
