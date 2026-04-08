# Lab: Re-encrypt Route to Nginx on OpenShift

This lab configures an Nginx app behind an OpenShift re-encrypt route using:

- Public certificate: browser to router
- Backend certificate: router to Nginx pod

Target hostname used in this example: `shop.hello.xyz`

## 1. Generate Public Certificate (Internet to Router)

Generate cert/key for the external hostname:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout public.key -out public.crt \
  -subj "/CN=shop.hello.xyz"
```

## 2. Generate Backend Certificate (Router to Pod)

Generate cert/key for backend TLS. The CN should match service DNS.

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout backend.key -out backend.crt \
  -subj "/CN=my-nginx.nginx-project.svc"
```

## 3. Prepare OpenShift Project and Secret

Create project and upload backend cert/key as a TLS secret:

```bash
oc new-project nginx-project
oc create secret tls nginx-server-certs --cert=backend.crt --key=backend.key
```

## 4. Create Nginx TLS Config (`default.conf`)

Create `default.conf`:

```nginx
server {
    listen 443 ssl;
    server_name shop.hello.xyz;

    ssl_certificate     /etc/nginx/certs/tls.crt;
    ssl_certificate_key /etc/nginx/certs/tls.key;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}
```

Upload the file as a ConfigMap:

```bash
oc create configmap nginx-server-conf --from-file=default.conf
```

## 5. Deploy Nginx and Mount Cert/Config Volumes

Deploy app:

```bash
oc new-app nginx:latest --name=my-nginx
```

Patch deployment to expose port `443` and mount secret/config:

```bash
oc patch deployment/my-nginx --patch '
spec:
  template:
    spec:
      containers:
      - name: my-nginx
        ports:
        - containerPort: 443
        volumeMounts:
        - name: certs-vol
          mountPath: /etc/nginx/certs
          readOnly: true
        - name: conf-vol
          mountPath: /etc/nginx/conf.d/default.conf
          subPath: default.conf
      volumes:
      - name: certs-vol
        secret:
          secretName: nginx-server-certs
      - name: conf-vol
        configMap:
          name: nginx-server-conf
'
```

## 6. Create HTTPS Service and Re-encrypt Route

Patch service to use HTTPS port:

```bash
oc patch svc/my-nginx -p '{"spec":{"ports":[{"name":"https","port":443,"targetPort":443}]}}'
```

Create re-encrypt route:

```bash
oc create route reencrypt nginx-ssl \
  --service=my-nginx \
  --hostname=shop.hello.xyz \
  --cert=public.crt \
  --key=public.key \
  --dest-ca-cert=backend.crt
```

## 7. Configure Route53 Record

Create a DNS record for `shop.hello.xyz` in Route53:

- Type: `CNAME` (or Alias where applicable)
- Target: Router canonical hostname (typically the OpenShift router/load balancer DNS)

## Encryption Flow Summary

1. Browser -> HTTPS using `public.crt` -> OpenShift Router
2. Router -> HTTPS using `backend.crt` trust -> Nginx Pod

## Troubleshooting Note

If Nginx fails with permission errors on port `443`, SCC constraints may be blocking startup.

Example workaround:

```bash
oc adm policy add-scc-to-user anyuid -z default
```