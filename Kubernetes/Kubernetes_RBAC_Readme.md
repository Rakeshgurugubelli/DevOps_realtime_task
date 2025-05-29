
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

**Below steps are implementing the cluster and role binding**

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

 **Step-4,5,6 is not required when we working eks directly go to step-7 if we are working with eks**

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

**Step-7:** Run  the below to edit configmap

command: 

        kubectl get configmap aws-auth -n kube-system -o yaml



