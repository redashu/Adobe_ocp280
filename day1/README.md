# Day 1 - OpenShift CLI Basics

## Check OCP Client Version

```bash
[user12@ip-172-31-28-96 ~]$ oc version
Client Version: 4.16.0
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
[user12@ip-172-31-28-96 ~]$
```

## Log In to OCP Cluster Using oc Client

### 1. Verify Access Before Login

```bash
oc get nodes
error: Missing or incomplete configuration info.  Please point to an existing, complete config file:

  1. Via the command-line flag --kubeconfig
  2. Via the KUBECONFIG environment variable
  3. In your home directory as ~/.kube/config

To view or setup config directly use the 'config' command.
[user12@ip-172-31-28-96 ~]$
```

### 2. Login to Cluster

```bash
[user12@ip-172-31-28-96 ~]$ oc login https://api.mayank.openshiftlab.xyz:6443 -u user12
The server uses a certificate signed by an unknown authority.
You can bypass the certificate check, but any data you send to the server could be intercepted by others.
Use insecure connections? (y/n): y

WARNING: Using insecure TLS client config. Setting this option is not supported!

Console URL: https://api.mayank.openshiftlab.xyz:6443/console
Authentication required for https://api.mayank.openshiftlab.xyz:6443 (openshift)
Username: user12
Password:
Login successful.

You don't have any projects. You can try to create a new project, by running

    oc new-project <projectname>

Welcome! See 'oc help' to get started.
```

### checking more details 

```
oc whoami
user12
[user12@ip-172-31-28-96 ~]$ oc whoami --show-server
https://api.mayank.openshiftlab.xyz:6443
[user12@ip-172-31-28-96 ~]$ oc whoami --show-console
https://console-openshift-console.apps.mayank.openshiftlab.xyz
[user12@ip-172-31-28-96 ~]$ 

```

### Creating project and switching into 

```
oc new-project  ashu-project 
Now using project "ashu-project" on server "https://api.mayank.openshiftlab.xyz:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.43 -- /agnhost serve-hostname

[root@openshift ashu]# 
[root@openshift ashu]# 
[root@openshift ashu]# oc project
Using project "ashu-project" on server "https://api.mayank.openshiftlab.xyz:6443".
[root@openshift ashu]# 


===>
oc project
Using project "ashu-project" on server "https://api.mayank.openshiftlab.xyz:6443".
[root@openshift ashu]# oc projects
You have one project on this server: "ashu-project".


```

### pod history 

```
[root@openshift ashu]# oc whoami
user12
[root@openshift ashu]# oc run  ashupod1  --image  nginx  --port 80 
pod/ashupod1 created
[root@openshift ashu]# oc get po 
NAME       READY   STATUS             RESTARTS      AGE
ashupod1   0/1     CrashLoopBackOff   6 (71s ago)   7m5s
[root@openshift ashu]# oc get po 
NAME       READY   STATUS             RESTARTS        AGE
ashupod1   0/1     CrashLoopBackOff   6 (3m51s ago)   9m45s
[root@openshift ashu]# oc logs  ashupod1
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: can not modify /etc/nginx/conf.d/default.conf (read-only file system?)
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/04/06 05:37:57 [warn] 1#1: the "user" directive makes sense only if the master process runs with super-user privileges, ignored in /etc/nginx/nginx.conf:2
nginx: [warn] the "user" directive makes sense only if the master process runs with super-user privileges, ignored in /etc/nginx/nginx.conf:2
2026/04/06 05:37:57 [emerg] 1#1: mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)
nginx: [emerg] mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)
[root@openshift ashu]# 
[root@openshift ashu]# 
[root@openshift ashu]# 
[root@openshift ashu]# oc describe pod ashupod1
Name:             ashupod1
Namespace:        ashu-project
Priority:         0
Service Account:  default
Node:             ip-10-0-125-231.ec2.internal/10.0.125.231
Start Time:       Mon, 06 Apr 2026 05:32:03 +0000
Labels:           run=ashupod1
Annotations:      k8s.ovn.org/pod-networks:
                    {"default":{"ip_addresses":["10.131.0.23/23"],"mac_address":"0a:58:0a:83:00:17","gateway_ips":["10.131.0.1"],"routes":[{"dest":"10.128.0.0...
                  k8s.v1.cni.cncf.io/network-status:


```


## Creating image and pushing to ACR 

```
git clone https://github.com/schoolofdevops/html-sample-app.git

Cloning into 'html-sample-app'...
remote: Enumerating objects: 74, done.
remote: Counting objects: 100% (3/3), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 74 (delta 0), reused 0 (delta 0), pack-reused 71 (from 1)
Receiving objects: 100% (74/74), 1.38 MiB | 5.70 MiB/s, done.
Resolving deltas: 100% (5/5), done.


[user12@ip-172-31-28-96 ~]$ ls
html-sample-app
[user12@ip-172-31-28-96 ~]$ 

[user12@ip-172-31-28-96 ~]$ ls
html-sample-app
[user12@ip-172-31-28-96 ~]$ 
[user12@ip-172-31-28-96 ~]$ cd html-sample-app/
[user12@ip-172-31-28-96 html-sample-app]$ ls
LICENSE.txt  README.txt  assets  elements.html  generic.html  html5up-phantom.zip  images  index.html
[user12@ip-172-31-28-96 html-sample-app]$ touch Dockerfile .dockerignore
[user12@ip-172-31-28-96 html-sample-app]$ ls
Dockerfile  LICENSE.txt  README.txt  assets  elements.html  generic.html  html5up-phantom.zip  images  index.html
[user12@ip-172-31-28-96 html-sample-app]$ 
[user12@ip-172-31-28-96 html-sample-app]$ 
[user12@ip-172-31-28-96 html-sample-app]$ ls -a
.  ..  .dockerignore  .git  Dockerfile  LICENSE.txt  README.txt  assets  elements.html  generic.html  html5up-phantom.zip  images  index.html
[user12@ip-172-31-28-96 html-sample-app]$ 
[user12@ip-172-31-28-96 html-sample-app]$ 

==> update Dockerfile & .dockerignore 

===>Buidling container image

docker build  -t  ashu:v1  .  
[+] Building 3.1s (6/7)                                                                       docker:default
 => [internal] load build definition from Dockerfile             

 ====> docker images check 

  docker images
REPOSITORY   TAG       IMAGE ID       CREATED              SIZE
prakakum     v1        3bd5fa83f251   47 seconds ago       163MB
anshul       v1        75ceb389a302   54 seconds ago       163MB


===> Docker tag. 

docker  tag  ashu:v1  adobe.azurecr.io/adobe:ashuv1
[user12@ip-172-31-28-96 html-sample-app]$ docker images
REPOSITORY                 TAG          IMAGE ID       CREATED         SIZE
adobe.azurecr.io/adobe     prakakumv1   3bd5fa83f251   4 minutes ago   163MB
prakakum                   v1           3bd5fa83f251   4 minutes ago   163MB


===> login to Registry 

docker login  adobe.azurecr.io 
Username: adobe
Password: 
WARNING! Your password will be stored unencrypted in /home/user12/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

===> Docker push 

 docker  push   adobe.azurecr.io/adobe:ashuv1
The push refers to repository [adobe.azurecr.io/adobe]
60b3d7fc8c16: Pushed 
4e0a2a122e2f: Layer already exists 
794b45c9a1a2: Layer already exists 


```

### Deploy pushed app 

```
oc  create  deployment  ashu-dep1 --image adobe.azurecr.io/adobe:ashuv1 --port 80 --dry-run=client  -o yaml  >ashudeploy_day1.yaml 

```



