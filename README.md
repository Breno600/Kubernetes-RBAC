Example Create User Kubernetes (Role and RoleBinding)
----------------

This an example of how to create user and atach Role:


1. Create a client certificate

    mkdir cert && cd cert

1.1. Generate a key using OpenSSL

    openssl genrsa -out {USER}.key 2048

1.2. Generate a Client Sign Request (CSR)
   
    openssl req -new -key {USER}.key -out {USER}.csr -subj /CN={USER}/O={GROUP}
    
1.3. Create a CertificateSigningRequest

    apiVersion: certificates.k8s.io/v1
    kind: CertificateSigningRequest
    metadata:
      name: {USER}
    spec:
      groups:
      - system:authenticated
      request: #Execute this command in terminal $(cat john.csr | base64 | tr -d '\n') and past result here
      signerName: kubernetes.io/kube-apiserver-client
      usages:
      - client auth

1.4. Signing of Certificate

    kubectl certificate approve {USER}

1.5. Create a Role

    kind: Role
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      namespace: default
      name: read-pods
    rules:
    - apiGroups: [""] # “” indicates the core API group
      resources: ["pods"]
      verbs: ["get", "watch", "list"]

1.6. Create a BindingRole

    kind: RoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: read-pods
      namespace: default
    subjects:
    - kind: User
      name: {USER} # Name is case sensitive
      apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: Role #this must be Role or ClusterRole
      name: read-pods # must match the name of the Role
      apiGroup: rbac.authorization.k8s.io

1.7. Apply files yamls

    kubectl apply -f .
    
1.8. Set Credential 

    kubectl config set-credentials john --client-key={USER}.key --client-certificate={USER}.crt --embed-certs=true

1.9. Set Context user created

    kubectl config set-context {USER}

1.9. Testing the allowed operations for user

    kubectl config use-context {USER}
    kubectl create namespace test # won't succeed, Forbidden
    kubectl get pods # this will succeed !

