---
title: Create an internal load balancer
titleSuffix: Azure Kubernetes Service
description: Learn how to create and use an internal load balancer to expose your services with Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 03/04/2019


#Customer intent: As a cluster operator or developer, I want to learn how to create a service in AKS that uses an internal Azure load balancer for enhanced security and without an external endpoint.
---

# Use an internal load balancer with Azure Kubernetes Service (AKS)

To restrict access to your applications in Azure Kubernetes Service (AKS), you can create and use an internal load balancer. An internal load balancer makes a Kubernetes service accessible only to applications running in the same virtual network as the Kubernetes cluster. This article shows you how to create and use an internal load balancer with Azure Kubernetes Service (AKS).

> [!NOTE]
> Azure Load Balancer is available in two SKUs - *Basic* and *Standard*. By default, the Standard SKU is used when you create an AKS cluster.  When creating a Service with type as LoadBalancer, you will get the same LB type as when you provision the cluster. For more information, see [Azure load balancer SKU comparison][azure-lb-comparison].

## Before you begin

This article assumes that you have an existing AKS cluster. If you need an AKS cluster, see the AKS quickstart [using the Azure CLI][aks-quickstart-cli] or [using the Azure portal][aks-quickstart-portal].

You also need the Azure CLI version 2.0.59 or later installed and configured. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI][install-azure-cli].

The AKS cluster cluster identity needs permission to manage network resources if you use an existing subnet or resource group. For information see [Use kubenet networking with your own IP address ranges in Azure Kubernetes Service (AKS)][use-kubenet] or [Configure Azure CNI networking in Azure Kubernetes Service (AKS)][advanced-networking]. If you are configuring your load balancer to use an [IP address in a different subnet][different-subnet], ensure the the AKS cluster identity also has read access to that subnet.

For more information on permissions, see [Delegate AKS access to other Azure resources][aks-sp].

## Create an internal load balancer

To create an internal load balancer, create a service manifest named `internal-lb.yaml` with the service type *LoadBalancer* and the *azure-load-balancer-internal* annotation as shown in the following example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: internal-app
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: internal-app
```

Deploy the internal load balancer using the [kubectl apply][kubectl-apply] and specify the name of your YAML manifest:

```console
kubectl apply -f internal-lb.yaml
```

An Azure load balancer is created in the node resource group and connected to the same virtual network as the AKS cluster.

When you view the service details, the IP address of the internal load balancer is shown in the *EXTERNAL-IP* column. In this context, *External* is in relation to the external interface of the load balancer, not that it receives a public, external IP address. It may take a minute or two for the IP address to change from *\<pending\>* to an actual internal IP address, as shown in the following example:

```
$ kubectl get service internal-app

NAME           TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
internal-app   LoadBalancer   10.0.248.59   10.240.0.7    80:30555/TCP   2m
```

## Specify an IP address

If you would like to use a specific IP address with the internal load balancer, add the *loadBalancerIP* property to the load balancer YAML manifest. In this scenario, the specified IP address must reside in the same subnet as the AKS cluster but can't already be assigned to a resource. For example, an IP address in the range designated for the Kubernetes subnet within the AKS cluster shouldn't be used.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: internal-app
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"
spec:
  type: LoadBalancer
  loadBalancerIP: 10.240.0.25
  ports:
  - port: 80
  selector:
    app: internal-app
```

When deployed and you view the service details, the IP address in the *EXTERNAL-IP* column reflects your specified IP address:

```
$ kubectl get service internal-app

NAME           TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
internal-app   LoadBalancer   10.0.184.168   10.240.0.25   80:30225/TCP   4m
```

For more information on configuring your load balancer in a different subnet, see [Specify a different subnet][different-subnet]

## Use private networks

When you create your AKS cluster, you can specify advanced networking settings. This approach lets you deploy the cluster into an existing Azure virtual network and subnets. One scenario is to deploy your AKS cluster into a private network connected to your on-premises environment and run services only accessible internally. For more information, see configure your own virtual network subnets with [Kubenet][use-kubenet] or [Azure CNI][advanced-networking].

No changes to the previous steps are needed to deploy an internal load balancer in an AKS cluster that uses a private network. The load balancer is created in the same resource group as your AKS cluster but connected to your private virtual network and subnet, as shown in the following example:

```
$ kubectl get service internal-app

NAME           TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
internal-app   LoadBalancer   10.1.15.188   10.0.0.35     80:31669/TCP   1m
```

> [!NOTE]
> You may need to grant the cluster identity for your AKS cluster the *Network Contributor* role to the resource group where your Azure virtual network resources are deployed. View the cluster identity with [az aks show][az-aks-show], such as `az aks show --resource-group myResourceGroup --name myAKSCluster --query "identity"`. To create a role assignment, use the [az role assignment create][az-role-assignment-create] command.

## Specify a different subnet

To specify a subnet for your load balancer, add the *azure-load-balancer-internal-subnet* annotation to your service. The subnet specified must be in the same virtual network as your AKS cluster. When deployed, the load balancer *EXTERNAL-IP* address is part of the specified subnet.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: internal-app
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"
    service.beta.kubernetes.io/azure-load-balancer-internal-subnet: "apps-subnet"
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: internal-app
```

## Delete the load balancer

When all services that use the internal load balancer are deleted, the load balancer itself is also deleted.

You can also directly delete a service as with any Kubernetes resource, such as `kubectl delete service internal-app`, which also then deletes the underlying Azure load balancer.

## Next steps

Learn more about Kubernetes services at the [Kubernetes services documentation][kubernetes-services].

<!-- LINKS - External -->
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubernetes-services]: https://kubernetes.io/docs/concepts/services-networking/service/
[aks-engine]: https://github.com/Azure/aks-engine

<!-- LINKS - Internal -->
[advanced-networking]: configure-azure-cni.md
[az-aks-show]: /cli/azure/aks#az_aks_show
[az-role-assignment-create]: /cli/azure/role/assignment#az_role_assignment_create
[azure-lb-comparison]: ../load-balancer/skus.md
[use-kubenet]: configure-kubenet.md
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[aks-sp]: kubernetes-service-principal.md#delegate-access-to-other-azure-resources
[different-subnet]: #specify-a-different-subnet