Kops is a CLI tool and must be installed on your local machine alongside kubectl. Getting a cluster running is as simple as running the kops create cluster command with all of the necessary options. Kops will manage most of the AWS resources required to run a Kubernetes cluster, and will work with either a new or existing VPC. Unlike EKS, kops will create your master nodes as EC2 instances as well, and you are able to access those nodes directly and make modifications. With access to the master nodes, you can choose which networking layer to use, choose the size of master instances, and directly monitor the master nodes. You also have the option of setting up a cluster with only a single master, which might be desirable for dev and test environments where high availability is not a requirement. Kops also supports generating terraform config for your resources instead of directly creating them, which is a nice feature if you use terraform.

Below is an example command for creating a cluster with 3 masters and 3 workers in a new VPC. Instructions for getting an appropriate AMI for your nodes can be found here.

kops create cluster \
--cloud aws \
--dns public \
--dns-zone ${ROUTE53_ZONE} \
--topology private \
--networking weave \
--associate-public-ip=false \
--encrypt-etcd-storage \
--network-cidr 10.2.0.0/16 \
--image ${AMI_ID} \
--kubernetes-version 1.10.11 \
--master-size t2.medium \
--master-count 3 \
--master-zones us-east-1a,us-east-1b,us-east-1d \
--master-volume-size 64 \
--zones us-east-1a,us-east-1b,us-east-1d \
--node-size t2.large \
--node-count 3 \
--node-volume-size 128 \
--ssh-access 10.0.0.0/16 \
${CLUSTER_NAME}