---
title: Setup Oauth on Kind Cluster
weight: 3
---

# Setup Github Oauth
For setting Ouath we would need a domain name and SSL certificate and then we will configure the following 

`https://<YOUR_DOMAIN_NAME>.com/dex/` -> Dex service for authenticating with Github  
`https://<YOUR_DOMAIN_NAME>.com/dex-auth/` -> A service which generate keys for accessing cluster

## Setup Kind cluster that enable OIDC
Create a new `kind` cluster with following configuration

```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: origin-dev
networking:
  apiServerAddress: "62.138.26.145"
  apiServerPort: 6443
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: ClusterConfiguration
    apiServer:
      extraArgs:
        oidc-ca-file: /etc/ssl/certs/oidc-ca.pem
        oidc-client-id: dex-k8s-authenticator
        oidc-groups-claim: groups
        oidc-issuer-url: https://<YOUR_DOMAIN_NAME>/dex
        oidc-username-claim: email
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraMounts:
  - hostPath: /root/ca_bundle.crt
    containerPath: /etc/ssl/certs/oidc-ca.pem
    readOnly: true
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
```
## Setup Github Org and Oauth App

1. Go to Github page, create a new org.
2. Go to new org settings -> Developer settings -> OAuth Apps
3. Create `New OAuth App`, 
* Application name: what-ever-you-like
* Homepage URL: https://what-ever-you-like
* Authorization callback URL: `https://<YOUR_DOMAIN_NAME>/dex/callback`
  
After create new Oauth App, you will see `Client ID` and `Generate a new client secret` -> click to generate a new client secret

## Setup Certificate for domain k8s-origin.devops-learning.com

Unzip `<YOUR_DOMAIN_NAME>.zip` and run following command to create tls certificate
```
kubectl create secret tls dex-tls --cert=certificate.crt --key=private.key
```
## Setup ClusterRole and ClusterRole binding for login user.
Create `cluster-read-all` cluster role
```
cat << 'EOF' | kubectl apply -f - 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-read-all
rules:
- apiGroups:
    - ""
    - apps
    - autoscaling
    - batch
    - extensions
    - policy
    - rbac.authorization.k8s.io
    - storage.k8s.io
  resources:
    - componentstatuses
    - configmaps
    - cronjobs
    - daemonsets
    - deployments
    - events
    - endpoints
    - horizontalpodautoscalers
    - ingress
    - ingresses
    - jobs
    - limitranges
    - namespaces
    - nodes
    - pods
    - pods/log
    - pods/exec
    - persistentvolumes
    - persistentvolumeclaims
    - resourcequotas
    - replicasets
    - replicationcontrollers
    - serviceaccounts
    - services
    - statefulsets
    - storageclasses
    - clusterroles
    - roles
  verbs:
    - get
    - watch
    - list
- nonResourceURLs: ["*"]
  verbs:
    - get
    - watch
    - list
- apiGroups: [""]
  resources: ["pods/exec"]
  verbs: ["create"]
EOF
```
Create Cluster role binding for specific user or group from github  
Change `zduymz to your github username.
```
cat << 'EOF' | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: dex-cluster-auth
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-read-all
subjects:
  - kind: User
    name: <EMAIL_ID_1>
    apiGroup: rbac.authorization.k8s.io
  - kind: User
    name: <EMAIL_ID_2>
    apiGroup: rbac.authorization.k8s.io
EOF
```

## Setup Dex and Dex-Authenticator
Create a new file `dex.yaml` with following content. 
**Change** `GITHUB_CLIENT_ID` and `GITHUB_CLIENT_SECRET` `GITHUB_GROUP` to what you created in previous step.
```
ingress:
  enabled: true
  hosts:
    - host: <YOUR_DOMAIN_NAME>.com
      paths:
        - path: /dex
          pathType: ImplementationSpecific
  tls:
   - secretName: dex-tls
     hosts:
       - k8s-origin.devops-learning.com
config:
  issuer: https://<YOUR_DOMAIN_NAME>/dex
  storage:
    type: sqlite3
    config:
      file: /var/dex/dex.db
  web:
    http: 0.0.0.0:5556
  frontend:
    theme: "coreos"
    issuer: "Example"
    issuerUrl: "https://example.com"
    logoUrl: https://example.com/images/logo-250x25.png
  expiry:
    signingKeys: "6h"
    idTokens: "24h"
  logger:
    level: debug
    format: json
  oauth2:
    responseTypes: ["code", "token", "id_token"]
    skipApprovalScreen: true
  connectors:
  - type: github
    id: github
    name: GitHub
    config:
      clientID: GITHUB_CLIENT_ID
      clientSecret: GITHUB_CLIENT_SECRET
      redirectURI: https://<YOUR_DOMAIN_NAME>/dex/callback
      orgs:
      - name: GITHUB_GROUP
      loadAllGroups: false
  staticClients:
  - id: dex-k8s-authenticator
    name: dex-k8s-authenticator
    secret: ac80f645bced928d60c8493f29ea3120
    redirectURIs:
    - https://<YOUR_DOMAIN_NAME>.com/dex-auth/callback/<CLUSTER-NAME>
  enablePasswordDB: True
  staticPasswords:
  - email: "admin@example.com"
    hash: "$2a$10$2b2cU8CPhOTaGrs1HRQuAueS7JTT5ZHsHSzYiFPm1leZck7Mc8T4W"
    username: "admin"
    userID: "08a8684b-db88-4b73-90a9-3cd1661f5466"
```
Create a new file `dex-auth.yaml` as following content  
**Change** `k8s_master_uri` to your kind cluster uri, `k8s_ca_pem_base64_encoded` to you kind cluster CA base64 encoded certificate. You can those infomation from `kubectl config view --raw=true`
```
image:
  repository: <your_registry>/dex-k8s-authenticator
  tag: "1.0.0"
config:
  insecure_skip_verify
  web_path_prefix: /dex-auth
  clusters:
  - name: "<CLUSTER-NAME>"
    short_description: "My Cluster"
    description: "Example Cluster Long Description..."
    issuer: https://<YOUR_DOMAIN_NAME>.com/dex
    client_id: dex-k8s-authenticator
    client_secret: ac80f645bced928d60c8493f29ea3120
    k8s_master_uri: https://<KUBE_API_ADDRESS>:6443
    redirect_uri: https://<YOUR_DOMAIN_NAME>.com/dex-auth/callback/<CLUSTER-NAME>
    k8s_ca_pem_base64_encoded: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeU1USXdNakF4TURFd05Gb1hEVE15TVRFeU9UQXhNREV3TkZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTFB3CkJNTTUxNTZRaU05MFM0Q1p6YjQxOEFCRjhmQUNCRlAzRUd6Nk41MEp4RGFhcElxWXdFd1hDbEFQbXNkUFI5M2cKS2NDOTE4Y29EVUplSm5HMDQ5V3IyVUlqczdRai9vcU43WmFuMlB1amMwYnZSdWRMdjdIL3ZFVzZsbGdZbXBEWAptTndqZWU3TkhPVTU5Sm81OFp5Vm55UFZkdVpLeTh4NmpvTStZQzRaWjRsRDhQRVVUZUM2Zms1UlRYdnExVmwzCkNNK1BMYTY5QTJUVWx1bUZPcE54U0V3UFBzaE1oSlBQTnhoM0diOHA5SFFSQU82NSthQXFBSm5hVUUwaFhvYlUKbU5MUjlIZFltZFhCNkZxZjU1R2pGZGJxM2QrRUFlTW9ycThIVGlSWk9GR21Oa203TjhHdGpRZEpMb1FZeDQwMQppYmcvekU1NTZQQlp3M0VxL0dVQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZBc2dkQkRXSERYNk5INjBtenJveVZTZWdXTTZNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFDSUt3aVAvbmJwdUJZUGdmQUtRMVVuUTRtQjBsZnQzOFU3NEt1b2hRUHZIVWVGQnNpegpMZ1Vaa1BPVGFkZ3dTM2ZEQUZBd1hoMjNiYy9wZFBqOFNMUUNsbDVYVll1d21Ib2xMaGpwclp2QzYrVDRIemdMCnhJbTJMRjA3NHgvRmM3c1FhdFJxQWlxTTVSdXc5blRCQnIzT3NoTFR2YzZ2UTBKVWtQeXJ2bDM1MjlWeHhEN2cKTFl3ekova0dmWmpmclk1QlV4ZUd5eVlPMmlUeFIydjh0UnZscitnS0pNTkRpWCtVZ093TTQzWDc2N0p1UW1pawpVMHhLU0g0ano2Mk9QcklrOHVBeE05c0JJTWRnY3RDTkpBMERXMDhvdzRqZVVKVUhJQzRDMjFndmtnMTV6aVNyCjVKOU5JWlJ3TnJFWGR2YTc4eGJzelVacGNoMnV6MENMTkMxdQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==

ingress:
  enabled: true
  className: nginx
  hosts:
    - host: <YOUR_DOMAIN_NAME>.com
      paths:
        - path: /dex-auth
          pathType: ImplementationSpecific
  tls:
    - secretName: dex-tls
      hosts:
        - k8s-origin.devops-learning.com
```
Install `dex` and `dex-auth`
```
helm repo add dex https://charts.dexidp.io
helm repo add skm https://charts.sagikazarmark.dev
helm repo update

helm upgrade --install dex -f dex.yaml dex/dex
helm upgrade --install dex-auth -f dex-auth.yaml skm/dex-k8s-authenticator
```

After install successfully, you can access `https://<YOUR_DOMAIN_NAME>.com/dex-auth/login`