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

