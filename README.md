# Tanzu Jumpbox using VMOperator

You can leverage this repository to deploy a Tanzu Jumpbox using VMOperator on your Supervisor Cluster (vSphere with Tanzu). This simple implementation exposes the Linux login shell securely over a web browser and can also be accessed by SSH. 

More details on VMOperator and how to use them can be found [here](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-F81E3535-C275-4DDE-B35F-CE759EA3B4A0.html)


Besides standard Linux troubleshooting and networking tools, some of the additional packages that get deployed in the Tanzu Jumpbox are - 

  - docker-ce
  - kubectl
  - jq
  - helm
  - pinniped cli
  - carvel tools
  - velero cli
  - govc

The available SHELL options are - 
  - bash
  - zsh

An additional 10 GB disk is mounted on `/data` for additional storage. 

The default user is `ubuntu` which can run `docker` and `sudo` commands. 

## Deploying the VM

1. Clone this repository.  
2. Modify the `cloud-init.yaml` file.
- Modify the *IP ADDRESS* in the `runcmd` section of the `cloud-init.yaml` to point to the LB IP address of the Supervisor cluster. In this example the K8s API for  the Supervisor Cluster is running on `192.168.11.20`
```
  - curl -ks https://192.168.11.20/wcp/plugin/linux-amd64/vsphere-plugin.zip -o /tmp/vsphere-plugin.zip
```
- Add any relevant Ubuntu packages that may be required
- Save the file.
- BASE64 encode the `cloud-init.yaml` file.
```
$ cat cloud-init.yaml|base64 
```
3. Modify the `vm.yaml`.
- Modify the ConfigMap `jumpbox-configmap` by 
  - Updating the `public_keys` value with your ssh_rsa.pub file content.
  - Updating the `user_data` value with the base64 encoded value from step 2. 
- Modify the `storageClassName` and `storageClass` to the value of your environment.
- Modify the Supervisor `namespace` where the VM will be deployed.
- Modify the `networkName` value with the network name that is configured for the Supervisor namespace. 
- Save the file. 

4. Deploy the VM 
```
kubectl apply -f vm.yaml`
```

## Accessing the VM

Get the IP of the service that was deployed by the vm.yaml 
```
kubectl get svc jumpbox-vmservices -n demo1
NAME                 TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)         AGE
jumpbox-vmservices   LoadBalancer   10.96.0.143   192.168.11.28   443:32546/TCP   12m
```

Open a browser to https://[[  EXTERNAL-IP ]]. After accepting the security warnings, you will be presented with a login session that will ask you to change the password for the first login. The initial password is `changeme` 

```
vmware-tanzu-jumpbox login: ubuntu
Password: 
You are required to change your password immediately (administrator enforced)
Changing password for ubuntu.
Current password: changeme
New password: 
Retype new password: 
```

NOTE - The VM may require a reboot to apply all the security patches that have been installed as part of the cloud-init process. 