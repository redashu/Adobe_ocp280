## clean project data 

<img src="del1.png">

### webapp 2 tier to deploy 

<img src="webapp.png">

## deployinng wbeapp

```
===> creating db secret to root password 

oc  create  secret generic  ashu-db-creds  --from-literal mypass="Adobe@1234"  --dry-run=client -o yaml >dbsecret.yaml 
[user12@ip-172-31-28-96 2twebapp]$ oc create -f dbsecret.yaml 
secret/ashu-db-creds created
[user12@ip-172-31-28-96 2twebapp]$ oc get secret
NAME                       TYPE                             DATA   AGE
ashu-azure-secret          kubernetes.io/dockerconfigjson   1      23h
ashu-db-creds              Opaque                           1      6s


===> creating  deployment 

oc create  deployment  ashudb --image adobe.azurecr.io/adobe:mysql --port  3306  --dry-run=client -o yaml >db-deploy.yaml


===> update the yaml with secret & env 

[user12@ip-172-31-28-96 2twebapp]$ oc create -f db-deploy.yaml 
deployment.apps/ashudb created
[user12@ip-172-31-28-96 2twebapp]$ oc get deploy
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
ashudb   1/1     1            1           17s
[user12@ip-172-31-28-96 2twebapp]$ oc get rs
NAME                DESIRED   CURRENT   READY   AGE
ashudb-5c595986bb   1         1         1       23s
[user12@ip-172-31-28-96 2twebapp]$ oc get pod
NAME                      READY   STATUS    RESTARTS   AGE
ashudb-5c595986bb-wjshq   1/1     Running   0          26s
[user12@ip-172-31-28-96 2twebapp]$ 

===> connecting to Db 

[user12@ip-172-31-28-96 ~]$ oc get po 
NAME                      READY   STATUS    RESTARTS   AGE
ashudb-5c595986bb-wjshq   1/1     Running   0          6m10s
[user12@ip-172-31-28-96 ~]$ oc rsh  ashudb-5c595986bb-wjshq 
sh-5.1# 
sh-5.1# 
sh-5.1# 
sh-5.1# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 9.6.0 MySQL Community Server - GPL

Copyright (c) 2000, 2026, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 



```


