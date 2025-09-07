Within a Kubernetes environment, there are different types of users:

- Human Users: Administrators and developers who perform administrative tasks and deploy applications.
- Service Accounts: Robot users that represent processes, services, or third-party applications.

```bash
#
password123,user1,u0001 password123,user2,u0002 password123,user3,u0003 password123,user4,u0004 password123,user5,u0005

kube-apiserver --basic-auth-file=user-details.csv
kube-apiserver --token-auth-file=user-token-details.csv

curl -v -k https://master-node-ip:6443/api/v1/pods --header "Authorization: Bearer KpjCVbI7cFAHYPkByTIzRb7gulcUc4B"
```

## TLS Kubernetes

ssh-keygen
cat ~/.ssh/authorized_keys
ssh -i id_rsa user1@server1

i go to google.com
google sends public key
browser uses pub key to generate symetric key
browser sends data and symetric key to google
google descrypts data and symmetric key using private key
following req, res are encrypted with symmetric key

:
openssl genrsa -out my-bank.key 1024
openssl rsa -in my-bank.key -pubout > mybank.pem
openssl genrsa -out ca.key 2048 openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
openssl genrsa -out admin.key 2048 openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
curl <https://kube-apiserver:6443/api/v1/pods> \ --key admin.key --cert admin.crt --cacert ca.crt

- --key-file=/path-to-certs/etcdserver.key - --cert-file=/path-to-certs/etcdserver.crt
- --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt

kube-apiserver
Then, create an OpenSSL configuration file (e.g., openssl.cnf) to include all necessary SANs
After configuring the CSR with the SANs, sign the certificate using your CA certificate and key.openssl req -new -key apiserver.key -subj "/CN=kube-apiserver" -out apiserver.csr

openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
journalctl -u etcd.service -l
