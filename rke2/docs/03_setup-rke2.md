## Setup RKE2 Servers

#### Add first server node

Connect to the master1 node

```bash
vagrant ssh master1
```

Change to root user

```bash
sudo -i
```

First, you must create the directory where the RKE2 config file is going to be placed

```bash
mkdir -p /etc/rancher/rke2/
```

Next, create the RKE2 config file at /etc/rancher/rke2/config.yaml.

> [!NOTE] 
> If you do not specify a pre-shared secret, RKE2 will generate one and place it at /var/lib/rancher/rke2/server/node-token.

> [!NOTE] 
> Don't forget to change the following IPs with yours.

```bash
cat << EOF > /etc/rancher/rke2/config.yaml
tls-san:
  - 192.168.68.64
  - loadbalancer
node-ip: "192.168.68.56"
cni: "calico"
write-kubeconfig-mode: "0644"
EOF
```

Additional options are below:

```
**cluster-domain**
Default: "cluster.local"
Purpose: Defines the internal DNS domain for the Kubernetes cluster.
**cluster-cidr**
Default: "10.42.0.0/16" (in RKE2)
Purpose: Defines the IP range for pod networking.
**service-cidr**
Default: "10.43.0.0/16" (in RKE2)
Purpose: Defines the IP range for services.
**tls-san**
Default: No SANs are specified by default, and the API server uses the node’s default IP/hostname.
Purpose: Specifies additional IPs or hostnames for the server’s TLS certificate.
**node-ip**
Default: Kubernetes automatically detects and uses the first non-loopback IP of the node.
Purpose: Specifies the IP address of the node.
**token**
Default: If no token is specified, RKE2 automatically generates one and stores it at /var/lib/rancher/rke2/server/node-token.
Purpose: A shared secret for securely joining nodes to the cluster.
**cni**
Default: "canal" (in RKE2, which is a combination of Calico and Flannel)
Purpose: Specifies the CNI plugin to use for networking.
**write-kubeconfig-mode**
Default: "0600"
Purpose: Sets the permissions for the kubeconfig file, restricting it to the owner (usually root).
**etcd-snapshot-schedule-cron**
Default: Not set by default. Automatic etcd snapshots are not scheduled unless you explicitly set this value.
Purpose: Schedules periodic etcd snapshots.
**resolv-conf**
Default: Kubernetes uses the default /etc/resolv.conf on the host unless a custom one is provided.
Purpose: Specifies the DNS resolver configuration for kubelet.

Customizing these options allows you to better control and optimize your cluster’s behavior, especially in more advanced or production setups.
```


After that, you need to run the install command and enable and start rke2

```bash
curl -sfL https://get.rke2.io | sh -
systemctl enable rke2-server.service
systemctl start rke2-server.service
```

You will find the token generated by rke2 here, use it in next server nodes.

```bash
cat /var/lib/rancher/rke2/server/node-token
```

#### Add next server node

Connect to the master2 node

```bash
vagrant ssh master2
```

Change to root user

```bash
sudo -i
```

Create same folder in next server node

```bash
mkdir -p /etc/rancher/rke2/
```

> [!NOTE] 
> Don't share your token publicly if you're in production environment.

Create config file

```bash
cat << EOF > /etc/rancher/rke2/config.yaml
token: "K104303aaf38355ca166a5f7f29e57b4f2bbb11bf38c847f9823e7a8fad65bab126::server:bf9363e5c9552e25e02cda7d0c6f744c"
server: https://loadbalancer:9345
tls-san:
  - 192.168.68.64
  - loadbalancer
node-ip: "192.168.68.57"
cni: "calico"
write-kubeconfig-mode: "0644"
EOF
```

Install and start rke2

```bash
curl -sfL https://get.rke2.io | sh -
systemctl enable rke2-server.service
systemctl start rke2-server.service
```

Connect to the next node and perform the same steps. Just remember to change the node-ip in config file.

- Use root user
- Create rke2 folder
- Create config file

```bash
cat << EOF > /etc/rancher/rke2/config.yaml
token: "K104303aaf38355ca166a5f7f29e57b4f2bbb11bf38c847f9823e7a8fad65bab126::server:bf9363e5c9552e25e02cda7d0c6f744c"
server: https://loadbalancer:9345
tls-san:
  - 192.168.68.64
  - loadbalancer
node-ip: "192.168.68.58"
cni: "calico"
write-kubeconfig-mode: "0644"
EOF
```

#### Control Server

```bash
/var/lib/rancher/rke2/bin/kubectl \
        --kubeconfig /etc/rancher/rke2/rke2.yaml get nodes
```

#### Add worker/agent node

Perform the same steps for agents as well.

- Use root user
- Create rke2 folder
- Create config file

```bash
cat << EOF > /etc/rancher/rke2/config.yaml
token: "K104303aaf38355ca166a5f7f29e57b4f2bbb11bf38c847f9823e7a8fad65bab126::server:bf9363e5c9552e25e02cda7d0c6f744c"
server: https://loadbalancer:9345
node-ip: "192.168.68.61"
cni: "calico"
write-kubeconfig-mode: "0644"
EOF
```

Run the installer and start service

```bash
curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE="agent" sh -
```

```bash
systemctl enable rke2-agent.service
```
```bash
systemctl start rke2-agent.service
```

#### Troubleshooting

See logs to detect problem

```bash
journalctl -u rke2-server.service -f
```

```bash
journalctl -xeu rke2-server.service
```

You can start again after running the scripts below. (Kill process and uninstall)
```bash
sudo /usr/local/bin/rke2-killall.sh
sudo /usr/local/bin/rke2-uninstall.sh
```