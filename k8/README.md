# OAI-Kubernetes
Open Air Interface EPC's MME, HSS, PWG &amp; SGW deployed at Kubernetes

OpenAirInterface is an implementation of the 3GPP specifications concerning the Evolved Packet Core Networks, that means it contains the implementation of the following network elements: MME, HSS, S-GW, P-GW.
[More information about Open Air Interface](https://gitlab.eurecom.fr/oai/openair-cn)

This project is aimed to run OpenAirInterface  in Docker Containers on a Kubernetes infrastructure

## Install Requirements
Please note that this procedure uses Juju 2.X and Ubuntu Xenial for deploying and performing configuration changes at K8 but any other K8 methodologies are supported


The enviroment installation:

- Install Kubernetes 1.7 
- Download kubectl configuration and binary

   1) Create the kubectl config directory at your home directory.
    `mkdir -p ~/.kube`

   2) Copy the kubeconfig file to the default location.
    `juju scp kubernetes-master/0:config ~/.kube/config` 
   
   3) Fetch a binary for the architecture you have deployed. 
   If your client is adifferent architecture you will need to get the appropriate kubectl binary through other means.
    `cd && juju scp kubernetes-master/0:kubectl ./kubectl`
    
   `sudo cp kubectl /usr/local/bin/`

   4)Query the cluster and make sure K8 is healthy.
    `kubectl cluster-info`

- Install Linux 4.7.2 low latency Kernel (4.7.1 is also supported) at all the K8-Workers

Please note that AWS may not like the here referenced low latency Kernel, in MS Azure and Baremetal/VMs it works fine

Download Kernel

 `wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.7.2/linux-headers-4.7.2-040702_4.7.2-040702.201608201334_all.deb`

 `wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.7.2/linux-headers-4.7.2-040702-lowlatency_4.7.2-040702.201608201334_amd64.deb`

 `wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.7.2/linux-image-4.7.2-040702-lowlatency_4.7.2-040702.201608201334_amd64.deb`

2.1) Install Kernel
 `sudo dpkg -i *.deb`

2.2) Restart your host machine
 `sudo shutdown -r now`

After reboot, login again and check Kernel

2.3) Verify your kernel
 `uname -r`

For other architectures:
http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.7.2/

3) Query the cluster and make sure K8 is healthy.
    `kubectl cluster-info`
    
- Enable support for running Privileged Containers at all the K8-Workers
( make sure to have “jq” otherwise install first sudo apt install jq )

`juju show-status kubernetes-master --format json | \
    jq --raw-output '.applications."kubernetes-master".units | keys[]' | \
    xargs -I UNIT juju ssh UNIT "echo -e '\n# Security Context \nKUBE_ALLOW_PRIV=\"--allow-privileged=true\"' | sudo tee -a /etc/default/kube-apiserver && sudo systemctl restart kube-apiserver.service"`

`juju show-status kubernetes-worker --format json | \
    jq --raw-output '.applications."kubernetes-worker".units | keys[]' | \
    xargs -I UNIT juju ssh UNIT "echo -e '\n# Security Context \nKUBE_ALLOW_PRIV=\"--allow-privileged=true\"' | sudo tee -a /etc/default/kubelet && sudo systemctl restart kubelet.service"`
    
     Query the cluster and make sure K8 is healthy.
    `kubectl cluster-info`

## Deploy Containers
