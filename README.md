Example Create User Kubernetes (Role and RoleBinding)
----------------

This an example of how to create user and atach Role:


1. Create a client certificate

    mkdir cert && cd cert

1.1. Generate a key using OpenSSL

    openssl genrsa -out {USER}.key 2048

1.2. Generate a Client Sign Request (CSR)
   
    openssl req -new -key {USER}.key -out {USER}.csr -subj /CN={USER}/O={GROUP}

1.3. Generate the certificate (CRT)

    openssl x509 -req -in {USER}.csr -CA /var/lib/rancher/k3s/server/tls/server-ca.crt -CAkey /var/lib/rancher/k3s/server/tls/server-ca.key -CAcreateserial -out {USER}.crt -days 7

2. Create your User

2.1. set yout user entry in kubeconfig

    kubectl config set-credentials {USER} --client-certificate={USER}.crt --client-key={USER}.key

2.2. Set a context entry in kubeconfig

    kubectl config set-context {USER} --cluster=default --user={USER}

You can check that it is successfully added to kubeconfig:

    kubectl config view

2.3. Switching to the created user

    kubectl config use-context {USER}

    kubectl config current-context #Check current user

    kubectl create namespace ns-test #Test Permission your User

3. Grant access to the user

3.1. Create a Role

    kind: Role
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      namespace: default
      name: read-pods
    rules:
    - apiGroups: [""] # “” indicates the core API group
      resources: ["pods"]
      verbs: ["get", "watch", "list"]

3.2. Create a BindingRole

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

Switched to user default, has permission with administrator:

    kubectl config use-context default

Apply your Role and RoleBinding:

    kubectl apply -f role.yaml role-binding.yaml

We check that the Role and BindingRole was created successfully:

    kubectl get roles
    kubectl get rolebindings

4. Testing the allowed operations for user

    kubectl config use-context {USER}
    kubectl create namespace test # won't succeed, Forbidden
    kubectl get pods # this will succeed !

