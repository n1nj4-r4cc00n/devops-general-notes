Start Minikube:

$ minikube start

View the content of the kubectl client's configuration manifest, observing the only context minikube and the only user minikube, created by default:

$ kubectl config view

apiVersion: v1
clusters:
- cluster:
  certificate-authority: /home/student/.minikube/ca.crt
  server: https://192.168.99.100:8443
  name: minikube
  contexts:
- context:
  cluster: minikube
  user: minikube
  name: minikube
  current-context: minikube
  kind: Config
  preferences: {}
  users:
- name: minikube
  user:
  client-certificate: /home/student/.minikube/profiles/minikube/client.crt
  client-key: /home/student/.minikube/profiles/minikube/client.key

Create lfs158 namespace:

$ kubectl create namespace lfs158

namespace/lfs158 created

Create the rbac directory and cd into it:

$ mkdir rbac

$ cd rbac/

Create a private key for the student user with openssl tool, then create a certificate signing request for the student user with openssl tool:

~/rbac$ openssl genrsa -out student.key 2048

Generating RSA private key, 2048 bit long modulus (2 primes)
.................................................+++++
.........................+++++
e is 65537 (0x010001)

~/rbac$ openssl req -new -key student.key -out student.csr -subj "/CN=student/O=learner"

Create a YAML manifest for a certificate signing request object, and save it with a blank value for the request field:

~/rbac$ vim signing-request.yaml

apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
name: student-csr
spec:
groups:
- system:authenticated
  request: <assign encoded value from next cat command>
  signerName: kubernetes.io/kube-apiserver-client
  usages:
- digital signature
- key encipherment
- client auth

View the certificate, encode it in base64, and assign it to the request field in the signing-request.yaml file:

~/rbac$ cat student.csr | base64 | tr -d '\n','%'

LS0tLS1CRUd...1QtLS0tLQo=

~/rbac$ vim signing-request.yaml

apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
name: student-csr
spec:
groups:
- system:authenticated
  request: LS0tLS1CRUd...1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  usages:
- digital signature
- key encipherment
- client auth

Create the certificate signing request object, then list the certificate signing request objects. It shows a pending state:

~/rbac$ kubectl create -f signing-request.yaml

certificatesigningrequest.certificates.k8s.io/student-csr created

~/rbac$ kubectl get csr

NAME          AGE   SIGNERNAME                            REQUESTOR       CONDITION

student-csr   12s   kubernetes.io/kube-apiserver-client   minikube-user   Pending

Approve the certificate signing request object, then list the certificate signing request objects again. It shows both approved and issued states:

~/rbac$ kubectl certificate approve student-csr

certificatesigningrequest.certificates.k8s.io/student-csr approved

~/rbac$ kubectl get csr

NAME          AGE   SIGNERNAME                            REQUESTOR       CONDITION

student-csr   57s   kubernetes.io/kube-apiserver-client   minikube-user   Approved,Issued

Extract the approved certificate from the certificate signing request, decode it with base64 and save it as a certificate file. Then view the certificate in the newly created certificate file:

~/rbac$ kubectl get csr student-csr -o jsonpath='{.status.certificate}' | base64 --decode > student.crt

~/rbac$ cat student.crt

-----BEGIN CERTIFICATE-----
MIIDGzCCA...
...
...NOZRRZBVunTjK7A==
-----END CERTIFICATE-----

Configure the kubectl client's configuration manifest with the student user's credentials by assigning the key and certificate:

~/rbac$ kubectl config set-credentials student --client-certificate=student.crt --client-key=student.key

User "student" set.

Create a new context entry in the kubectl client's configuration manifest for the student user, associated with the lfs158 namespace in the minikube cluster:

~/rbac$ kubectl config set-context student-context --cluster=minikube --namespace=lfs158 --user=student

Context "student-context" created.

View the contents of the kubectl client's configuration manifest again, observing the new context entry student-context, and the new user entry student:

~/rbac$ kubectl config view

apiVersion: v1
clusters:
- cluster:
  certificate-authority: /home/student/.minikube/ca.crt
  server: https://192.168.99.100:8443
  name: minikube
  contexts:
- context:
  cluster: minikube
  user: minikube
  name: minikube
- context:
  cluster: minikube
  namespace: lfs158
  user: student
  name: student-context
  current-context: minikube
  kind: Config
  preferences: {}
  users:
- name: minikube
  user:
  client-certificate: /home/student/.minikube/profiles/minikube/client.crt
  client-key: /home/student/.minikube/profiles/minikube/client.key
- name: student
  user:
  client-certificate: /home/student/rbac/student.crt
  client-key: /home/student/rbac/student.key

While in the default minikube context, create a new deployment in the lfs158 namespace:

~/rbac$ kubectl -n lfs158 create deployment nginx --image=nginx:alpine

deployment.apps/nginx created

From the new context student-context try to list pods. The attempt fails because the student user has no permissions configured for the student-context:

~/rbac$ kubectl --context=student-context get pods

Error from server (Forbidden): pods is forbidden: User "student" cannot list resource "pods" in API group "" in the namespace "lfs158"

The following steps will assign a limited set of permissions to the student user in the student-context.

Create a YAML configuration manifest for a pod-reader Role object, which allows only get, watch, list actions in the lfs158 namespace against pod objects. Then create the role object and list it from the default minikube context, but from the lfs158 namespace:

~/rbac$ vim role.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
name: pod-reader
namespace: lfs158
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

~/rbac$ kubectl create -f role.yaml

role.rbac.authorization.k8s.io/pod-reader created

~/rbac$ kubectl -n lfs158 get roles

NAME         CREATED AT
pod-reader   2020-10-07T03:47:45Z

Create a YAML configuration manifest for a rolebinding object, which assigns the permissions of the pod-reader Role to the student user. Then create the rolebinding object and list it from the default minikube context, but from the lfs158 namespace:

~/rbac$ vim rolebinding.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
name: pod-read-access
namespace: lfs158
subjects:
- kind: User
  name: student
  apiGroup: rbac.authorization.k8s.io
  roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io

~/rbac$ kubectl create -f rolebinding.yaml

rolebinding.rbac.authorization.k8s.io/pod-read-access created

~/rbac$ kubectl -n lfs158 get rolebindings

NAME              ROLE              AGE
pod-read-access   Role/pod-reader   28s

Now that we have assigned permissions to the student user, we can successfully list pods from the new context student-context.

~/rbac$ kubectl --context=student-context get pods

NAME                     READY   STATUS    RESTARTS   AGE
nginx-565785f75c-kl25r   1/1     Running   0          7m41s

Annotate
