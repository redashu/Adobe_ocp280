# OpenShift Bare-Metal (UPI) Install Guide

## 1. Prepare on Student Machine

Install Chromium:

```bash
sudo yum install chromium -y
```

Download pull secret:

- [Red Hat OpenShift Hybrid Cloud Platform](https://cloud.redhat.com/openshift/install/metal/user-provisioned)

Convert pull secret to formatted JSON:

```bash
python3 -m json.tool Downloads/pull-secret.txt > pull-secret.json
```

Update registry entry (replace `quay.io` usage with local registry details):

```json
"nexus-registry-int.apps.tools-apac150.prod.ole.redhat.com": {
  "auth": "",
  "email": ""
}
```

Convert to one-line installer format:

```bash
cat pull-secret.json | jq . -c > pull-secret-oneline.json
```

Transfer to utility server:

```bash
scp pull-secret-oneline.json lab@utility:
```

Reference image:

![Pull Secret Example](secret1.png)

## 2. Connect to Utility Server

SSH into utility host:

```bash
ssh lab@utility
```

Install Podman:

```bash
sudo yum install podman -y
```

Verify pull secret by pulling images:

```bash
sudo podman pull --authfile ~/pull-secret-oneline.json nexus-registry-int.apps.tools-apac150.prod.ole.redhat.com/openshift/ocp4:4.6.4-x86_64
```

```bash
sudo podman pull --authfile ~/pull-secret-oneline.json registry.redhat.io/ubi8/ubi:latest
```

List downloaded images:

```bash
sudo podman images
```

Example output:

```text
REPOSITORY                                                     TAG            IMAGE ID       CREATED        SIZE
registry.redhat.io/ubi8/ubi                                   latest         3269c37eae33   8 weeks ago    208 MB
nexus-registry-int.apps.tools-na150.prod.ole.redhat.com/openshift/ocp4   4.6.4-x86_64   26f7cd4cf1fb   2 months ago   322 MB
```

## 3. Verify Core Services

Check DNS, DHCP, and HTTPD:

```bash
systemctl status dhcpd named httpd
```

Create HTTPD directories:

```bash
sudo mkdir -p /var/www/html/openshift4/images
sudo mkdir -p /var/www/html/openshift4/4.6.4/ignitions
```

Restore SELinux contexts:

```bash
sudo restorecon -Rv /var/www/html/openshift4
```

Verify directory structure:

```bash
sudo tree /var/www/html/
```

Expected structure:

```text
/var/www/html/
└── openshift4
    ├── 4.6.4
    │   └── ignitions
    └── images
```

## 4. Download RHCOS Boot Images

```bash
ocp_maj=4.6
rhcos_ver=4.6.1
mirror=https://mirror.openshift.com/pub/openshift-v4
baseurl=${mirror}/dependencies/rhcos/${ocp_maj}/${rhcos_ver}

sudo wget ${baseurl}/rhcos-${rhcos_ver}-x86_64-live-rootfs.x86_64.img -P /var/www/html/openshift4/images
sudo wget ${baseurl}/rhcos-${rhcos_ver}-x86_64-live-kernel-x86_64 -P /var/www/html/openshift4/images
sudo wget ${baseurl}/rhcos-${rhcos_ver}-x86_64-live-initramfs.x86_64.img -P /var/www/html/openshift4/images
```

Verify downloads:

```bash
ls /var/www/html/openshift4/images
```

## 5. Configure HAProxy for Install Phase

Check HAProxy status:

```bash
systemctl status haproxy
```

Edit config:

```bash
sudo vi /etc/haproxy/haproxy.cfg
```

Add/update entries:

```cfg
#---------------------------------------------------------------------
# round robin balancing for RHOCP Kubernetes API Server
#---------------------------------------------------------------------
frontend k8s_api
  bind *:6443
  mode tcp
  default_backend k8s_api_backend
backend k8s_api_backend
  balance roundrobin
  mode tcp
  server bootstrap 192.168.50.9:6443 check
  server master01 192.168.50.10:6443 check
  server master02 192.168.50.11:6443 check
  server master03 192.168.50.12:6443 check

# ---------------------------------------------------------------------
# round robin balancing for RHOCP Machine Config Server
# ---------------------------------------------------------------------
frontend machine_config
  bind *:22623
  mode tcp
  default_backend machine_config_backend
backend machine_config_backend
  balance roundrobin
  mode tcp
  server bootstrap 192.168.50.9:22623 check
  server master01 192.168.50.10:22623 check
  server master02 192.168.50.11:22623 check
  server master03 192.168.50.12:22623 check

# ---------------------------------------------------------------------
# round robin balancing for RHOCP Ingress Insecure Port
# ---------------------------------------------------------------------
frontend ingress_insecure
  bind *:80
  mode tcp
  default_backend ingress_insecure_backend
backend ingress_insecure_backend
  balance roundrobin
  mode tcp
  server worker01 192.168.50.13:80 check
  server worker02 192.168.50.14:80 check

# ---------------------------------------------------------------------
# round robin balancing for RHOCP Ingress Secure Port
# ---------------------------------------------------------------------
frontend ingress_secure
  bind *:443
  mode tcp
  default_backend ingress_secure_backend
backend ingress_secure_backend
  balance roundrobin
  mode tcp
  server worker01 192.168.50.13:443 check
  server worker02 192.168.50.14:443 check
```

Validate and reload:

```bash
haproxy -c -f /etc/haproxy/haproxy.cfg
sudo systemctl reload haproxy
sudo systemctl status haproxy
```

## 6. Configure TFTP Boot Files

Download PXE configs:

```bash
sol_url=http://classroom.example.com/materials/solutions

sudo wget $sol_url/pxelinux.cfg/01-52-54-00-00-32-09 -P /var/lib/tftpboot/pxelinux.cfg/
sudo wget $sol_url/pxelinux.cfg/01-52-54-00-00-32-0a -P /var/lib/tftpboot/pxelinux.cfg/
sudo wget $sol_url/pxelinux.cfg/01-52-54-00-00-32-0b -P /var/lib/tftpboot/pxelinux.cfg/
sudo wget $sol_url/pxelinux.cfg/01-52-54-00-00-32-0c -P /var/lib/tftpboot/pxelinux.cfg/
sudo wget $sol_url/pxelinux.cfg/01-52-54-00-00-32-0d -P /var/lib/tftpboot/pxelinux.cfg/
sudo wget $sol_url/pxelinux.cfg/01-52-54-00-00-32-0e -P /var/lib/tftpboot/pxelinux.cfg/
```

Verify:

```bash
tree /var/lib/tftpboot/pxelinux.cfg/
```

Expected output:

```text
/var/lib/tftpboot/pxelinux.cfg/
├── 01-52-54-00-00-32-09
├── 01-52-54-00-00-32-0a
├── 01-52-54-00-00-32-0b
├── 01-52-54-00-00-32-0c
├── 01-52-54-00-00-32-0d
├── 01-52-54-00-00-32-0e
└── 01-example
```

## 7. Install OpenShift CLI and Installer

```bash
mirror="https://mirror.openshift.com/pub/openshift-v4/clients"

wget ${mirror}/ocp/4.6.4/openshift-client-linux-4.6.4.tar.gz
wget ${mirror}/ocp/4.6.4/openshift-install-linux-4.6.4.tar.gz

sudo tar -xvf openshift-client-linux-4.6.4.tar.gz -C /usr/bin/
sudo tar -xvf openshift-install-linux-4.6.4.tar.gz -C /usr/bin/
```

Verify versions:

```bash
oc version
openshift-install version
```

## 8. Create Cluster Config and Ignition

Generate SSH key:

```bash
ssh-keygen -t rsa -b 4096 -N '' -f .ssh/ocp4upi
```

Create working directory and download template:

```bash
mkdir ocp4upi && cd ocp4upi
wget http://classroom.example.com/materials/solutions/ocp4upi/install-config.yaml
```

Update `install-config.yaml`:

- Paste SSH public key
- Paste `pull-secret-oneline.json` content
- Update registry to `nexus-registry-int.apps.tools-apac150.prod.ole.redhat.com`

Backup config:

```bash
cp install-config.yaml /home/lab/
```

Create manifests and ignition configs:

```bash
openshift-install create manifests --dir=.
openshift-install create ignition-configs --dir=.
```

Reference image:

![Install Output Reference](lab1.png)

Copy ignition files to HTTPD path:

```bash
sudo cp *.ign /var/www/html/openshift4/4.6.4/ignitions/
sudo chmod +r /var/www/html/openshift4/4.6.4/ignitions/*.ign
```

## 9. Boot Nodes and Wait for Bootstrap

Boot nodes from console:

- bootstrap
- master01-03
- worker01-02

Verify connectivity from utility server:

```bash
ping bootstrap.ocp4.example.com
ping master01.ocp4.example.com
ping master02.ocp4.example.com
ping master03.ocp4.example.com
ping worker01.ocp4.example.com
ping worker02.ocp4.example.com
```

Wait for bootstrap completion (typically 20 to 30 minutes, sometimes faster):

```bash
cd ~
openshift-install --dir=./ocp4upi wait-for bootstrap-complete
```

Reference image:

![Bootstrap Output](boot.png)

## 10. Remove Bootstrap from HAProxy

Edit HAProxy config and remove bootstrap server lines:

```cfg
# In k8s_api_backend, remove:
server bootstrap 192.168.50.9:6443 check

# In machine_config_backend, remove:
server bootstrap 192.168.50.9:22623 check
```

Reload HAProxy:

```bash
sudo systemctl reload haproxy
```

## 11. Validate Cluster and Complete Install

Set kubeconfig and test access:

```bash
export KUBECONFIG=~/ocp4upi/auth/kubeconfig
oc whoami
oc get nodes
```

Example node output:

```text
master01   Ready   master   76m   v1.19.0+9f84db3
master02   Ready   master   70m   v1.19.0+9f84db3
master03   Ready   master   69m   v1.19.0+9f84db3
```

Run final install completion:

```bash
openshift-install --dir=ocp4upi wait-for install-complete --log-level=debug
```

If CSRs are pending, approve manually:

```bash
oc get csr
oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs oc adm certificate approve
```


[lab@utility ~]$ oc get nodes

master01   Ready    master   76m   v1.19.0+9f84db3

master02   Ready    master   70m   v1.19.0+9f84db3

master03   Ready    master   69m   v1.19.0+9f84db3

