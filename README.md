Example Create User Kubernetes (Role and RoleBinding)
----------------

This an example of how to create user and atach Role:


1. Create a client certificate

    mkdir certs && cd certs

2. Create a namespace

    kubectl create ns developers

3. Create a private key for your user

    openssl genrsa -out {USER}.key 2048
    
4. Create a certificate sign request employee.csr using the private key you just created (robel.key in this example). Make sure you specify your username and group in the -subj section 

    openssl req -new -key {USER}.key -out {USER}.csr -subj "/CN={USER}/O=developers"

5. Generate the final certificate by approving the certificate signing request

    openssl x509 -req -in {USER}.csr -CA /var/lib/rancher/k3s/server/tls/client-ca.crt -CAkey /var/lib/rancher/k3s/server/tls/client-ca.key -CAcreateserial -out {USER}.crt -days 7
    
6. Add a new context with the new credentials for your k8s cluster

    kubectl config set-credentials {USER} --client-certificate={USER}.crt --client-key={USER}.key
    kubectl config set-context {USER}-context --cluster={NAME_CLUSTER} --namespace=developers --user={USER}
    
7. Now you should get access denied error when using kubectl CLI with this configuration file as we have not defined yet any permitted operations for this user. Try kubectl --context=robel-context get pods Create the role for managing deployments

    kind: Role
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      namespace: developers
      name: role-manager
    rules:
      - apiGroups: ["", "extensions", "apps"]
        resources: ["deployments", "replicaset", "pods", "*/log", "services", "endpoints"]
        verbs: ["get", "list", "watch", "patch", "update"] # You can also use ["*"]
        
8. Bind the role to the robel user

    kind: RoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      namespace: developers
      name: role-manager-binding
    subjects:
      - kind: User
        name: {USER}
        apiGroup: ""
    roleRef:
       kind: Role
       name: role-manager
       apiGroup: ""
       
9. Apply files .yaml

    kubectl apply -f .
    
10. Test the RBAC rule

    kubectl --context={USER}-context run --image=nginx nginx
    kubectl --context={USER}-context get pods
    kubectl --context={USER}-context get svc
