## Kubespray
Pre-requis :
 - Ansible Node (Kubespray Node): Gitpod Node
 - 1 Controller Nodes: rhel8
 - 2 Worker Nodes: rhel8

 ## automate installation of ansible/kubespray and configure the Ansible node

 create this .gitpod.yml file if not exists and replace the content

```bash
 tasks:
  - name: Install kubespray
    init: |
      sudo apt update
      sudo apt install git python3 python3-pip -y
      git clone https://github.com/kubernetes-incubator/kubespray.git
      cd kubespray
      pip install -r requirements.txt
      ansible --version
```

Create the hosts inventory:
  - use the hosts.yaml on this repertorie and replace IP address that suits to your deployment
  - run below commands and don’t forget to replace IP address that suits to your deployment
      ```bash
      cp -rfp inventory/sample inventory/mycluster
      declare -a IPS=(172.31.27.251 172.31.16.38 172.31.17.46)
      CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
      ```
add these "k8s-cluster.yml" and "addons.yml" in the inventory/mycluster/group_vars/k8s_cluster directory

be sure that these params are the same for each file:

“inventory/mycluster/group_vars/k8s_cluster/k8s-cluster.yml” file

```bash
kube_version: v1.26.2
kube_network_plugin: calico
kube_pods_subnet: 10.233.64.0/18
kube_service_addresses: 10.233.0.0/18
cluster_name: linuxtechi.local
```

“inventory/mycluster/group_vars/k8s_cluster/addons.yml”

```bash
dashboard_enabled: true
ingress_nginx_enabled: true
ingress_nginx_host_network: true
```
## Prepare Cluster Installation

### Be Sure all nodes are SSH accessible from ansible node

```bash
# First generate the ssh-keys for your local user on your ansible node
ssh-keygen
# Copy the ssh-keys using ssh-copy-id command on each node
ssh-copy-id cloud_user@public_IP

#Setup passwordless ssh login on each node; connect via ssh and run the following command on each node
echo "cloud_user ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/cloud_user

```

### Disable Firewall and Enable IPV4 forwarding on all nodes

run these commands on ansible node

```bash
cd kubespray
# Disable firewall
ansible all -i inventory/mycluster/hosts.yaml -m shell -a "sudo systemctl stop firewalld && sudo systemctl disable firewalld" -u cloud_user
# enable IPv4
ansible all -i inventory/mycluster/hosts.yaml -m shell -a "echo 'net.ipv4.ip_forward=1' | sudo tee -a /etc/sysctl.conf" -u cloud_user
# Disable swap
ansible all -i inventory/mycluster/hosts.yaml -m shell -a "sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab && sudo swapoff -a" -u cloud_user

```

## Start Kubernetes deplyment

ansible-playbook -i inventory/mycluster/hosts.yaml --become --become-user=root cluster.yml -u cloud_user

## Links

https://github.com/kubernetes-sigs/kubespray