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