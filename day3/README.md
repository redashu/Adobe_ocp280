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

### attach pvc to your Db 

```
oc replace -f db-deploy.yaml --force 
deployment.apps "ashudb" deleted
deployment.apps/ashudb replaced
[user12@ip-172-31-28-96 2twebapp]$ oc get deploy
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
ashu-web   2/2     2            2           21h
ashudb     0/1     1            0           6s
[user12@ip-172-31-28-96 2twebapp]$ oc get po 
NAME                        READY   STATUS              RESTARTS   AGE
ashu-web-5b444dd958-6sqdf   1/1     Running             1          21h
ashu-web-5b444dd958-rg57h   1/1     Running             1          21h
ashudb-6d657fcb98-kp96t     0/1     ContainerCreating   0          9s
[user12@ip-172-31-28-96 2twebapp]$ oc get pvc
NAME       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
ashu-pvc   Bound    pvc-bc755d1c-6d48-4108-862b-9664d9e72bea   10Gi       RWO            gp3-csi        <unset>                 11m
[user12@ip-172-31-28-96 2twebapp]$ oc get po 
NAME                        READY   STATUS    RESTARTS   AGE
ashu-web-5b444dd958-6sqdf   1/1     Running   1          21h
ashu-web-5b444dd958-rg57h   1/1     Running   1          21h
ashudb-6d657fcb98-kp96t     1/1     Running   0          20s

```
