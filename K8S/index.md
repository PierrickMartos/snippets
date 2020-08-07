# Kubernetes snippet

## Linking EKS with acs cli

`aws eks --region us-east-2 update-kubeconfig --name eksCLusterName`

>> Replace region name and `eksClusterName` with cluster name

## Setup dashboard

Download last metrics server release: [https://github.com/kubernetes-sigs/metrics-server/releases](https://github.com/kubernetes-sigs/metrics-server/releases)

Then extract and run:

`kubectl apply -f metrics-server-0.3.7/deploy/1.8+/`

To check install is a success:
`kubectl get deployment metrics-server -n kube-system`

Then install dashboard:
[https://github.com/kubernetes/dashboard/releases](https://github.com/kubernetes/dashboard/releases)

`kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.3/aio/deploy/recommended.yaml`

Then apply RBAC:
`kubectl apply -f https://raw.githubusercontent.com/hashicorp/learn-terraform-provision-eks-cluster/master/kubernetes-dashboard-admin.rbac.yaml`

## Get token for dashboard

`kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')`

>> Replace `admin-user` with right user

## Connect to dashboard

`kubectl proxy`

Then go to [http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/](http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/)
