---
layout: default
title: Kubernetes
---

# Kubernetes
{: .no_toc }

### Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

----

## Linking EKS with aws cli

`aws eks --region us-east-2 update-kubeconfig --name eksCLusterName`

> Replace region name `us-east-2` with the good one and `eksClusterName` with cluster name

## Unauthorized to run commands or how to link kubectl to an existing cluster

If you have `error: You must be logged in to the server (Unauthorized)` this type of error:

1. Get yours arn : `aws sts get-caller-identity`
2. Then update config with `aws eks update-kubeconfig --name eksCLusterNames --region eu-west-1 --role-arn arn:aws:iam::xxxxxxx:user/pierrick`

Replace `role-arn` with the role from step 1 and replace `eksCLusterName` wih your cluster name and the right region.

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

> Replace `admin-user` with right user

## Connect to dashboard

`kubectl proxy`

Then go to [http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/](http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/)

## Add secret to a private registry

First, generate a deploy tokens into the project / group : settings > repository > Deploy tokens. **Check only read registry**

`kubectl create secret docker-registry registry.gitlab.com --docker-server=https://registry.gitlab.com --docker-username=gitlab+deploy-token-xxxxxx --docker-password=mypassword --docker-email=mymail@mysam.co`

Then into the pod yml add:
```yaml
spec:
  containers:
    - name: example
      image: 'registry.gitlab.com/my-sam/devops/environment/nginx:latest'
      ports:
        - containerPort: 80
          protocol: TCP
      resources:
        limits:
          cpu: 500m
          memory: 512Mi
        requests:
          cpu: 250m
          memory: 50Mi
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      imagePullPolicy: IfNotPresent
  restartPolicy: Always
  terminationGracePeriodSeconds: 30
  dnsPolicy: ClusterFirst
  serviceAccountName: default
  serviceAccount: default
  automountServiceAccountToken: false
  nodeName: ip-10-0-2-218.us-east-2.compute.internal
  shareProcessNamespace: false
  securityContext: {}
  imagePullSecrets:
    - name: registry.gitlab.com
```
