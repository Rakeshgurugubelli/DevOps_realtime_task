
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

**Step2** Add admin access to the user(by using below command we can do admin access)

command:

        usermod -aG sudo rakesh

**Step-3:** create a directory in the tmp folder or any folder based on requirement and go inside the folder

command:

        mkdir /tmp/rbac

![image](https://github.com/user-attachments/assets/a1afe566-a263-454d-a981-e40bbdb2de2c)





