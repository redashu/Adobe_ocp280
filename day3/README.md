## enable bash completion for oc 

```
[user12@ip-172-31-28-96 ~]$ oc completion bash > ~/.oc_bash_completion
[user12@ip-172-31-28-96 ~]$ 
[user12@ip-172-31-28-96 ~]$ 
[user12@ip-172-31-28-96 ~]$ source  ~/.oc_bash_completion 
[user12@ip-172-31-28-96 ~]$ 
[user12@ip-172-31-28-96 ~]$ oc get  
Display all 227 possibilities? (y or n)
[user12@ip-172-31-28-96 ~]$ oc get  n
namespaces                                      networks.operator.openshift.io
network-attachment-definitions.k8s.cni.cncf.io  nodes
networkpolicies.networking.k8s.io               nodes.config.openshift.io
networks.config.openshift.io                    nodes.metrics.k8s.io
[user12@ip-172-31-28-96 ~]$ oc get  n


```