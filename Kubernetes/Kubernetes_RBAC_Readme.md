
**Authentication:** In Kubernetes, authentication validates the identity of a user or service account making a request to the API server.

Example: When you run kubectl, Kubernetes checks your credentials (like tokens or certificates) to verify your identity.

**Authorization** After authentication, Kubernetes checks if the user is authorized to perform the requested operation (like reading a pod or deleting a service).

Example: Even if your identity is verified, Kubernetes will deny access if you don’t have permission to delete a deployment.
