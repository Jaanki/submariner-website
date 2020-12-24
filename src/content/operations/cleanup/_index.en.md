---
title: "Uninstalling Submariner"
date: 2020-12-23T21:25:11+01:00
weight: 50
---

To properly uninstall Submariner from a cluster, follow the steps below:

1. Delete Submariner related namespaces

   On each participating cluster, issue the following command:

   ```bash
   kubectl delete namespace submariner-operator
   ```

   On the Broker cluster, issue the following command:

   ```bash
   kubectl delete namespace submariner-k8s-broker
    ```

2. Delete the Submariner CRDs

   On each participating cluster, issue the following command:

   ```bash
   for CRD in `kubectl get crds | grep -iE 'submariner|multicluster.x-k8s.io'| awk '{print $1}'`; do kubectl delete crd $CRD; done
   ```

3. Delete ClusterRoles and ClusterRoleBindings

   On each participating cluster, issue the following command:

   ```bash
   roles="submariner-operator submariner-operator-globalnet submariner-lighthouse submariner-networkplugin-syncer"
   kubectl delete clusterrole,clusterrolebinding $roles --ignore-not-found
   ```

4. Remove Submariner's iptables chains

   On all nodes in each participating cluster, issue the following commands:

   ```bash
   iptables --flush SUBMARINER-INPUT
   iptables -D INPUT $(iptables -L INPUT --line-numbers | grep SUBMARINER-INPUT | awk '{print $1}')
   iptables --delete-chain SUBMARINER-INPUT

   iptables --flush SUBMARINER-POSTROUTING
   iptables -D INPUT $(iptables -L INPUT --line-numbers | grep SUBMARINER-POSTROUTING | awk '{print $1}')
   iptables --delete-chain SUBMARINER-POSTROUTING
   ```

   If Globalnet is enabled in the setup, issue the following commands as well

   ```bash
   iptables --flush SUBMARINER-GN-INGRESS
   iptables -D INPUT $(iptables -L INPUT --line-numbers | grep SUBMARINER-GN-INGRESS | awk '{print $1}')
   iptables --delete-chain SUBMARINER-GN-INGRESS

   iptables --flush SUBMARINER-GN-EGRESS
   iptables -D INPUT $(iptables -L INPUT --line-numbers | grep SUBMARINER-GN-EGRESS | awk '{print $1}')
   iptables --delete-chain SUBMARINER-GN-EGRESS

   iptables --flush SUBMARINER-GN-MARK
   iptables -D INPUT $(iptables -L INPUT --line-numbers | grep SUBMARINER-GN-MARK | awk '{print $1}')
   iptables --delete-chain SUBMARINER-GN-MARK
   ```

   {{% notice note %}}
   Note that Kind based setup will have just `SUBMARINER-INPUT` chain.
   {{% /notice %}}

5. Delete the `vx-submariner` interface

   On all nodes in each participating cluster, issue the following command:

   ```bash
   ip link delete vx-submariner
   ```

6. Remove the Submariner gateway labels

   On all nodes in each participating cluster, issue the following command:

   ```bash
   kubectl label --all node submariner.io/gateway-
   ```

7. For OpenShift deployments, delete `Lighthouse ServiceExport DNS`

   For each participating cluster, issue the following command:

   ```bash
   oc apply -f -  <<EOF
   apiVersion: operator.openshift.io/v1
   kind: DNS
   metadata:
     finalizers:
     - dns.operator.openshift.io/dns-controller
     name: default
   spec:
     servers: []
   EOF
   ```
