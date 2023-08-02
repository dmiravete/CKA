# Creating users in kubernetes

## Key Based

[Example here](https://dev.to/technovangelist/how-to-create-users-in-kubernetes-40p0)

```
# Key Creation
openssl genrsa -out diego.key
```

Then, we create the certtificate request with the key. *Important!* It's necesary add the CN with the name of the user ir group.

```
openssl req -new -key admin.key -subj "/CN=diego" -out diego.csr
```

Creates a CertificateSigningRequest in kubernetes adding the CSR in base64 to the request.

```
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: diego
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZUQ0NBVDBDQVFBd0VERU9NQXdHQTFVRUF3d0ZaR2xsWjI4d2dnRWlNQTBHQ1NxR1NJYjNEUUVCQVFVQQpBNElCRHdBd2dnRUtBb0lCQVFDekZaVHZ5TS9QTUV2ajltMzVac2F2dXBsdFdQdEI3b2Vzd050TkZqdXcwemhPClpEK1ZXRWgzT1RLb0VaU25RWVFZcmFlRks5aUNaZHUyVk9MR2pIUGFGVmJqTmRVRnVVdTVvbUx4OUJtWTdoSmkKRXpsckJCbnNzQ3M3QkwrWFdNQzNhN1FPcGFJS3R0dDNQRWpqRUFBMHBDeDFtcWRPR2htQk5nUzliOG1uR2IybgpReFkwdzZ6bTlrNWFjbGh3U3Ria0poSVB4Y2VpRkp1TlRYNE8xbjh3aDlJeUt0Zmg0WVpvWHU4WWNsL2hhZTNDCkNtV210WTd2SENlVGxJRzZSdnM3R0ZYYVEzbUdtYkhSMHoyNG8xU1djTHV6aXRSaWJGaGVRRW8zaTM5Qk5scU8KWk9WakNBdGlzQUVDVFlVUVgwT0JMTmdHcWZKc25McURXK0lzeThqbEFnTUJBQUdnQURBTkJna3Foa2lHOXcwQgpBUXNGQUFPQ0FRRUFRQjk3b0p5N2hlV2hFNHlOZ0dRSFl0MSsycDF6UW53YjRJaHhyamFDY1FybTZHODJVbUZBCkRTK21FVUV0MmI1SnN2SlUyMmZjRG5LcUxjWndKTG4vNGFVUnA1WjlJVWRnaVlEQWlLZEVUNzhQcGFPbUQxMm4Kb28rSWVaZ3dyOU5WQUg2VnBLL2hwU21VZEs2VG9aUndubERrM2F5TlhIVDVnSkFGM0NMRndkeTlseGszZFBucgpzcm4wZngvZmp5cFVoRS9uQVFreUFVVFFCd2hTMEI5SDRlbHV2WTh2RFowbkZaTFI4cWxCRTQ0N3hndnJJMnBsCnRnZmtWWlRwWGhVNTFmVUY0OUJOUUZlVUkrbnJxVmVwdXUyZEdFVEZYQmZOWXk4WCt4ZWIyT0hzYlFodGFqSjQKQmI5S3oyMVNpM2ZJcjFmYm5COTZRZGMvdUhHNG9SemZhUT09Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # one day
  usages:
  - client auth
EOF
```

Now we can view and approve the CSR:
```
# List the csr
k get csr

# Approve the csr
k certificate approve diego
```

We can extract our certficate and save in a file:
```
k get csr diego -o jsonpath='{.status.certificate}'| base64 -d > diego.crt
```

With the key and the cert you can add the user to the kubeconfig. You can do that modifying manually the file or with this command:
```
# Add user to the kubefile
k --kubeconfig kubeconfig config set-credentials diego --client-key=diego.key --client-certificate=diego.crt --embed-certs=true

# Add the context
kubectl --kubeconfig kubeconfig config set-context diego --cluster=kind-kind26 --user=diego
```


### How to review certificates
```
openssl x509 -in mycert -text -noout
```

## Key Based Admin

[Example here](https://www.frakkingsweet.com/adding-a-full-admin-user-in-kubernetes/)

```
# Key Creation
openssl genrsa -out kadmin.key
```

Then, we create the certtificate request with the key. *Important!* It's necesary add the CN with the name of the user and group.

```
openssl req -new -key kadmin.key -subj "/CN=kadmin/O=system:masters" -out kadmin.csr
```

If you try to create a CSR Kubernetes return a error:
```
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: kadmin
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ2J6Q0NBVmNDQVFBd0tqRVBNQTBHQTFVRUF3d0dhMkZrYldsdU1SY3dGUVlEVlFRS0RBNXplWE4wWlcwNgpiV0Z6ZEdWeWN6Q0NBU0l3RFFZSktvWklodmNOQVFFQkJRQURnZ0VQQURDQ0FRb0NnZ0VCQUt5OS9XZlRqSWdFCm5mYnRrZmhuQktSRU91cGp4cHovT0VHL3BTVkN6aWRBKzV0V1FVR2dVdUxlRlV2d0Vkd2tqa3NWOG1IcjUrSTYKdXBMUDN0TExkRnE0TkM2YzZUdWNDMnpzQWxwNHFDQW9qaFA1enI5d0ZOS3ZOSzkzV2tOWnhiTW1LTEpFbGNqQgpVbEQ5NUt1V3JhSWxDaEVnZWxUeDRqcjB2c0E4RlgyeDEyWmNQS0xDclh6eUZrL1RObCtGZ2tvUFhaUjl3UnJpCmZjSitjcGRnRkVqS0hBcjNpbnpjWmJ6NnkwLzZJZlRDODNLNUFMZFg5QWVaNk93YUtDM3RjTHJOeW1MbHgzNUMKcS80djR4eWZmeXNIZFZWM2ZsMHNrWUhqRGlKU1gzQXhFSDVxT2kwdG04TW5FaGJjN2FQM3lQZVRZcDRqbURpbwo3MWU0L0Z5blM2MENBd0VBQWFBQU1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQThsMk9pa3FnSHB3MlhNV2d0CjlkblNyNExkQUpuZUJXaGFWOHJyNDZISDE4T3UvNjIrUzVZNDB2MnBsVGhaZWcxTWhFa290eTBMM1dVdGxvTG0KYUVwQWJMZFJTY05DSkhiTWsrWDNLL1dLK2NxazFSU1JXcnRLY1ZGZFFrbUVXcGt4K2NsdEJOZHRVcWdJSTdBYQpPYjBzT3BGLy9aK21lcXQwKzRXbnBkdDFRTEpGRUZNNFA4ZHQyOGJjZ0pWNFFveEswL0M0cVZNWGgxSm4wNUhuCmZQOWVWNkFaUjRnM296ditDYUVsVW8rbmFGODE2QkVCQ0xSUC9XTXJtTkwvVlN5bGlLUjN0eVZBdUxGQkNJM3EKSitGSVlobG1KaWZOblVPQXc2ajd4NE9LT1RNNUY3NWdGbEtTL1J4c28ySi8xQWxDazdGOUNSWmJDYk1VZGg5SQpOYkJkCi0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # one day
  usages:
  - digital signature
  - key encipherment
  - client auth
  groups:
  - system:authenticated
EOF

Error from server (Forbidden): error when creating "STDIN": certificatesigningrequests.certificates.k8s.io "kadmin" is forbidden: use of kubernetes.io/kube-apiserver-client signer with system:masters group is not allowed
```
You need to connect to a master node and do that (you need to move or create the csr in the master node) like in [this example](https://medium.com/@JoooostB/kubernetes-how-to-create-a-system-masters-user-and-why-you-really-shouldnt-8c17d19e7b8e):

```
openssl x509 -req -in kadmin.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out kadmin.crt
```

## File based

[Example here](https://techexpertise.medium.com/setting-up-basic-authentication-for-kubernetes-cluster-on-minikube-1-e84e1b56c64)



Create the folder and the file with the user information. This file is a CSV “user-details.csv”  with three columns, password, username and userid respectively:

```
mkdir /tmp/users

cat > /tmp/users/user-details.csv
123456,diegos,diegos1234
```

Then is necesary to modify the manifest yaml of the kube-apiserver in order to add:

```
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 10.89.0.4:6443
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=10.89.0.4
    - --allow-privileged=true
    # To authenticate with a file needs "Node"
    - --authorization-mode=Node,RBAC
    ....
    # Configure the property pointing to the file path
    - --basic-auth-file=/tmp/auth/user-details.csv
    volumeMounts:
    # Mount the volume
    - mountPath: /tmp/auth
      name: user-authentication
      readOnly: true
    ....
  volumes:
  # Mount the volume
  - hostPath:
      path: /tmp/users/
      type: DirectoryOrCreate
    name: user-authentication

```

In order to test the API:

```
curl -kv https://127.0.0.1:57820/api/v1/namespaces/default/pods -u "diegos:123456"
```

## Groups

