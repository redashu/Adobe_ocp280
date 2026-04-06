# Adobe_ocp280

### user details 

<img src="user.png">



### how to assign anyuid to any project 

```
oc adm policy add-scc-to-user anyuid -z default -n ashu-project 
```